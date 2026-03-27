---
name: statamic-php-api
description: >-
  This skill should be used when the user works with "Entry facade", "query builder",
  "Statamic events", "query scope", "view model", "computed values", "PHP API",
  "Entry::query", "Collection::computed", or uses Statamic facades for data retrieval,
  content manipulation, event handling, or controller queries.
version: 0.1.0
---

# Statamic PHP API

## Facades for Data Retrieval

Use facades from `Statamic\Facades` to interact with all content types. The primary facades are:

```php
use Statamic\Facades\Entry;
use Statamic\Facades\Collection;
use Statamic\Facades\Term;
use Statamic\Facades\GlobalSet;
use Statamic\Facades\Asset;
use Statamic\Facades\User;
use Statamic\Facades\Nav;
use Statamic\Facades\Blueprint;
use Statamic\Facades\Site;
```

Each facade provides `find()`, `query()`, and `make()` methods as entry points.

### Facade Purposes

- **Entry** -- query, create, update, and delete entries across all collections
- **Collection** -- manage collection configurations, register computed values, access collection metadata
- **Term** -- work with taxonomy terms, query by taxonomy
- **GlobalSet** -- access site-wide global variable sets and their localized variants
- **Asset** -- manage files in asset containers, query by container, manipulate metadata
- **User** -- query user accounts, create users, manage roles and groups
- **Nav** -- access navigation structures and their tree data
- **Blueprint** -- retrieve and manipulate content blueprints and their field definitions
- **Site** -- access multi-site configuration, get current site, list all configured sites

## Finding Content

Retrieve individual items by ID or URI:

```php
$entry = Entry::find('abc123');
$entry = Entry::findByUri('/blog/my-post');
$entry = Entry::findByUri('/blog/my-post', 'french'); // Site-specific lookup
```

`find()` accepts a string ID (the filename stem from the flat-file content). `findByUri()` accepts the URI path and returns the entry matching that route. Pass an optional site handle as the second argument for multi-site lookups. Both methods return `null` when no match is found.

For other content types, use the equivalent facade methods:

```php
$term = Term::find('tags::featured');
$user = User::findByEmail('admin@example.com');
$asset = Asset::find('images::hero.jpg');    // container::path format
$nav = Nav::findByHandle('main');
```

## Query Builder

Build queries using the fluent query builder accessed via `::query()`:

```php
$entries = Entry::query()
    ->where('collection', 'blog')
    ->where('status', 'published')
    ->where('date', '>=', now()->subDays(30))
    ->orderBy('date', 'desc')
    ->limit(10)
    ->get();
```

### Available Query Methods

- `where($field, $operator, $value)` -- filter by field value
- `whereBetween($field, [$min, $max])` -- range filter
- `whereIn($field, $values)` -- match any value in array
- `whereNull($field)` / `whereNotNull($field)` -- null checks
- `whereDate($field, $value)`, `whereMonth`, `whereDay`, `whereYear`, `whereTime` -- date component filters
- `whereJsonContains($field, $value)` -- JSON field search
- `whereJsonLength($field, $operator, $value)` -- JSON array length
- `whereColumn($col1, $operator, $col2)` -- compare columns
- `when($condition, $callback)` / `unless($condition, $callback)` -- conditional clauses
- `orderBy($field, $direction)` -- sort results
- `limit($n)` -- cap result count
- `get()` -- execute and return collection
- `paginate($perPage)` -- paginate results

### Query Builder Operators

Standard operators: `=`, `<>`, `!=`, `like`, `not like`, `regexp`, `not regexp`, `>`, `<`, `>=`, `<=`.

Closure-based queries for complex grouping:

```php
Entry::query()->where(function ($query) {
    $query->where('status', 'published')
          ->orWhere('featured', true);
})->get();
```

See `references/events-and-operators.md` for complete operator reference with examples.

## Creating and Manipulating Entries

### Create a new entry

```php
$entry = Entry::make()
    ->collection('blog')
    ->slug('new-post')
    ->published(true)
    ->data(['title' => 'New Post', 'content' => 'Hello!']);
$entry->save();
```

### Update an existing entry

```php
$entry->set('title', 'Updated Title');
$entry->save();
```

### Save without firing events

