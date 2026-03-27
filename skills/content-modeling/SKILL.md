---
name: content-modeling
description: >-
  This skill should be used when the user asks to "create a collection",
  "configure taxonomy", "define blueprint", "add fieldset", "content modeling",
  "collection config", "blueprint YAML", "set up globals", "configure navigation",
  "add field validation", or works with Statamic fieldtypes and content structures.
version: 0.1.0
---

# Statamic Content Modeling

## Collections

Collections hold groups of related entries (blog posts, pages, products). Each entry is a Markdown file with YAML front matter stored in `content/collections/{collection}/`. A Statamic site typically has multiple collections -- one for pages, one for blog posts, one for products, etc. Each collection has its own configuration, blueprints, and routing rules.

### Collection config

Store collection configuration at `content/collections/{collection}.yaml`.

```yaml
title: Blog
date: true
date_behavior:
  past: public
  future: private
route: 'blog/{slug}'
sort_dir: desc
template: blog/show
layout: layout
taxonomies:
  - tags
  - categories
inject:
  author: default-author
  show_sidebar: true
revisions: true
search_index: blog
preview_targets:
  - label: Entry
    url: /blog/{slug}
  - label: Index
    url: /blog
```

Available collection options: `title`, `date` (bool -- enables date-based entries), `date_behavior.past/future` (`public`/`unlisted`/`private` -- controls visibility of entries based on their date), `route` (URI pattern for entries), `sort_dir` (`asc`/`desc`), `sort_by` (field handle to sort by, defaults to `order` for orderable or `date` for dated collections), `template` (default template), `layout` (default layout), `inject` (default data merged into every entry), `slugs` (`false` to disable slug requirement), `max_depth` (for structured/tree collections), `mount` (entry ID whose URL becomes the collection base), `taxonomies` (array of taxonomy handles), `title_format` (pattern for auto-generated titles like `{first_name} {last_name}`), `revisions` (bool -- enable revision history), `search_index` (name of search index), `preview_targets` (array of label/URL pairs for live preview).

### Route variables

Use these placeholders in the `route` option: `{slug}`, `{year}`, `{month}`, `{day}`, `{parent_uri}`, `{depth}`, `{mount}`, plus any entry field handle as `{field_handle}`. Examples: `blog/{year}/{month}/{slug}` for date-organized URLs, `{parent_uri}/{slug}` for nested page structures, `products/{category}/{slug}` using a custom field.

### Entry files

Name entry files as `content/collections/{collection}/{date}.{slug}.md` (date-based) or `{slug}.md` (non-dated).

```yaml
---
title: My Blog Post
id: 3a28f050-f8d2-4a56-ba8a-314a9d46bf38
author: john
tags:
  - tech
  - news
---
The post content goes here in **Markdown**.
```

Every entry must have a unique `id` (UUID). Statamic generates these automatically via the Control Panel; when creating entries by hand, generate a UUID v4 and include it. The filename date prefix (e.g., `2024-01-15`) is only used for dated collections and determines the entry's date. For non-dated collections, omit the date prefix entirely. Content below the closing `---` is the entry's Markdown body, accessible via the `content` variable in templates.

### Entry statuses

- **published** -- visible on the front end (default)
- **draft** -- set `published: false` in front matter
- **scheduled** -- future date with `date_behavior.future: private`
- **expired** -- past date with past behavior set to `private`

## Taxonomies

Taxonomies classify content with terms that are global across collections. Unlike collections, taxonomy terms are shared -- a "tech" tag created in one collection is the same "tech" tag in another. Assign taxonomy values to entries by adding the taxonomy handle as a field in the entry front matter (e.g., `tags: [tech, news]`).

### Taxonomy config

Store at `content/taxonomies/{taxonomy}.yaml`:

```yaml
title: Tags
inject:
  foo: bar
```

### Assigning taxonomies to collections

Add the `taxonomies` array in the **collection** YAML config (not in the blueprint):

```yaml
taxonomies:
  - tags
  - categories
```

### Term data

Store term data at `content/taxonomies/{taxonomy}/{term-slug}.yaml`:

