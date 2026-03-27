---
name: assets-forms-search
description: >-
  This skill should be used when the user asks about "asset container", "Glide",
  "image manipulation", "form blueprint", "form submission", "search config",
  "Algolia", "search index", "REST API", "GraphQL", "API endpoints", "multi-site",
  "live preview", "git automation", or works with Statamic assets, forms, search,
  APIs, or Control Panel features.
version: 0.1.0
---

# Assets, Forms, Search, and API

## Asset Containers

### Configuration

Define each asset container in `content/assets/{handle}.yaml`:

```yaml
title: Images
disk: assets
```

Register the corresponding filesystem disk in `config/filesystems.php`:

```php
'assets' => [
    'driver' => 'local',
    'root' => public_path('assets'),
    'url' => '/assets',
    'visibility' => 'public',
],
```

Statamic supports local disks, Amazon S3, and any Flysystem adapter. Asset metadata is stored automatically in `.meta` subdirectories alongside the assets themselves.

### S3 Configuration

For S3, configure the disk with the `s3` driver in `config/filesystems.php`:

```php
'assets' => [
    'driver' => 's3',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION'),
    'bucket' => env('AWS_BUCKET'),
    'url' => env('AWS_URL'),
    'visibility' => 'public',
],
```

The asset container YAML references the disk by its key name -- no other changes needed. Any Flysystem adapter (DigitalOcean Spaces, MinIO, etc.) works the same way by configuring the appropriate driver.

### Asset Metadata

Statamic stores metadata for each asset in `.meta` subdirectories as YAML files. Metadata includes dimensions, file size, last modified date, and any custom fields defined in the asset container blueprint. Regenerate metadata after bulk file operations with `php please assets:meta`.

## Glide Image Manipulation

### Configuration

Configure Glide in `config/statamic/assets.php` under the `image_manipulation` key:

```php
'image_manipulation' => [
    'route' => 'img',
    'cache' => true,
    'cache_path' => public_path('img'),
    'presets' => [
        'thumbnail' => ['w' => 300, 'h' => 300, 'q' => 75, 'fit' => 'crop'],
        'hero' => ['w' => 1440, 'h' => 600, 'q' => 90],
    ],
    'driver' => 'gd', // or 'imagick'
],
```

Set `driver` to `gd` (default) or `imagick` depending on the PHP extension available.

### Parameters

All Glide parameters: `width`/`w`, `height`/`h`, `fit` (contain, max, fill, stretch, crop, crop_focal), `quality`/`q` (0-100), `format`/`fm` (jpg, png, webp, avif), `blur`, `brightness`, `contrast`, `sharpen`, `pixelate`, `filter` (greyscale, sepia), `dpr`, `orient`, `flip`, `bg`, and watermark params (`mark`, `markw`, `markh`, etc.).

See `references/api-endpoints.md` for the full Glide parameter table.

### Presets

Define reusable presets in the config array. Apply in Antlers with the `preset` parameter on the `glide` tag or in the asset URL. Pre-generate all presets with:

```bash
php please assets:generate-presets
```

### CLI Commands

```bash
php please glide:clear                    # Clear Glide image cache
php please assets:generate-presets        # Pre-generate all preset images
php please assets:meta                    # Regenerate asset metadata
php please assets:clear-cache             # Clear asset caches
```

## Forms

### Form Blueprint

Define form fields in `resources/blueprints/forms/{handle}.yaml`:

```yaml
fields:
  - handle: name
    field: { display: Name, type: text, validate: required }
  - handle: email
    field: { display: Email, type: text, validate: 'required|email' }
  - handle: message
    field: { display: Message, type: textarea, validate: required }
```

Validation rules follow Laravel validation syntax. Pipe-delimit multiple rules (e.g., `'required|email|max:255'`). Use single quotes around rules containing pipes in YAML.

### Email Notifications

Configure email notifications in `resources/forms/{handle}.yaml`:

