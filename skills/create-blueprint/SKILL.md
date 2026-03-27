---
name: create-blueprint
description: This skill should be used when the user asks to "create a blueprint", "generate blueprint", "define fields", "blueprint YAML", "add fields to blueprint", or wants to generate Statamic blueprint YAML from a natural language description of content fields.
argument-hint: "product with title, price (integer), description (markdown), images (assets max 5), category (select: tech, design, other)"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
version: 0.1.0
---

# Create Statamic Blueprint

Generate blueprint YAML from a natural language field description.

## Workflow

### 1. Parse Field Descriptions

Extract fields from the user's input. Supported formats:
- `price (integer)` — field with explicit type
- `images (assets max 5)` — field with type and options
- `category (select: tech, design, other)` — field with type and values
- `content` — infer type from common field names
- `author (users, max 1)` — relationship field with constraint
- `featured (toggle, default true)` — field with default value
- `event_date (date, time enabled)` — field with options

### 2. Infer Types from Field Names

When no type is specified, infer from the handle name:

| Handle pattern | Inferred type | Options |
|---------------|---------------|---------|
| content, body, description, bio | markdown | — |
| title, name, label, heading | text | — |
| slug | slug | — |
| excerpt, summary, intro | textarea | — |
| image, photo, avatar, hero_image, logo | assets | max_files: 1 |
| images, photos, gallery | assets | — |
| featured, active, published, show_* | toggle | — |
| date, *_date, *_at | date | — |
| email | text | input_type: email |
| url, link, website | text | input_type: url |
| phone | text | input_type: tel |
| price, amount, cost | integer | — |
| color, colour | color | — |
| author | users | max_items: 1 |
| tags | terms | taxonomy: tags |
| categories, category | terms | taxonomy: categories |
| video | video | — |
| icon | icon | — |
| code, snippet | code | — |

### 3. Determine Blueprint Location

Identify where to save based on context:
- **Collection blueprint**: `resources/blueprints/collections/{collection}/{name}.yaml`
- **Taxonomy blueprint**: `resources/blueprints/taxonomies/{taxonomy}/{name}.yaml`
- **Global blueprint**: `resources/blueprints/globals/{handle}.yaml`
- **Form blueprint**: `resources/blueprints/forms/{handle}.yaml`
- **Asset blueprint**: `resources/blueprints/assets/{container}.yaml`
- **User blueprint**: `resources/blueprints/user.yaml`

Check if the target directory exists. Create it if not.

### 4. Organize into Sections

Group fields into logical sections:
- **main**: Primary content fields (content, description, body)
- **sidebar**: Metadata fields (author, date, tags, categories, status)
- **media**: Image and asset fields
- **seo**: SEO-related fields (meta_title, meta_description, og_image)

If fewer than 5 fields total, use a single `main` section.

### 5. Generate Blueprint YAML

```yaml
sections:
  main:
    display: Main
    fields:
      - handle: content
        field:
          type: markdown
          display: Content
      - handle: featured_image
        field:
          type: assets
          display: Featured Image
          container: assets
          max_files: 1
  sidebar:
    display: Sidebar
    fields:
      - handle: author
        field:
          type: users
          display: Author
          max_items: 1
      - handle: tags
        field:
          type: terms
          display: Tags
          taxonomy: tags
```

### 6. Field Configuration Patterns

**Assets field**:
```yaml
- handle: images
  field:
    type: assets
    display: Images
    container: assets
    max_files: 5
    mode: list
```

**Select field with options**:
```yaml
- handle: category
  field:
    type: select
    display: Category
    options:
      tech: Technology
      design: Design
      other: Other
    clearable: true
```

**Bard (rich text) field**:
```yaml
- handle: content
  field:
    type: bard
    display: Content
    buttons:
      - h2
      - h3
      - bold
      - italic
      - unorderedlist
      - orderedlist
      - quote
      - link
      - image
    container: assets
    save_html: false
```

**Grid field**:
```yaml
- handle: features
  field:
    type: grid
    display: Features
    fields:
      - handle: title
        field: { type: text, display: Title }
      - handle: description
        field: { type: textarea, display: Description }
      - handle: icon
        field: { type: icon, display: Icon }
    min_rows: 1
    max_rows: 10
```

**Replicator field**:
```yaml
- handle: page_builder
  field:
    type: replicator
    display: Page Builder
    sets:
      content_blocks:
        display: Content Blocks
        sets:
          text:
            display: Text Block
            fields:
              - handle: text
                field: { type: bard, display: Text }
          image:
            display: Image Block
            fields:
              - handle: image
                field: { type: assets, display: Image, max_files: 1 }
              - handle: caption
                field: { type: text, display: Caption }
```

**Conditional visibility**:
```yaml
- handle: venue
  field:
    type: text
    display: Venue
    if:
      event_type: in_person
    validate: [sometimes, required]
```

**Fieldset reference**:
```yaml
fields:
  - import: common
    prefix: post_
  - handle: category
    field: common.category
    config:
      display: Post Category
```

### 7. Validation Rules

Add validation where appropriate:
- Required fields: `validate: required`
- Email: `validate: 'required|email'`
- URL: `validate: 'nullable|url'`
- Min/max: `validate: 'min:3|max:255'`
- Unique: `validate: 'new \Statamic\Rules\UniqueEntryValue({collection}, {id}, {site})'`

### 8. Summary

After creating the blueprint, display:
- File path of the created blueprint
- List of fields with their types
- Any validation rules applied
- How to reference fieldset fields if applicable

## Common Blueprint Templates

Use these as starting points:
- **Blog post**: content (markdown), featured_image (assets:1), excerpt (textarea), author (users:1), tags (terms)
- **Page**: content (bard), hero_image (assets:1), meta_title (text), meta_description (textarea)
- **Product**: description (bard), price (integer), images (assets), sku (text), in_stock (toggle)
- **Team member**: bio (markdown), photo (assets:1), role (text), email (text), social_links (grid)
- **Event**: description (markdown), start_date (date+time), end_date (date+time), location (text), registration_url (text)