```yaml
title: My Term
description: A custom term description
```

### Taxonomy routes

Statamic creates routes automatically only when a corresponding view exists:

- Taxonomy index: `/{taxonomy}` -- view: `{taxonomy}/index.antlers.html`
- Term detail: `/{taxonomy}/{term}` -- view: `{taxonomy}/show.antlers.html`
- Collection-scoped: `/{collection-url}/{taxonomy}/{term}` -- view: `{collection}/{taxonomy}/show.antlers.html`

## Globals

Globals provide site-wide variables available in all templates. Use globals for content that appears on every page -- footer text, social media links, site-wide banners, company information, and similar cross-cutting data. Each global set has its own blueprint defining its fields.

### Global set metadata

Store at `content/globals/{handle}.yaml`:

```yaml
title: Footer
```

### Global data

Store the actual values at `content/globals/default/{handle}.yaml`:

```yaml
copyright: "2024 My Company"
flair: "Made with love"
```

### Template access

- Antlers: `{{ footer:copyright }}`
- Blade: `{{ $footer->copyright }}`

## Navigation

Hierarchical link structures for menus and navigation.

### Nav config

Store at `content/navigation/{nav}.yaml`:

```yaml
title: Main Navigation
max_depth: 3
collections:
  - pages
  - posts
```

### Nav tree