```php
$entry->saveQuietly();
```

Use `saveQuietly()` to suppress all `EntrySaving` and `EntrySaved` events. Useful in migrations, imports, or batch operations where event side-effects are undesirable.

### Batch Operations

When creating or updating multiple entries, wrap operations to avoid excessive event firing and Stache rebuilds:

```php
$items = [['title' => 'Post A', 'slug' => 'post-a'], ['title' => 'Post B', 'slug' => 'post-b']];

foreach ($items as $item) {
    Entry::make()
        ->collection('blog')
        ->slug($item['slug'])
        ->data(['title' => $item['title']])
        ->saveQuietly();
}
```

### Deleting Entries

```php
$entry = Entry::find('abc123');
$entry->delete();
```

Deletion fires `EntryDeleting` (preventable) and `EntryDeleted` events.

## Data Access: Augmented vs Raw

Statamic distinguishes between augmented (processed) and raw (stored) values:

```php
$entry->title;              // Augmented value
$entry->content;            // Augmented (e.g., Markdown converted to HTML)
$entry->get('title');       // Raw stored value
$entry->value('title');     // Raw with inheritance (falls back to collection defaults)
$entry->data();             // All raw data as array
$entry->toAugmentedArray(); // All values, augmented
```

**Augmented** values pass through fieldtype processing. A Markdown field returns HTML. A relationship field returns entry objects. Use augmented values for display.

**Raw** values return exactly what is stored in the YAML file. Use raw values for comparisons, conditions, and data manipulation.

**Inherited** values via `value()` check the entry first, then fall back to the collection's `inject` defaults.

## Global Sets

Access global variable sets:

```php
$footer = GlobalSet::findByHandle('footer')->inDefaultSite();
$copyright = $footer->get('copyright');
```

Call `inDefaultSite()` or `inCurrentSite()` to get the localized variables object, then access data via `get()`.

## Events

Statamic fires 86+ events in the `Statamic\Events` namespace covering the full content lifecycle. Key events:

- `EntrySaving` / `EntrySaved` / `EntryCreated` / `EntryDeleted`
- `TermSaved` / `AssetUploaded` / `FormSubmitted`
- `UserSaved` / `BlueprintSaved` / `CollectionSaved`
- `NavTreeSaved` / `GlobalVariablesSaved`
- `SearchIndexUpdated` / `StacheWarmed` / `UrlInvalidated`

### Preventing Actions

Return `false` from a `Saving` event listener to prevent the save:

```php
use Statamic\Events\EntrySaving;

class PreventDraftPublishing
{
    public function handle(EntrySaving $event)
    {
        if ($event->entry->get('needs_review')) {
            return false; // Prevents the save
        }
    }
}
```

Register listeners in `EventServiceProvider` as with any Laravel event.

See `references/events-and-operators.md` for the full event catalog.

## Hooks (PHP Pipelines)

Hooks allow tapping into tag pipelines to modify results. Use the `hook()` method on tag classes:

```php
use Statamic\Tags\Collection;

Collection::hook('fetched-entries', function ($entries, $next) {
    return $next($entries->take(3));
});
```

The hook receives the current value and a `$next` closure. Always call `$next()` with the modified (or unmodified) value to continue the pipeline. Register hooks in a service provider's `boot()` method.

### Pipeline Rules

- Always call `$next($value)` to pass data to the next hook in the chain. Omitting this call stops all downstream hooks.
- The first argument is the pipeline value, typically a Laravel Collection of entries.
- Multiple hooks on the same point execute in the order they were registered.
- Place hook registrations in the `boot()` method of a service provider, not in `register()`.

### Example: Injecting Supplemental Data

```php
Collection::hook('fetched-entries', function ($entries, $next) {
    $entries->each(function ($entry) {
        $entry->setSupplement('external_score', fetchScore($entry->slug()));
    });
    return $next($entries);
});
```

Use `setSupplement()` to attach transient data to entries without modifying stored content. Supplements are available in templates alongside regular fields.

## Query Scopes

Define reusable query constraints as scope classes:

```php
namespace App\Scopes;

use Statamic\Query\Scopes\Scope;

class Featured extends Scope
{
    public function apply($query, $values)
    {
        $query->where('featured', true);
    }
}
```

