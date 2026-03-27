---
name: routing-and-caching
description: >-
  This skill should be used when the user asks about "collection route", "static caching",
  "Stache", "deployment", "statamic routes", "cache clear", "static:warm", "stache:refresh",
  "deploy Statamic", "error pages", "headless mode", or configures Statamic routing,
  caching layers, or production deployment.
version: 0.1.0
---

# Statamic Routing and Caching

## Collection Routes

Define URL patterns for collection entries in the collection's YAML configuration file (e.g., `content/collections/blog.yaml`). The route property controls how Statamic generates and resolves URLs for every entry in the collection.

### Single-site route

```yaml
route: /blog/{slug}
```

Available route variables include `{slug}`, `{year}`, `{month}`, `{day}`, and any field defined on the entry's blueprint. Combine variables to create structured URL hierarchies:

```yaml
route: /blog/{year}/{month}/{slug}    # Date-based: /blog/2026/03/my-post
route: /docs/{parent_slug}/{slug}     # Nested: /docs/getting-started/installation
```

The route pattern determines both URL generation (when outputting `{{ url }}` in templates) and URL resolution (when Statamic matches an incoming request to an entry). If no route is defined on a collection, its entries have no front-end URLs.

### Multisite routes

Specify per-site route patterns when running a multisite installation:

```yaml
route:
  en: /blog/{slug}
  fr: /blogue/{slug}
```

Each site handle maps to its own URI pattern, allowing fully localized URL structures while sharing the same underlying collection. Ensure every configured site has an entry in the route map, or entries for that site will lack front-end URLs.

## Statamic Routes (Route::statamic)

Register template-based routes in `routes/web.php` using `Route::statamic()`. These routes render Antlers (or Blade) templates without requiring a controller or entry.

```php
Route::statamic('about', 'about');                         // URI + template
Route::statamic('about', 'about', ['title' => 'About']);   // URI + template + injected data
Route::statamic('things/{thing}', 'things.show');          // Route with parameters
Route::statamic('feed', 'feed', ['content_type' => 'atom']); // Custom content type
```

**First argument:** the URI pattern.
**Second argument:** the template name (dot notation resolves to subdirectories).
**Third argument (optional):** an array of data injected into the template's cascade, plus special keys like `content_type`.

Use `Route::statamic` for one-off pages that do not belong to a collection (contact, legal, feeds, sitemaps). These routes participate in the Antlers cascade, so layout variables, globals, and nav data are all available within the rendered template.

Standard Laravel routes (`Route::get`, `Route::post`, etc.) still work alongside Statamic routes. Use standard Laravel routes for form handlers, webhooks, or any endpoint that returns JSON rather than a rendered template.

## Error Pages

Create error templates at `resources/views/errors/{status_code}.antlers.html` to customize error responses. Common status codes:

- `404.antlers.html` -- Page not found
- `500.antlers.html` -- Server error
- `503.antlers.html` -- Maintenance mode

Define a shared error layout at `resources/views/errors/layout.antlers.html`. Error templates receive the standard Antlers cascade, including globals and navigation data, so maintain consistent site chrome on error pages.

Statamic checks for a matching status-code template first. If none exists, Laravel's default error handling takes over. Create at minimum a `404.antlers.html` template for every project.

## Headless Mode

Disable all front-end routes to run Statamic as a headless CMS (API-only or CP-only):

```php
// config/statamic/routes.php
'enabled' => false,
```

When disabled, Statamic serves no front-end pages. The Control Panel and REST/GraphQL APIs remain active. Use this configuration when a separate front-end application (Next.js, Nuxt, SvelteKit, etc.) consumes content exclusively via the Content API or GraphQL endpoint. Collection routes, `Route::statamic` calls, and error page templates are all bypassed in headless mode.

## Caching

Statamic provides four distinct caching layers. Understand each layer to diagnose performance issues and configure appropriate invalidation.

### Layer 1: Stache (Content Cache)

The Stache is Statamic's flat-file indexing system. It reads YAML/Markdown content files and builds an in-memory index for fast querying. The Stache cannot be disabled -- it is fundamental to how Statamic resolves content.

**Key commands:**

```bash
php please stache:clear     # Remove all Stache indexes
php please stache:warm      # Rebuild all Stache indexes from content files
php please stache:refresh   # Clear + rebuild in one step
php please stache:doctor    # Diagnose Stache issues
```

**Configuration** (`config/statamic/stache.php`):

```php
'watcher' => env('STATAMIC_STACHE_WATCHER', 'auto'), // auto|true|false
'stores' => [
    'entries' => ['directory' => base_path('content/collections')],
],
'indexes' => ['custom_field'], // Additional fields to index for query performance
'lock' => ['enabled' => true, 'timeout' => 30],
```

- **`watcher`**: Controls file-change detection. Set to `auto` (recommended) or `false` in production when content does not change on disk between deploys.
- **`stores`**: Map store names to content directories. Rarely changed from defaults.
- **`indexes`**: Add field names that appear in `where` clauses to speed up queries.
- **`lock`**: Prevents concurrent Stache rebuilds. Keep enabled in production.

### Layer 2: Application Cache

The standard Laravel cache, used by Statamic for computed data and query results. Backed by the configured cache driver (file, Redis, Memcached, etc.).

