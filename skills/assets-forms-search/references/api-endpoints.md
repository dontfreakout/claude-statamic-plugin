# API Endpoints and Glide Parameter Reference

## REST API Endpoints

Enable: `STATAMIC_API_ENABLED=true` in `.env`. All endpoints are read-only GET requests prefixed with `/api/`.

### Endpoint Table

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/collections/{collection}/entries` | List all entries in a collection |
| GET | `/api/collections/{collection}/entries/{id}` | Retrieve a single entry by ID |
| GET | `/api/collections/{collection}/tree` | Get collection tree structure |
| GET | `/api/taxonomies/{taxonomy}/terms` | List all terms in a taxonomy |
| GET | `/api/taxonomies/{taxonomy}/terms/{slug}` | Retrieve a single term by slug |
| GET | `/api/globals` | List all global sets |
| GET | `/api/globals/{handle}` | Retrieve a single global set |
| GET | `/api/navs/{nav}/tree` | Get navigation tree |
| GET | `/api/users` | List all users |
| GET | `/api/assets/{container}` | List assets in a container |
| GET | `/api/forms/{handle}` | Get form details and configuration |

### Filtering

Apply filters using the `filter` query parameter with bracket syntax:

```
?filter[field:operator]=value
```

Examples:

```
/api/collections/blog/entries?filter[title:contains]=awesome
/api/collections/blog/entries?filter[status]=published
/api/collections/blog/entries?filter[title:contains]=awesome&filter[status]=published
```

Available filter operators:
- `is` (default when no operator specified) -- exact match
- `contains` -- partial string match
- `not` -- negation
- Field-specific operators depending on fieldtype

### Sorting

Use the `sort` query parameter. Prefix a field with `-` for descending order. Separate multiple sort fields with commas.

```
?sort=title                  # Ascending by title
?sort=-date                  # Descending by date
?sort=-date,title            # Descending date, then ascending title
```

### Field Selection

Limit the response to specific fields with the `fields` parameter:

```
?fields=id,title,content
```

Only the listed fields (plus system fields like `id`) appear in the response.

### Pagination

Control pagination with `limit` and `page`:

```
?limit=10&page=2
```

The response includes a `meta` object with pagination details:

```json
{
  "data": [...],
  "meta": {
    "total": 47,
    "count": 10,
    "per_page": 10,
    "current_page": 2,
    "total_pages": 5
  },
  "links": {
    "first": "/api/collections/blog/entries?page=1",
    "last": "/api/collections/blog/entries?page=5",
    "prev": "/api/collections/blog/entries?page=1",
    "next": "/api/collections/blog/entries?page=3"
  }
}
```

### Combined Example

```
/api/collections/blog/entries?filter[status]=published&sort=-date&fields=id,title,date,excerpt&limit=10&page=1
```

## GraphQL API

Enable: `STATAMIC_GRAPHQL_ENABLED=true` in `.env`. GraphiQL IDE is accessible from the Control Panel.

### Available Queries

| Query | Arguments | Description |
|-------|-----------|-------------|
| `entries` | `collection`, `limit`, `page`, `sort`, `filter` | List entries with optional filtering |
| `entry` | `id` | Retrieve a single entry |
| `collections` | -- | List all collections |
| `collection` | `handle` | Get a single collection |
| `terms` | `taxonomy`, `limit`, `page`, `sort`, `filter` | List taxonomy terms |
| `term` | `id` | Retrieve a single term |
| `assets` | `container`, `limit`, `page` | List assets |
| `asset` | `id`, `container`, `path` | Retrieve a single asset |
| `globalSets` | -- | List all global sets |
| `globalSet` | `handle` | Get a single global set |
| `navs` | -- | List all navigations |
| `nav` | `handle` | Get a single navigation |
| `users` | `limit`, `page`, `sort`, `filter` | List users |
| `user` | `id`, `email` | Retrieve a single user |
| `forms` | -- | List all forms |
| `form` | `handle` | Get a single form |

### Blueprint-Specific Types

Each collection blueprint generates a specific GraphQL type. Access blueprint-specific fields using inline fragments:

```graphql
{
  entries(collection: "blog") {
    data {
      id
      title
      slug
      ... on Entry_Blog_Post {
        intro
        content
        featured_image {
          url
          alt
        }
      }
    }
  }
}
```

The type name follows the pattern `Entry_{Collection}_{Blueprint}` with PascalCase names.

### Single Entry Query

```graphql
{
  entry(id: "abc-123-def") {
    title
    slug
    ... on Entry_Blog_Post {
      intro
      content
    }
  }
}
```

### Taxonomy Terms Query

```graphql
{
  terms(taxonomy: "tags") {
    data {
      title
      slug
    }
  }
}
```

### Global Set Query

```graphql
{
  globalSet(handle: "site_settings") {
    ... on GlobalSet_SiteSettings {
      site_name
      tagline
    }
  }
}
```

### Navigation Query

```graphql
{
  nav(handle: "main") {
    tree {
      title
      url
      children {
        title
        url
      }
    }
  }
}
```

## Glide Parameter Reference

All parameters for Statamic's Glide image manipulation. Use in Antlers tags, URLs, or preset definitions.

### Dimensions and Sizing

| Parameter | Alias | Values | Description |
|-----------|-------|--------|-------------|
| `width` | `w` | Integer (pixels) | Target width |
| `height` | `h` | Integer (pixels) | Target height |
| `fit` | -- | `contain`, `max`, `fill`, `stretch`, `crop`, `crop_focal` | Resize fitting method |
| `dpr` | -- | Float (e.g., `2`) | Device pixel ratio multiplier |

**Fit modes explained:**
- `contain` -- Resize to fit within dimensions, maintaining aspect ratio
- `max` -- Like contain, but never upscale
- `fill` -- Resize to fill dimensions, cropping overflow
- `stretch` -- Stretch to exact dimensions, ignoring aspect ratio
- `crop` -- Crop to exact dimensions from center
- `crop_focal` -- Crop using the asset's focal point (set in CP)

### Quality and Format

| Parameter | Alias | Values | Description |
|-----------|-------|--------|-------------|
| `quality` | `q` | 0-100 | Image quality (JPEG/WebP/AVIF) |
| `format` | `fm` | `jpg`, `png`, `webp`, `avif` | Output format |

### Effects and Filters

| Parameter | Values | Description |
|-----------|--------|-------------|
| `blur` | 0-100 | Gaussian blur amount |
| `brightness` | -100 to 100 | Brightness adjustment |
| `contrast` | -100 to 100 | Contrast adjustment |
| `sharpen` | 0-100 | Sharpen amount |
| `pixelate` | Integer | Pixelation block size |
| `filter` | `greyscale`, `sepia` | Apply color filter |

### Orientation and Transform

| Parameter | Values | Description |
|-----------|--------|-------------|
| `orient` | `auto`, `0`, `90`, `180`, `270` | Rotation angle |
| `flip` | `v`, `h`, `both` | Flip vertically, horizontally, or both |
| `bg` | Hex color (e.g., `FFFFFF`) | Background color for transparent images |

### Watermark

| Parameter | Description |
|-----------|-------------|
| `mark` | Path to watermark image |
| `markw` | Watermark width (pixels or percentage) |
| `markh` | Watermark height (pixels or percentage) |
| `markx` | Horizontal offset from edge |
| `marky` | Vertical offset from edge |
| `markpad` | Padding around watermark |
| `markpos` | Position: `top-left`, `top`, `top-right`, `left`, `center`, `right`, `bottom-left`, `bottom`, `bottom-right` |
| `markfit` | Fit mode for watermark |
| `markalpha` | Watermark opacity (0-100) |

### Usage Examples

**In Antlers template:**
```antlers
{{ glide:featured_image width="800" height="600" fit="crop" quality="80" format="webp" }}
```

**As a URL:**
```
/img/asset/photos/hero.jpg?w=1440&h=600&fit=crop&q=90&fm=webp
```

**As a preset in config:**
```php
'presets' => [
    'thumbnail' => ['w' => 300, 'h' => 300, 'q' => 75, 'fit' => 'crop'],
    'hero' => ['w' => 1440, 'h' => 600, 'q' => 90, 'fm' => 'webp'],
    'avatar' => ['w' => 64, 'h' => 64, 'fit' => 'crop', 'q' => 80],
],
```