### Registration

Register scopes in a service provider:

```php
use App\Scopes\Featured;
use Statamic\Facades\Scope;

Scope::register('featured', Featured::class);
```

### Usage in Templates

```antlers
{{ collection:blog query_scope="featured" }}
```

The `$values` parameter receives any additional parameters passed from the tag. Pass extra parameters from Antlers and access them via `$values->get('param_name')` inside the scope's `apply()` method.

### Filter Scopes for the Control Panel

Extend `Statamic\Query\Scopes\Filter` instead of `Scope` to create interactive filter UI in the control panel listings. Filter scopes define `fieldItems()` for the filter form, `apply()` for the query logic, `badge()` for the active filter label, and `visibleTo()` to control which listing pages show the filter. See `references/events-and-operators.md` for a full Filter scope example.

## Computed Values

Define computed fields that derive values at runtime rather than storing them:

```php
// app/Providers/AppServiceProvider.php boot()
use Statamic\Facades\Collection;

Collection::computed('articles', 'read_time', function ($entry, $value) {
    return ceil(str_word_count(strip_tags($entry->content)) / 200);
});
```

Arguments: collection handle, field handle, closure receiving the entry and current value.

Set the field visibility to `computed` in the blueprint to make it read-only in the control panel:

```yaml
fields:
  -
    handle: read_time
    field:
      type: integer
      visibility: computed
```

Computed values are available in templates and the API like any other field. They are recalculated each time the entry is accessed, so keep computations lightweight or cache expensive results.

### Multiple Computed Values

Register multiple computed values for the same or different collections:

```php
Collection::computed('articles', 'read_time', function ($entry, $value) {
    return ceil(str_word_count(strip_tags($entry->content)) / 200);
});

Collection::computed('articles', 'excerpt_auto', function ($entry, $value) {
    return Str::limit(strip_tags($entry->content), 160);
});

Collection::computed('products', 'price_formatted', function ($entry, $value) {
    return '$' . number_format($entry->get('price') / 100, 2);
});
```

## View Model Pattern

Use view models to inject derived data into templates without cluttering entries:

```php
namespace App\ViewModels;

use Statamic\View\ViewModel;

class ArticleStats extends ViewModel
{
    public function data(): array
    {
        $content = strip_tags($this->cascade->get('content'));
        $words = str_word_count($content);
        return [
            'word_count' => $words,
            'read_time' => ceil($words / 200),
        ];
    }
}
```

### Attaching to a Collection

Configure in the collection YAML:

```yaml
inject:
  view_model: App\ViewModels\ArticleStats
```

The `data()` return values merge into the template cascade and are accessible as variables alongside entry data. Access the full cascade (including entry data, globals, and site variables) via `$this->cascade` within the view model.

### When to Use View Models vs Computed Values

Use **computed values** for single derived fields tied to a specific collection (e.g., read time, formatted price). Use **view models** when injecting multiple related variables, when the computation needs access to the full cascade (not just the entry), or when the logic is complex enough to warrant its own class.

## Content Queries in Controllers

Use Statamic facades in custom Laravel controllers for full control over routing and rendering:

```php
use Statamic\Facades\Entry;

class BlogController extends Controller
{
    public function index()
    {
        $entries = Entry::query()
            ->where('collection', 'blog')
            ->where('status', 'published')
            ->orderBy('date', 'desc')
            ->paginate(15);

        return (new \Statamic\View\View)
            ->template('blog.index')
            ->layout('layout')
            ->with(['entries' => $entries]);
    }
}
```

Use `\Statamic\View\View` instead of Laravel's `view()` helper to render through the Statamic view layer with Antlers support, cascade injection, and layout handling. Pass data via `with()`.

### Returning JSON from Controllers

For API-style endpoints, return augmented data directly:

```php
public function show($slug)
{
    $entry = Entry::query()
        ->where('collection', 'blog')
        ->where('slug', $slug)
        ->first();

    if (! $entry) {
        abort(404);
    }

    return response()->json($entry->toAugmentedArray());
}
```

### Registering Routes for Custom Controllers

Register routes in `routes/web.php` as standard Laravel routes. Statamic's route handling coexists with Laravel routes -- explicit Laravel routes take priority over Statamic's content routing.
