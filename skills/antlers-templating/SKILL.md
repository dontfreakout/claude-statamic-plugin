---
name: antlers-templating
description: >-
  This skill should be used when the user works with ".antlers.html" files, writes
  Antlers template code, uses Antlers tags (collection, nav, partial, form, taxonomy,
  search, glide, cache), applies modifiers with pipe syntax, builds layouts/templates/partials,
  writes conditionals and loops in Antlers, or integrates Blade with Statamic via
  the "s:" prefix, Fluent Tags, or "@antlers" directive.
version: 0.1.0
---

# Antlers Templating

## File Conventions

- Use the `.antlers.html` extension for all Antlers templates (required for the parser to engage). Files without this double extension are treated as plain HTML and will not process Antlers syntax.
- Place all view files under `resources/views/`. Subdirectories map to partial and template references using slash notation (e.g., `resources/views/blog/card.antlers.html` is referenced as `partial:blog/card`).
- Use `.blade.php` for Blade-based Statamic templates as an alternative. Both engines can coexist in the same project.

## Delimiters

```antlers
{{ variable }}              {{-- Render variables and tags --}}
{{# This is a comment #}}   {{-- Code comment, not rendered --}}
{{? $php = "code"; ?}}       {{-- Execute raw PHP --}}
{{$ echo $result; $}}        {{-- Echo a PHP expression --}}
```

## Variables and Data Access

Antlers automatically receives data from the current page's entry, global sets, the view cascade, and any injected variables. Access entry data, globals, and cascaded variables using these notations:

```antlers
{{ title }}                   {{-- Simple variable --}}
{{ mailing_address:city }}    {{-- Nested via colon notation --}}
{{ mailing_address.city }}    {{-- Nested via dot notation --}}
{{ mailing_address['city'] }} {{-- Nested via bracket notation --}}
{{ sports:0 }}                {{-- Array index access --}}
{{ $content }}                {{-- Prefix with $ to disambiguate from a tag --}}
```

## Looping

Loop over arrays, collections, and query results by wrapping the variable in a tag pair:

```antlers
{{ blog_posts }}
  <h2>{{ title }}</h2>
  <p>{{ content }}</p>
  {{ if first }}First item!{{ /if }}
  {{ if last }}Last item!{{ /if }}
  Next: {{ next:title }}
  Prev: {{ prev:title }}
{{ /blog_posts }}
```

Loop context variables available inside every loop: `first`, `last`, `count`, `index`, `total_results`, `next`, `prev`.

## Conditionals

```antlers
{{ if sold }}
  Sold out!
{{ elseif quantity < 5 }}
  Only {{ quantity }} left!
{{ else }}
  In stock
{{ /if }}

{{ unless logged_in }}
  Please log in.
{{ /unless }}
```

### Ternary, Null Coalescence, Gatekeeper, Switch

```antlers
{{ is_sold ? "Sold" : "Available" }}
{{ meta_title ?? title ?? "Default" }}
{{ show_bio ?= author:bio }}

{{ switch(
  (size == 'sm') => '35vw',
  (size == 'lg') => '75vw',
  () => '100vw'
) }}
```

## Operators

- **Comparison**: `==`, `===`, `!=`, `!==`, `>`, `>=`, `<`, `<=`, `<=>`
- **Logical**: `&&` / `and`, `||` / `or`, `!`, `xor`
- **Math**: `+`, `-`, `*`, `/`, `%`, `**`
- **String concatenation**: `+`
- **Assignment**: `=`, `+=`, `-=`, `*=`, `/=`, `%=`

## Modifier Pipe Syntax

Chain modifiers with the pipe `|` character. Pass arguments in parentheses:

```antlers
{{ title | upper }}
{{ title | ensure_right('!') }}
{{ content | markdown | truncate(200, '...') }}
{{ price | format_number(2) }}
{{ items | where('active', true) | count }}
{{ date | format('F j, Y') }}
{{ content | read_time }}
{{ image | glide:width(300):height(200) }}
```

Common modifiers by purpose:

| Purpose | Modifiers |
|---|---|
| Case | `upper`, `lower`, `title`, `ucfirst`, `slugify` |
| Truncation | `truncate`, `safe_truncate`, `excerpt`, `backspace` |
| Markup | `markdown`, `nl2br`, `strip_tags`, `sanitize`, `widont` |
| Arrays | `count`, `first`, `last`, `sort`, `where`, `pluck`, `limit`, `offset`, `unique`, `flatten`, `group_by`, `sum` |
| Dates | `format`, `relative`, `modify_date`, `days_ago`, `is_past`, `is_future` |
| Math | `add`, `subtract`, `multiply`, `divide`, `round`, `ceil`, `floor`, `format_number` |
| Utility | `to_json`, `raw`, `dump`, `partial`, `read_time`, `word_count` |

For the complete list of approximately 150 modifiers, consult `references/modifiers.md`.

## Advanced Array Operations

```antlers
{{-- Merge --}}
{{ articles = favourite merge not_favourite }}

{{-- Sort --}}
{{ people orderby (age 'desc', name 'asc') }}

{{-- Group --}}
{{ items = posts groupby (category) }}
  <h2>{{ key }}</h2>
  {{ values }} <li>{{ title }}</li> {{ /values }}
{{ /items }}

{{-- Filter --}}
{{ affordable = products where (price < 100) }}

{{-- Take / Skip --}}
{{ featured = items take (3) }}
{{ rest = items skip (3) }}

{{-- Pluck --}}
{{ names = users pluck ('name') }}
```

