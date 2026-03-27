# Statamic Events and Query Operators Reference

## Query Builder Operators

### Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `=` | Equal (default) | `->where('status', 'published')` |
| `<>` | Not equal | `->where('status', '<>', 'draft')` |
| `!=` | Not equal (alias) | `->where('status', '!=', 'draft')` |
| `>` | Greater than | `->where('price', '>', 100)` |
| `<` | Less than | `->where('price', '<', 50)` |
| `>=` | Greater than or equal | `->where('date', '>=', now())` |
| `<=` | Less than or equal | `->where('stock', '<=', 10)` |

### Pattern Matching Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `like` | SQL LIKE pattern | `->where('title', 'like', '%laravel%')` |
| `not like` | Negated LIKE | `->where('title', 'not like', '%draft%')` |
| `regexp` | Regular expression | `->where('slug', 'regexp', '^blog-.*')` |
| `not regexp` | Negated regex | `->where('slug', 'not regexp', '^test-.*')` |

### Advanced Where Clauses

#### whereBetween / whereNotBetween

```php
Entry::query()->whereBetween('price', [10, 100])->get();
Entry::query()->whereNotBetween('date', ['2025-01-01', '2025-06-30'])->get();
```

#### whereIn / whereNotIn

```php
Entry::query()->whereIn('status', ['published', 'scheduled'])->get();
Entry::query()->whereNotIn('collection', ['drafts', 'archive'])->get();
```

#### whereNull / whereNotNull

```php
Entry::query()->whereNull('published_at')->get();
Entry::query()->whereNotNull('featured_image')->get();
```

#### Date Component Filters

```php
Entry::query()->whereDate('created_at', '2025-03-15')->get();
Entry::query()->whereMonth('created_at', 3)->get();
Entry::query()->whereDay('created_at', 15)->get();
Entry::query()->whereYear('created_at', 2025)->get();
Entry::query()->whereTime('created_at', '>=', '09:00:00')->get();
```

#### JSON Queries

```php
// Check if a JSON array field contains a value
Entry::query()->whereJsonContains('tags', 'laravel')->get();

// Check length of a JSON array field
Entry::query()->whereJsonLength('tags', '>=', 3)->get();
```

#### Column Comparison

```php
Entry::query()->whereColumn('updated_at', '>', 'created_at')->get();
```

#### Conditional Clauses

```php
Entry::query()
    ->when($request->has('featured'), function ($query) {
        $query->where('featured', true);
    })
    ->unless($showDrafts, function ($query) {
        $query->where('status', 'published');
    })
    ->get();
```

#### Closure-Based Grouping

```php
Entry::query()->where(function ($query) {
    $query->where('status', 'published')
          ->orWhere('featured', true);
})->get();
```

## Statamic Events

All events live in the `Statamic\Events` namespace. There are 86+ events covering the full content lifecycle.

### Entry Events

| Event | Fired When | Can Prevent? |
|-------|-----------|-------------|
| `EntrySaving` | Before an entry is saved | Yes (return `false`) |
| `EntrySaved` | After an entry is saved | No |
| `EntryCreating` | Before a new entry is created | Yes (return `false`) |
| `EntryCreated` | After a new entry is created | No |
| `EntryDeleting` | Before an entry is deleted | Yes (return `false`) |
| `EntryDeleted` | After an entry is deleted | No |
| `EntryBlueprintFound` | When an entry's blueprint is resolved | No |

### Collection Events

| Event | Fired When | Can Prevent? |
|-------|-----------|-------------|
| `CollectionSaving` | Before a collection is saved | Yes |
| `CollectionSaved` | After a collection is saved | No |
| `CollectionCreating` | Before a new collection is created | Yes |
| `CollectionCreated` | After a new collection is created | No |
| `CollectionDeleting` | Before a collection is deleted | Yes |
| `CollectionDeleted` | After a collection is deleted | No |
| `CollectionTreeSaving` | Before a collection tree is saved | Yes |
| `CollectionTreeSaved` | After a collection tree is saved | No |

### Taxonomy Events

| Event | Fired When | Can Prevent? |
|-------|-----------|-------------|
| `TaxonomySaving` | Before a taxonomy is saved | Yes |
| `TaxonomySaved` | After a taxonomy is saved | No |
| `TaxonomyCreating` | Before a new taxonomy is created | Yes |
| `TaxonomyCreated` | After a new taxonomy is created | No |
| `TaxonomyDeleting` | Before a taxonomy is deleted | Yes |
| `TaxonomyDeleted` | After a taxonomy is deleted | No |
| `TermSaving` | Before a term is saved | Yes |
| `TermSaved` | After a term is saved | No |
| `TermCreating` | Before a new term is created | Yes |
| `TermCreated` | After a new term is created | No |
| `TermDeleting` | Before a term is deleted | Yes |
| `TermDeleted` | After a term is deleted | No |

