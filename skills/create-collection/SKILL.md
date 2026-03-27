---
name: create-collection
description: This skill should be used when the user asks to "create a collection", "scaffold a collection", "add a new collection", "set up blog collection", "create pages collection", or wants to generate Statamic collection configuration files, blueprints, and templates together as a package.
argument-hint: "blog with dates, tags taxonomy, route /blog/{slug}"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Bash
  - Grep
version: 0.1.0
---

# Create Statamic Collection

Scaffold a complete Statamic collection with configuration, blueprint, and Antlers templates in one step.

## Workflow

### 1. Parse the Request

Extract from the user's description:
- **Collection handle** (kebab-case, e.g., `blog-posts`)
- **Title** (human-readable, e.g., "Blog Posts")
- **Date-enabled** (default: false unless "blog", "posts", "articles", or "events" in name, or user mentions dates)
- **Route pattern** (default: `/{collection-handle}/{slug}`)
- **Taxonomies** (any mentioned, e.g., tags, categories)
- **Template name** (default: `{collection-handle}/show`)
- **Sort direction** (default: `desc` if date-enabled, `asc` otherwise)
- **Blueprint fields** (any content fields mentioned)

If critical details are ambiguous, infer sensible defaults rather than asking — collections are easy to adjust later.

### 2. Verify Project Structure

Before creating files, confirm the project is a Statamic project:
- Check for `content/collections/` directory
- Check for `resources/blueprints/collections/` directory
- Check for `resources/views/` directory

If directories are missing, warn the user and ask whether to create them.

### 3. Create Collection Config

Write to `content/collections/{handle}.yaml`:

```yaml
title: {Title}
date: {true|false}
date_behavior:
  past: public
  future: private
route: '{route-pattern}'
sort_dir: {asc|desc}
template: {handle}/show
layout: layout
taxonomies:
  - {taxonomy1}
revisions: false
```

Only include `date`, `date_behavior`, and `sort_dir` if date-enabled. Only include `taxonomies` if specified. Only include `revisions` if the user mentions it.

### 4. Create Blueprint

Write to `resources/blueprints/collections/{handle}/{handle}.yaml`:

```yaml
sections:
  main:
    display: Main
    fields:
      - handle: content
        type: markdown
  sidebar:
    display: Sidebar
    fields:
      - handle: author
        type: users
        max_items: 1
```

Adjust fields based on user request. Common patterns:
- Blog: content (markdown), featured_image (assets), excerpt (textarea)
- Pages: content (bard or markdown), hero_image (assets)
- Products: description (bard), price (integer), images (assets), category (select/taxonomy)
- Events: description (markdown), date (date), location (text), registration_url (text)

### 5. Create Templates

Write the **show template** to `resources/views/{handle}/show.antlers.html`:

```antlers
{{ partial:hero title="{title}" }}

<article>
  <h1>{{ title }}</h1>
  {{ if date }}<time datetime="{{ date format="Y-m-d" }}">{{ date format="F j, Y" }}</time>{{ /if }}
  {{ if featured_image }}
    {{ glide:featured_image width="800" height="400" fit="crop" }}
      <img src="{{ url }}" alt="{{ alt ?? title }}">
    {{ /glide:featured_image }}
  {{ /if }}
  <div class="content">
    {{ content }}
  </div>
</article>
```

Write the **index template** to `resources/views/{handle}/index.antlers.html`:

```antlers
<h1>{{ title ?? '{Title}' }}</h1>

{{ collection:{handle} limit="10" sort="date:desc" }}
  <article>
    <h2><a href="{{ url }}">{{ title }}</a></h2>
    {{ if date }}<time>{{ date format="M d, Y" }}</time>{{ /if }}
    {{ if excerpt }}<p>{{ excerpt }}</p>{{ /if }}
  </article>
{{ /collection:{handle} }}
```

Adapt templates to include only fields that exist in the blueprint.

### 6. Create Initial Entry (optional)

If the user requests it, create a sample entry at `content/collections/{handle}/{filename}.md`:

```markdown
---
title: Sample Entry
id: {generate-uuid}
---
Sample content goes here.
```

For date-enabled collections, prefix filename with date: `2026-01-01.sample-entry.md`.

### 7. Summary

After creating all files, display:
- List of created files with paths
- The collection route URL
- How to access it in templates: `{{ collection:{handle} }}`
- Reminder to create the layout if it doesn't exist

## Defaults Reference

| Collection type | Date | Route | Sort | Default fields |
|----------------|------|-------|------|----------------|
| blog/posts | yes | /blog/{slug} | desc | content, featured_image, excerpt, author |
| pages | no | /{slug} | asc | content, hero_image |
| products | no | /products/{slug} | asc | description, price, images, category |
| events | yes | /events/{slug} | asc | description, date, location |
| team | no | /team/{slug} | asc | bio, photo, role, email |
| projects | no | /projects/{slug} | desc | description, images, client, url |

## Key Conventions

- Collection handle: kebab-case, plural preferred (e.g., `blog-posts`)
- Blueprint filename matches collection handle
- Templates in subdirectory matching collection handle
- Use `inject` in collection config for default values shared across entries
- Route variables available: `{slug}`, `{year}`, `{month}`, `{day}`, `{parent_uri}`, `{depth}`, `{mount}`, plus any entry field