```yaml
title: Contact
email:
  - to: [email protected]
    from: "{{ email }}"
    reply_to: "{{ email }}"
    subject: "New Contact: {{ subject ?? 'Form Submission' }}"
    html: emails/contact
    attachments: true
```

The `html` key points to a view under `resources/views/`. Antlers variables from the submission are available in the template. Multiple email entries can be defined to send to different recipients.

### Antlers Rendering

Render a form in Antlers templates:

```antlers
{{ form:contact }}
    {{ fields }}
        <label>{{ display }}</label>
        {{ field }}
    {{ /fields }}
    <button type="submit">Send</button>
{{ /form:contact }}
```

Handle success and errors:

```antlers
{{ form:contact }}
    {{ if success }}
        <p>Thank you!</p>
    {{ /if }}
    {{ if errors }}
        {{ errors }}
            <p>{{ value }}</p>
        {{ /errors }}
    {{ /if }}
    ...
{{ /form:contact }}
```

### AJAX Form Submission

Submit forms via AJAX by posting to `POST /!/forms/{form-handle}`. Include the `X-Requested-With: XMLHttpRequest` header. The request body must include `_token` (CSRF token) and `_params` fields alongside the form data. The endpoint returns JSON with success/error status and any validation messages.

Example JavaScript submission:

```javascript
fetch('/!/forms/contact', {
    method: 'POST',
    headers: {
        'X-Requested-With': 'XMLHttpRequest',
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({
        _token: document.querySelector('meta[name="csrf-token"]').content,
        name: 'John',
        email: '[email protected]',
        message: 'Hello',
    }),
});
```

### Form Submissions Storage

Submissions are stored as YAML files in `storage/forms/{handle}/`. Each submission gets a timestamped filename. Access submissions programmatically via the `Form` facade or view them in the Control Panel under Forms.

## Search

### Configuration

Configure search indexes in `config/statamic/search.php`:

```php
'default' => [
    'driver' => 'local',          // or 'algolia'
    'searchables' => 'content',   // or ['collection:blog', 'taxonomy:tags', 'users']
    'fields' => ['title', 'content'],
    'transformers' => [...],
],
```

Set `searchables` to `'content'` to index everything, or pass an array of specific collections, taxonomies, or `'users'`. The `fields` array controls which fields are indexed for searching. Use `transformers` to modify field values before indexing (e.g., stripping HTML tags from content fields).

Define multiple named indexes for different search contexts (e.g., a `blog` index and a `docs` index with different searchable content and field weights).

### Drivers

**Local (Comb)** -- JSON-based index with configurable scoring and stemming. No external dependencies. Suitable for small to medium sites.

**Algolia** -- Requires `algolia/algoliasearch-client-php:^3.4`. Set `driver` to `algolia` and configure API credentials in `.env`:

```env
ALGOLIA_APP_ID=your-app-id
ALGOLIA_SECRET=your-secret-key
```

### CLI Commands

```bash
php please search:update              # Interactive index update
php please search:update default      # Update a specific index
php please search:update --all        # Update all indexes
```

## REST API (Pro Feature)

Enable the REST API by setting `STATAMIC_API_ENABLED=true` in `.env`.

All endpoints are prefixed with `/api/`. See `references/api-endpoints.md` for the full endpoint table with methods, paths, and descriptions.

### Filtering

```
/api/collections/blog/entries?filter[title:contains]=awesome&filter[status]=published
```

Filter operators include `contains`, `is`, `not`, and field-specific operators.

### Sorting

```
?sort=-date,title
```

Prefix with `-` for descending order. Comma-separate multiple sort fields.

### Field Selection

```
?fields=id,title,content
```

Limit response payload to specific fields.

### Pagination

```
?limit=10&page=2
```

Response includes pagination metadata with `total`, `count`, `per_page`, `current_page`, `total_pages`, and navigation links.

### API Configuration