### Navigation Events

| Event | Fired When | Can Prevent? |
|-------|-----------|-------------|
| `NavSaving` | Before a nav is saved | Yes |
| `NavSaved` | After a nav is saved | No |
| `NavCreating` | Before a new nav is created | Yes |
| `NavCreated` | After a new nav is created | No |
| `NavDeleting` | Before a nav is deleted | Yes |
| `NavDeleted` | After a nav is deleted | No |
| `NavTreeSaving` | Before a nav tree is saved | Yes |
| `NavTreeSaved` | After a nav tree is saved | No |

### Global Set Events

| Event | Fired When | Can Prevent? |
|-------|-----------|-------------|
| `GlobalSetSaving` | Before a global set is saved | Yes |
| `GlobalSetSaved` | After a global set is saved | No |
| `GlobalSetCreating` | Before a new global set is created | Yes |
| `GlobalSetCreated` | After a new global set is created | No |
| `GlobalSetDeleting` | Before a global set is deleted | Yes |
| `GlobalSetDeleted` | After a global set is deleted | No |
| `GlobalVariablesSaving` | Before global variables are saved | Yes |
| `GlobalVariablesSaved` | After global variables are saved | No |

### Asset Events

| Event | Fired When | Can Prevent? |
|-------|-----------|-------------|
| `AssetSaving` | Before an asset is saved | Yes |
| `AssetSaved` | After an asset is saved | No |
| `AssetCreating` | Before a new asset is created | Yes |
| `AssetCreated` | After a new asset is created | No |
| `AssetDeleting` | Before an asset is deleted | Yes |
| `AssetDeleted` | After an asset is deleted | No |
| `AssetUploading` | Before an asset is uploaded | Yes |
| `AssetUploaded` | After an asset is uploaded | No |
| `AssetReplacing` | Before an asset file is replaced | Yes |
| `AssetReplaced` | After an asset file is replaced | No |
| `AssetReuploaded` | After an asset is re-uploaded | No |
| `AssetContainerSaving` | Before a container is saved | Yes |
| `AssetContainerSaved` | After a container is saved | No |
| `AssetContainerCreating` | Before a new container is created | Yes |
| `AssetContainerCreated` | After a new container is created | No |
| `AssetContainerDeleting` | Before a container is deleted | Yes |
| `AssetContainerDeleted` | After a container is deleted | No |
| `AssetFolderSaved` | After a folder is saved | No |
| `AssetFolderDeleted` | After a folder is deleted | No |

### User Events

| Event | Fired When | Can Prevent? |
|-------|-----------|-------------|
| `UserSaving` | Before a user is saved | Yes |
| `UserSaved` | After a user is saved | No |
| `UserCreating` | Before a new user is created | Yes |
| `UserCreated` | After a new user is created | No |
| `UserDeleting` | Before a user is deleted | Yes |
| `UserDeleted` | After a user is deleted | No |
| `UserGroupSaved` | After a user group is saved | No |
| `UserGroupDeleted` | After a user group is deleted | No |
| `UserRegistering` | Before registration completes | Yes |
| `UserRegistered` | After a user registers | No |

### Blueprint and Fieldset Events

| Event | Fired When | Can Prevent? |
|-------|-----------|-------------|
| `BlueprintSaving` | Before a blueprint is saved | Yes |
| `BlueprintSaved` | After a blueprint is saved | No |
| `BlueprintCreating` | Before a new blueprint is created | Yes |
| `BlueprintCreated` | After a new blueprint is created | No |
| `BlueprintDeleting` | Before a blueprint is deleted | Yes |
| `BlueprintDeleted` | After a blueprint is deleted | No |
| `FieldsetSaving` | Before a fieldset is saved | Yes |
| `FieldsetSaved` | After a fieldset is saved | No |
| `FieldsetCreating` | Before a new fieldset is created | Yes |
| `FieldsetCreated` | After a new fieldset is created | No |
| `FieldsetDeleting` | Before a fieldset is deleted | Yes |
| `FieldsetDeleted` | After a fieldset is deleted | No |

### Form Events

| Event | Fired When | Can Prevent? |
|-------|-----------|-------------|
| `FormSaving` | Before a form is saved | Yes |
| `FormSaved` | After a form is saved | No |
| `FormCreating` | Before a new form is created | Yes |
| `FormCreated` | After a new form is created | No |
| `FormDeleting` | Before a form is deleted | Yes |
| `FormDeleted` | After a form is deleted | No |
| `SubmissionSaving` | Before a submission is saved | Yes |
| `SubmissionSaved` | After a submission is saved | No |
| `SubmissionCreating` | Before a submission is created | Yes |
| `SubmissionCreated` | After a submission is created | No |
| `SubmissionDeleting` | Before a submission is deleted | Yes |
| `SubmissionDeleted` | After a submission is deleted | No |
| `FormSubmitted` | When a form is submitted (front-end) | Yes |