Store the tree structure at `content/trees/navigation/{nav}.yaml`. Each node is a YAML object with keys: `entry` (entry ID for linking to an existing entry), `url` (for manual/external links), `title` (override the linked entry's title or set text for manual links), and `children` (nested array of child nodes for multi-level menus). The `collections` option in the nav config restricts which collections editors can pick entries from in the Control Panel.

## Blueprints

Blueprints define the field structure for content types. Store as YAML in `resources/blueprints/`. A blueprint controls which fields appear in the Control Panel publish form, how they are organized into sections/tabs, and what validation rules apply. Each collection can have multiple blueprints (e.g., a "post" blueprint and a "video_post" blueprint within the same blog collection), allowing different field layouts for different content shapes.

### Blueprint structure

Organize fields into named sections. Each section maps to a tab in the Control Panel publish form. Each section has a `display` name and a `fields` array. Within the fields array, each field is defined inline (with `handle` and `type`) or referenced from a fieldset (with `handle` and `field`):

```yaml
# resources/blueprints/collections/blog/post.yaml
sections:
  main:
    display: Main
    fields:
      - handle: content
        type: markdown
      - handle: featured_image
        type: assets
        container: assets
        max_files: 1
  sidebar:
    display: Sidebar
    fields:
      - handle: author
        type: users
        max_items: 1
      - handle: category
        field: common.category
        config:
          validate: required
```

### Blueprint locations

Place blueprint files according to the content type:

- **Collections**: `resources/blueprints/collections/{collection}/{blueprint}.yaml`
- **Taxonomies**: `resources/blueprints/taxonomies/{taxonomy}/{blueprint}.yaml`
- **Globals**: `resources/blueprints/globals/{handle}.yaml`
- **Assets**: `resources/blueprints/assets/{container}.yaml`
- **Forms**: `resources/blueprints/forms/{handle}.yaml`
- **Users**: `resources/blueprints/user.yaml`

### Field references in blueprints

Reference a single field from a fieldset:

```yaml
- handle: my_field
  field: common.category      # fieldset_name.field_handle
  config:
    display: "Post Category"  # override any config option
```

## Fieldsets (Reusable Field Groups)

Fieldsets allow defining a group of fields once and reusing them across multiple blueprints. This avoids duplicating field definitions and ensures consistency. When a fieldset is updated, every blueprint that imports it picks up the change.

Define reusable field groups at `resources/fieldsets/{name}.yaml`:

```yaml
# resources/fieldsets/common.yaml
fields:
  - handle: category
    type: select
    options:
      tech: Technology
      design: Design
  - handle: featured
    type: toggle
```

### Importing fieldsets into blueprints

Import all fields from a fieldset, optionally with a handle prefix (useful when importing the same fieldset multiple times to avoid handle collisions):

```yaml
fields:
  - import: common
    prefix: post_
```

Import a single field with config overrides:

```yaml
fields:
  - handle: my_field
    field: common.category
    config:
      display: "Post Category"
```

## Field Common Options

Every fieldtype supports these universal options:

| Option | Values | Purpose |
|--------|--------|---------|
| `display` | string | Label shown in the Control Panel |
| `handle` | string | Variable name used in templates |
| `instructions` | string (supports Markdown) | Help text for content editors |
| `instructions_position` | `above` / `below` | Position of the help text |
| `listable` | `true` / `false` / `hidden` | Visibility in CP listing tables |
| `visibility` | `visible` / `read_only` / `computed` / `hidden` | Publish form field visibility |
| `validate` | array | Laravel validation rules |
| `required` | bool | Shortcut for adding `required` validation |
| `if` / `unless` | object | Conditional visibility rules based on sibling field values |
| `always_save` | bool | Force saving the field value even when hidden by conditions |

## Validation

Apply Laravel validation rules to any field via the `validate` option. All standard Laravel validation rules are supported (`required`, `email`, `url`, `min`, `max`, `regex`, `unique`, etc.), plus Statamic provides custom rules like `UniqueEntryValue`:

```yaml
- handle: slug
  field:
    type: slug
    validate:
      - required
      - alpha_dash
      - 'min:4'
      - 'new \Statamic\Rules\UniqueEntryValue({collection}, {id}, {site})'
```

For nested fields inside Grid or Replicator, reference sibling fields with the `{this}` prefix:

```yaml
validate: 'required_with:{this}.purchasable'
```

## Computed Values

Define virtual fields in PHP that are evaluated at runtime. Computed values are not stored in files -- they are calculated on-the-fly when the entry is accessed. This is useful for derived data like read time, full names, or aggregated counts. Register computed values in the `boot()` method of a service provider:

```php
// app/Providers/AppServiceProvider.php boot()
use Statamic\Facades\Collection;

Collection::computed('articles', 'read_time', function ($entry, $value) {
    return ceil(str_word_count(strip_tags($entry->content)) / 200);
});
```

Set the field's `visibility` to `computed` in the blueprint to display it as read-only in the CP.

## Data Inheritance Cascade

Statamic resolves values in this priority order:

**Partial > ViewModel > Entry > Origin Entry > Collection inject > Global > System > null**

Use the null-coalescence pattern to fall through the cascade:

```
{{ meta_title ?? breadcrumb_title ?? title }}
```

The `inject` option on collections and taxonomies inserts default values that entries inherit unless they define their own value for that field. Understanding the cascade is essential for designing content models that minimize redundancy -- place shared defaults at the collection level via `inject`, use globals for truly site-wide values, and let entries override only what differs.

## Content Modeling Patterns

### Choosing between collections and taxonomies

Use collections for primary content types that have their own pages and URLs (articles, products, events). Use taxonomies for classification systems that group content across collections (tags, categories, topics). A taxonomy term does not need its own page -- Statamic only generates taxonomy routes when corresponding views exist.

### Structuring blueprints with fieldsets

Extract common field groups (SEO fields, social sharing metadata, author information) into fieldsets. Import them into multiple blueprints with the `import` directive. Use the `prefix` option when importing the same fieldset more than once in a single blueprint to keep handles unique. Override individual field settings via the `config` block when a specific blueprint needs different validation or display.

### Conditional fields

Use `if` and `unless` on fields to show or hide them based on sibling field values. The conditions reference field handles and expected values:

```yaml
- handle: event_url
  type: text
  if:
    type: equals event
- handle: product_price
  type: float
  unless:
    is_free: equals true
```

Fields hidden by conditions are still saved unless `always_save: false` is set. Use `always_save: true` to ensure values persist even when the field is not visible.

## Fieldtypes Reference

Statamic ships with 46 built-in fieldtypes across seven categories: Text/Content, Structured, Selection, Number/Date, Media, Relationships, and Special. See [references/fieldtypes.md](references/fieldtypes.md) for the complete table with storage formats and key options for each fieldtype.