Customize API behavior in `config/statamic/api.php`. Configure allowed resources, enable or disable specific endpoints, set default pagination limits, and define custom middleware. Restrict API access with authentication middleware or Laravel Sanctum for token-based auth.

## GraphQL API (Pro Feature)

Enable with `STATAMIC_GRAPHQL_ENABLED=true` in `.env`. Access GraphiQL from the Control Panel.

Key queries: `entries`, `entry(id:)`, `collections`, `collection(handle:)`, `terms`, `term(id:)`, `assets`, `asset`, `globalSets`, `globalSet(handle:)`, `navs`, `nav(handle:)`, `users`, `user`, `forms`, `form(handle:)`.

Use blueprint-specific types via fragments:

```graphql
{
  entries(collection: "blog") {
    data {
      title
      ... on Entry_Blog_Post {
        intro
        content
      }
    }
  }
}
```

See `references/api-endpoints.md` for full GraphQL query reference.

## Multi-site (Pro Feature)

Enable multi-site by setting `'multisite' => true` in `config/statamic/system.php`, then run:

```bash
php please multisite
```

Define sites in `resources/sites.yaml`:

```yaml
en:
  name: English
  url: /
  locale: en_US
fr:
  name: French
  url: /fr/
  locale: fr_FR
  lang: fr
```

Each site can have its own URL prefix, locale, and language. Content can be localized per site with origin-based entry linking. The origin entry is the default locale version; localized entries reference it and override specific fields.

Configure per-site content in collections by setting `sites` in the collection YAML to control which sites a collection is available on. Taxonomy terms and global sets can also be localized per site.

## Git Automation (Pro Feature)

Enable automatic git commits for content changes:

```env
STATAMIC_GIT_ENABLED=true
STATAMIC_GIT_AUTOMATIC=true
STATAMIC_GIT_PUSH=true
```

Configure tracked paths in `config/statamic/git.php`. By default, Statamic tracks `content/`, `users/`, and `resources/` directories. Customize tracked paths to include or exclude specific directories.

When `STATAMIC_GIT_AUTOMATIC=true`, Statamic commits after every content change in the Control Panel. When `STATAMIC_GIT_PUSH=true`, it also pushes to the remote. Trigger a manual commit:

```bash
php please git:commit
```

Set commit message format and author in the git config. Use queue workers for async git operations to avoid blocking CP responses.

## Live Preview

Configure in `config/statamic/live_preview.php` -- set device sizes, toolbar inputs, and hot reload behavior.

Define preview targets per collection in the collection YAML:

```yaml
preview_targets:
  - label: Entry
    url: /blog/{slug}
  - label: Index
    url: /blog
```

## Control Panel

Access the CP at `/cp`. Open the Command Palette with `Cmd+K` (macOS) or `Ctrl+K` (Linux/Windows). Statamic v6 uses Vue 3 + Pinia architecture.

Create users via `php please make:user` or through CP invitations.

### Permissions (Pro)

Role-based permissions stored in `resources/users/roles.yaml`. Assign roles to users or user groups for granular access control. Permissions cover collections, taxonomies, globals, assets, users, forms, and utilities. Define custom roles with fine-grained permissions (e.g., allow editing blog entries but not deleting them, or restrict access to specific asset containers).

### Conditional Fields in Blueprints

```yaml
- handle: venue
  field:
    type: text
    if:
      event_type: in_person
    validate: [sometimes, required]

- handle: stream_url
  field:
    type: text
    unless:
      event_type: in_person
```

**Operators**: `is`/`equals`/`==`, `not`/`isnt`/`!=`, `===`, `!==`, `>`, `>=`, `<`, `<=`, `contains`, `contains_any`, `empty`, `not empty`, `null`, `true`, `false`.

**Logic**: Multiple conditions default to AND. Use `if_any` / `unless_any` for OR logic. Access nested fields with dot notation (`address.country`), parent context with `$parent.field`, root context with `$root.field`.