### Search Events

| Event | Fired When |
|-------|-----------|
| `SearchIndexUpdated` | After a search index is updated |

### Cache and System Events

| Event | Fired When |
|-------|-----------|
| `StacheWarmed` | After the Stache cache is warmed |
| `UrlInvalidated` | When a URL is invalidated from static cache |
| `ResponseCreated` | After a response is created |

### Revision Events

| Event | Fired When |
|-------|-----------|
| `RevisionSaving` | Before a revision is saved |
| `RevisionSaved` | After a revision is saved |
| `RevisionDeleting` | Before a revision is deleted |
| `RevisionDeleted` | After a revision is deleted |

### Site Events

| Event | Fired When |
|-------|-----------|
| `SiteSaved` | After a site is saved |
| `SiteDeleted` | After a site is deleted |

## Preventing Actions with Events

Any `*Saving`, `*Creating`, `*Deleting`, or `*Uploading` event can be prevented by returning `false` from a listener:

```php
use Statamic\Events\EntrySaving;

class ValidateEntry
{
    public function handle(EntrySaving $event)
    {
        if ($event->entry->get('requires_approval') && ! $event->entry->get('approved')) {
            return false; // Prevents the save
        }
    }
}
```

Register listeners in `EventServiceProvider`:

```php
protected $listen = [
    \Statamic\Events\EntrySaving::class => [
        \App\Listeners\ValidateEntry::class,
    ],
];
```

## Hook Pipeline Patterns

Hooks allow intercepting and modifying data within Statamic's tag pipelines. Register hooks in a service provider's `boot()` method.

### Basic Hook Registration

```php
use Statamic\Tags\Collection;

Collection::hook('fetched-entries', function ($entries, $next) {
    // Modify the entries before they reach the template
    $filtered = $entries->reject(fn ($entry) => $entry->get('hidden'));
    return $next($filtered);
});
```

### Pipeline Rules

1. Always call `$next($value)` to continue the pipeline. Omitting it stops all downstream hooks.
2. The first argument is the current pipeline value (often a Collection of entries).
3. Multiple hooks on the same point execute in registration order.
4. Register hooks in a service provider `boot()` method, not in `register()`.

### Available Hook Points

Tag classes expose hook points for their data-fetching lifecycle. The `Collection` tag exposes `fetched-entries` after retrieving entries from the query. Other tags follow similar patterns. Consult Statamic source or addon documentation for specific hook names.

### Example: Injecting Extra Data

```php
Collection::hook('fetched-entries', function ($entries, $next) {
    $entries->each(function ($entry) {
        $entry->setSupplement('external_rating', fetchRating($entry->slug()));
    });
    return $next($entries);
});
```

## Query Scopes

Query scopes encapsulate reusable query logic into named, composable classes.

### Creating a Scope

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

### Creating a Filter Scope (Control Panel)

```php
namespace App\Scopes;

use Statamic\Query\Scopes\Filter;

class StatusFilter extends Filter
{
    public $pinned = true;

    public static function title()
    {
        return __('Status');
    }

    public function fieldItems()
    {
        return [
            'status' => [
                'type' => 'radio',
                'options' => [
                    'published' => __('Published'),
                    'draft' => __('Draft'),
                ],
            ],
        ];
    }

    public function apply($query, $values)
    {
        $query->where('status', $values['status']);
    }

    public function badge($values)
    {
        return __('Status') . ': ' . $values['status'];
    }

    public function visibleTo($key)
    {
        return $key === 'entries';
    }
}
```

### Registration

Register scopes in a service provider `boot()` method:

```php
use Statamic\Facades\Scope;

public function boot()
{
    Scope::register('featured', \App\Scopes\Featured::class);
    Scope::register('status_filter', \App\Scopes\StatusFilter::class);
}
```

### Usage in Antlers Templates

```antlers
{{ collection:blog query_scope="featured" }}
    {{ title }}
{{ /collection:blog }}
```

### Usage with Parameters

Pass parameters to scopes from templates:

```antlers
{{ collection:blog query_scope="filtered" min_words:100 }}
```

The parameters are available in the `$values` array:

```php
class Filtered extends Scope
{
    public function apply($query, $values)
    {
        if ($minWords = $values->get('min_words')) {
            $query->where('word_count', '>=', $minWords);
        }
    }
}
```

### Usage in PHP

Apply scopes programmatically in controllers or service classes:

```php
Entry::query()
    ->where('collection', 'blog')
    ->tap(function ($query) {
        app(\App\Scopes\Featured::class)->apply($query, collect());
    })
    ->get();
```