```bash
php artisan cache:clear    # Clear the entire application cache
```

Clearing the application cache does not affect the Stache or static cache.

### Layer 3: View Fragments

Cache expensive template sections using the Antlers `cache` tag:

```antlers
{{ cache for="1 hour" }}
    {{# Expensive query or computation here #}}
{{ /cache }}
```

Fragment caches are stored in the application cache and respect the same cache driver and TTL configuration. Use for navigation trees, sidebar widgets, footer content, or any repeated partial that performs expensive queries. The `for` parameter accepts human-readable durations: `"30 minutes"`, `"1 hour"`, `"1 day"`.

Nest fragment caches carefully -- avoid caching an outer block that already contains cached inner blocks, as this can lead to stale content or redundant cache entries.

### Layer 4: Static Caching

Full-page caching for maximum performance. Two strategies are available.

#### Half Measure (Application Driver)

Store rendered pages in the Laravel cache. Requests still pass through PHP, but skip template rendering. Approximately 50% faster than uncached responses.

```php
// config/statamic/static_caching.php
'strategy' => 'half',
```

No server configuration changes required. Suitable for sites where full-measure server rewrite setup is impractical, or for pages that need some dynamic behavior (e.g., CSRF tokens, session-dependent content). The half measure still respects invalidation on content save.

#### Full Measure (File Driver)

Generate static `.html` files served directly by the web server, bypassing PHP entirely. Response times around 2ms.

```php
// config/statamic/static_caching.php
'strategy' => 'full',
'strategies' => ['full' => [
    'driver' => 'file',
    'path' => public_path('static'),
    'warm_concurrency' => 25,
]],
```

**Requirements:**

- Server rewrite rules for Apache, Nginx, or IIS to serve static files before hitting PHP.
- Auto-invalidation triggers on content save through Statamic's event system.

**Static cache commands:**

```bash
php please static:warm           # Crawl all URLs and generate cached pages
php please static:warm --queue   # Run warming in the background via queue
php please static:clear          # Remove all cached static pages
```

Set `warm_concurrency` to control how many pages are warmed in parallel during `static:warm`. Start with 25 and adjust based on server capacity.

#### Choosing a strategy

| Criterion | Half Measure | Full Measure |
|-----------|-------------|--------------|
| Speed gain | ~50% faster | ~98% faster (2ms responses) |
| Server config | None | Rewrite rules required |
| Dynamic content | Supported (runs PHP) | Not supported (static HTML) |
| Setup complexity | Low | Medium |

Use the `{{ nocache }}` tag to exclude dynamic sections (forms, user-specific content) from static caching when using the full measure. Content within `{{ nocache }}...{{ /nocache }}` blocks is rendered on each request even when the surrounding page is served from the static cache.

## Deployment Checklist

Follow these steps when deploying Statamic to production:

1. **Set environment variables:**
   - `APP_ENV=production`
   - `APP_DEBUG=false`
   - `STATAMIC_STACHE_WATCHER=false` (or `auto`)

2. **Warm the Stache after deploy:**
   ```bash
   php please stache:warm
   ```

3. **Configure static caching** for performance (half or full measure as appropriate).

4. **Rebuild search indexes:**
   ```bash
   php please search:update --all
   ```

5. **Clear application cache** if deploy script requires it:
   ```bash
   php artisan cache:clear
   ```

6. **Configure Git Automation** push for content changes made through the Control Panel.

A typical deploy script runs these in sequence: `cache:clear` -> `stache:warm` -> `static:warm` -> `search:update --all`.

### Common deployment pitfalls

- **Forgetting `stache:warm`**: First request after deploy triggers a cold Stache build, causing a slow response for that visitor. Always warm proactively.
- **Leaving `APP_DEBUG=true`**: Exposes stack traces and sensitive configuration to end users. Verify this is `false` before going live.
- **Stache watcher in production**: Setting `STATAMIC_STACHE_WATCHER=true` on a server where files do not change between deploys wastes resources on filesystem polling. Use `false` or `auto`.
- **Missing static cache invalidation**: If using full-measure static caching, ensure the deploy script runs `static:clear` before `static:warm` to avoid serving stale pages.

## Key Conventions

- Use Antlers (`.antlers.html`) as the default template language.
- Prefix partial-only views with an underscore: `_card.antlers.html`.
- Store content as flat files unless scaling demands the Eloquent driver.
- Use `inject` on collections for default cascade values.
- Never commit `.env` to version control.
- Access configuration in templates with `{{ config:app:key }}` syntax.
- Use version range constraints (`^6.0`) in `composer.json`.
- Maintain a README documenting any overrides to Statamic defaults.

## CLI Commands

For the complete Statamic CLI command reference organized by category (project management, content creation, cache management, search, code generation, git/deployment, imports/migrations, starter kits, and updating), see [references/cli-commands.md](references/cli-commands.md).

### Essential cache commands (quick reference)

| Command | Purpose |
|---------|---------|
| `php please stache:clear` | Remove Stache indexes |
| `php please stache:warm` | Rebuild Stache from files |
| `php please stache:refresh` | Clear + warm in one step |
| `php please static:warm` | Crawl and cache all pages |
| `php please static:clear` | Remove all static cache files |
| `php artisan cache:clear` | Clear Laravel application cache |
| `php please glide:clear` | Clear image manipulation cache |
