# Statamic Fieldtypes Reference

All 46 built-in fieldtypes organized by category.

## Text/Content

| Fieldtype | Storage Format | Key Options |
|-----------|---------------|-------------|
| **Text** | `string` | `placeholder`, `input_type` (text/email/url/tel/etc.), `character_limit`, `prepend`, `append` |
| **Textarea** | `string` | `placeholder`, `character_limit` |
| **Slug** | `string` | Auto-generates URL-safe slugs from another field |
| **Code** | `string` | `language`, `theme`, `line_numbers` -- syntax-highlighted code editor |
| **Hidden** | `any` | Not displayed in the Control Panel; stores arbitrary values |
| **Bard** | ProseMirror JSON (augmented to HTML) | `buttons`, `sets` (for block content), `container` (asset container), `save_html` (bool), `inline` (bool), `toolbar_mode` (fixed/floating) |
| **Markdown** | Markdown string (augmented to HTML) | `container`, `automatic_line_breaks`, `smartypants`, `parser` |
| **Replicator** | array of typed sets | `sets` (define set groups and their fields), `max_sets`, `button_label` |

## Structured

| Fieldtype | Storage Format | Key Options |
|-----------|---------------|-------------|
| **Grid** | array of rows | `fields` (column definitions), `min_rows`, `max_rows`, `mode` (`table` / `stacked`) |
| **Group** | grouped object | Container for organizing related fields into a single nested object |
| **Table** | array of rows/cols | Tabular data management with spreadsheet-like UI |
| **Array** | key/value pairs | `keys`, `mode` (`keyed` / `single` / `dynamic`) |
| **List** | simple array | Reorderable list of string items |
| **YAML** | array (editable as raw YAML) | Raw YAML editing textarea |

## Selection

| Fieldtype | Storage Format | Key Options |
|-----------|---------------|-------------|
| **Select** | `string` (single) or `array` (multiple) | `options`, `multiple` (bool), `searchable`, `taggable`, `clearable` |
| **Button Group** | `string` | `options` -- radio-like button selection UI |
| **Checkboxes** | `array` | `options`, `inline` (bool) |
| **Radio** | `string` | `options`, `inline` (bool) |
| **Dictionary** | key string(s) | `dictionary` type (`File` / `Countries` / `Timezones` / `Currencies`), `max_items` |
| **Tags** | `array` of strings | Free-form tagging input |
| **Toggle** | `boolean` | `inline_label`, `default` (bool) |
| **Revealer** | none (UI only) | Shows/hides other fields conditionally; child field values are still submitted |

## Number/Date

| Fieldtype | Storage Format | Key Options |
|-----------|---------------|-------------|
| **Integer** | `int` | Whole number input |
| **Float** | `float` | Decimal number input |
| **Range** | `number` | `min`, `max`, `step` -- slider UI |
| **Date** | date string | `mode` (`single` / `range`), `format`, `time_enabled` (bool), `earliest_date`, `latest_date` |
| **Time** | time string | Time picker widget |

## Media

| Fieldtype | Storage Format | Key Options |
|-----------|---------------|-------------|
| **Assets** | `array` of paths (`string` when `max_files: 1`) | `container`, `folder`, `max_files`, `mode` (`list` / `grid`), `allow_uploads` (bool) |
| **Video** | URL string | YouTube/Vimeo embed URL |
| **Color** | hex string | `swatches`, `allow_any` (bool), `color_modes` |
| **Icon** | `string` | Icon selector from configured icon sets |

## Relationships

All relationship fieldtypes store IDs or handles and are augmented to full objects at render time. Shared options: `max_items`, `mode` (`default` / `select` / `typeahead`).

| Fieldtype | Storage Format | Notes |
|-----------|---------------|-------|
| **Entries** | entry ID(s) | Reference entries from any collection |
| **Terms** | term ID(s) | Reference taxonomy terms |
| **Users** | user ID(s) | Reference user accounts |
| **Collections** | collection handle(s) | Reference collection definitions |
| **Taxonomies** | taxonomy handle(s) | Reference taxonomy definitions |
| **Navs** | nav handle(s) | Reference navigation structures |
| **Structures** | structure handle(s) | Reference page structures |
| **Sites** | site handle(s) | Reference configured sites |
| **User Groups** | group handle(s) | Reference user groups |
| **User Roles** | role handle(s) | Reference user roles |
| **Form** | form handle(s) | Reference form definitions |

## Special

| Fieldtype | Storage Format | Key Options |
|-----------|---------------|-------------|
| **Template** | `string` | Select from available view templates |
| **Link** | URL string | URL input with entry-linking support |
| **Spacer** | none (UI only) | Visual spacing in the publish form |
| **HTML** | none (UI only) | Render raw HTML in the publish form |
| **Width** | none (layout) | Control field width in the publish form layout |