## Creating Variables

```antlers
{{ total = 0 }}
{{ total += 1 }}
{{ items = {collection:products sort="price" limit="5"} }}
{{ todo = ['Get haircut', 'Bake bread'] }}
```

## Tags

Tags fetch data and perform actions. Use the tag pair pattern for loops and self-closing tags for single values.

### Collection

```antlers
{{ collection:blog limit="10" sort="date:desc" }}
  <h2><a href="{{ url }}">{{ title }}</a></h2>
  <p>{{ date format="M d, Y" }}</p>
{{ /collection:blog }}
```

Apply conditions to filter entries. See `references/tags-and-conditions.md` for the full condition operator list and taxonomy filtering syntax.

```antlers
{{ collection:blog taxonomy:tags="featured" author:is="john" status:is="published" }}
```

### Navigation

```antlers
{{ nav:main }}
  <li><a href="{{ url }}">{{ title }}</a>
    {{ if children }}
      <ul>{{ *recursive children* }}</ul>
    {{ /if }}
  </li>
{{ /nav:main }}
```

### Assets

```antlers
{{ assets container="images" limit="10" sort="date" }}
  <img src="{{ url }}" alt="{{ alt }}">
{{ /assets }}
```

### Glide (Image Manipulation)

```antlers
{{ glide:hero_image width="800" height="400" fit="crop" quality="80" }}
```

### Forms

```antlers
{{ form:contact }}
  {{ if errors }}<div class="error">{{ errors }}{{ value }}{{ /errors }}</div>{{ /if }}
  {{ if success }}<p>{{ success }}</p>{{ /if }}
  {{ fields }}
    <label>{{ display }}</label>
    {{ field }}
  {{ /fields }}
  <button type="submit">Send</button>
{{ /form:contact }}
```

### Search

```antlers
{{ search:results index="default" }}
  {{ if no_results }}<p>No results for {{ get:q }}</p>
  {{ else }}<a href="{{ url }}">{{ title }}</a>{{ /if }}
{{ /search:results }}
```

### Taxonomy

```antlers
{{ taxonomy:tags }}
  <a href="{{ url }}">{{ title }}</a> ({{ entries_count }})
{{ /taxonomy:tags }}
```

### Partials

```antlers
{{ partial:blog/card }}
{{ partial:hero title="Welcome" :image="hero_image" }}
```

Prefix a parameter with `:` to pass a variable by reference rather than a literal string.

### Sections and Yields

```antlers
{{ section:sidebar }}Sidebar content{{ /section:sidebar }}
{{ yield:sidebar }}
```

### Cache and Nocache

```antlers
{{ cache for="1 hour" }}
  Expensive content here
{{ /cache }}

{{ nocache }}Dynamic content inside static cache{{ /nocache }}
```

### User Checks

```antlers
{{ user:can permission="edit blog entries" }}...{{ /user:can }}
{{ if logged_in }}Welcome {{ current_user:name }}{{ /if }}
```

## Layouts, Templates, and Partials

Define a layout at `resources/views/layout.antlers.html`:

```antlers
<html>
<head><title>{{ title }} | {{ site_name }}</title></head>
<body>
  {{ partial:nav }}
  {{ template_content }}
  <footer>&copy; {{ now format="Y" }} {{ site_name }}</footer>
</body>
</html>
```

Use `{{ template_content }}` in layouts to inject the template body. Use `{{ partial:name }}` to include reusable partials.

### Stacks

Push assets to named stacks and render them in the layout:

```antlers
{{ stack:scripts }}
{{ push:scripts }}<script src="/app.js"></script>{{ /push:scripts }}
```

## System Variables

These variables are always available in every template:

`current_url`, `current_uri`, `environment`, `csrf_token`, `csrf_field`, `logged_in`, `current_user`, `now`, `site`, `sites`, `homepage`, `is_homepage`, `get` (query params), `post`, `old`, `last_segment`, `segment_1` through `segment_n`, `response_code`, `xml_header`, `live_preview`, `config`

## Prevent Parsing

Escape double curly braces or wrap blocks to prevent Antlers from processing them:

```antlers
@{{ not_parsed }}
{{ noparse }}{{ also_not_parsed }}{{ /noparse }}
```

Use `@` prefix for single expressions (useful when outputting Vue.js or Alpine.js templates). Use the `noparse` tag pair for larger blocks.

## Blade Integration

When using `.blade.php` templates with Statamic, access tags and modifiers through these patterns:

```blade
{{-- Antlers Blade Components (s: prefix) --}}
<s:collection:blog limit="5" sort="date:desc">
  <h2>{{ $title }}</h2>
</s:collection:blog>

{{-- Fluent Tags --}}
@foreach(Statamic::tag('collection:blog')->limit(3) as $post)
  <li>{{ $post->title }}</li>
@endforeach

{{-- Modifiers --}}
{{ Statamic::modify($content)->striptags()->truncate([100, '...']) }}

{{-- @antlers directive for inline Antlers --}}
@antlers
  {{ nav:main }}
    <a href="{{ url }}">{{ title }}</a>
  {{ /nav:main }}
@endantlers

{{-- @cascade directive --}}
@cascade(['site', 'page' => null])
```

Use `<s:tag_name>` Blade components for tag pairs. Use `Statamic::tag()` fluent API for programmatic access. Use `Statamic::modify()` for applying modifiers in Blade. Use `@antlers` / `@endantlers` to embed Antlers syntax directly inside Blade views.
