# Tag Conditions and Filtering Reference

Use conditions on Statamic tags (primarily `collection`, but also `taxonomy`, `assets`, and others) to filter results directly in the template.

## Condition Syntax

Apply conditions as parameters on tag pairs using the pattern:

```
{field}:{condition}="{value}"
```

Multiple conditions are combined with AND logic:

```antlers
{{ collection:blog author:is="john" status:is="published" date:is_after="2025-01-01" }}
  {{ title }}
{{ /collection:blog }}
```

## Condition Operators

### String Conditions

| Condition | Description | Example |
|---|---|---|
| `is` | Equals the given value | `author:is="john"` |
| `not` / `isnt` | Does not equal the given value | `status:not="draft"` |
| `exists` | Field has a value (is not null) | `hero_image:exists="true"` |
| `doesnt_exist` / `null` | Field is null or does not exist | `subtitle:doesnt_exist="true"` |
| `contains` | Field value contains the substring | `title:contains="guide"` |
| `doesnt_contain` | Field value does not contain the substring | `title:doesnt_contain="draft"` |
| `in` | Field value is one of the given pipe-separated values | `status:in="published|scheduled"` |
| `not_in` | Field value is not one of the given values | `category:not_in="archived|hidden"` |
| `starts_with` | Field value starts with the given string | `slug:starts_with="how-to"` |
| `ends_with` | Field value ends with the given string | `email:ends_with="@example.com"` |
| `gt` | Greater than (numeric/date comparison) | `price:gt="100"` |
| `gte` | Greater than or equal to | `price:gte="50"` |
| `lt` | Less than | `stock:lt="10"` |
| `lte` | Less than or equal to | `rating:lte="3"` |
| `matches` / `regex` | Matches a regular expression | `sku:matches="/^[A-Z]{3}/"` |
| `is_alpha` | Value contains only alphabetic characters | `code:is_alpha="true"` |
| `is_numeric` | Value is numeric | `quantity:is_numeric="true"` |
| `is_url` | Value is a valid URL | `website:is_url="true"` |
| `is_email` | Value is a valid email address | `contact:is_email="true"` |
| `is_after` | Date is after the given date | `date:is_after="2025-06-01"` |
| `is_before` | Date is before the given date | `date:is_before="2025-12-31"` |

### Array Conditions

| Condition | Description | Example |
|---|---|---|
| `overlaps` | Array field shares at least one value with the given pipe-separated list | `tags:overlaps="php|laravel"` |
| `doesnt_overlap` | Array field shares no values with the given list | `tags:doesnt_overlap="archived|deprecated"` |

## Taxonomy Filtering

Filter collection entries by taxonomy terms using the `taxonomy:` prefix. Separate multiple terms with the pipe `|` character.

### Any Match (default)

Return entries tagged with any of the listed terms (OR logic):

```antlers
{{ collection:blog taxonomy:tags="review|colorful" }}
  {{ title }}
{{ /collection:blog }}
```

### All Match

Return only entries tagged with all of the listed terms (AND logic):

```antlers
{{ collection:blog taxonomy:tags:all="review|colorful" }}
  {{ title }}
{{ /collection:blog }}
```

### Exclude

Return entries that are not tagged with any of the listed terms:

```antlers
{{ collection:blog taxonomy:tags:not="boring" }}
  {{ title }}
{{ /collection:blog }}
```

### Combining Taxonomy Filters

Combine multiple taxonomy conditions to narrow results across different taxonomies:

```antlers
{{ collection:blog taxonomy:tags="featured" taxonomy:categories:all="tutorials|php" taxonomy:authors:not="guest" }}
  {{ title }}
{{ /collection:blog }}
```

## Common Tag Parameters

These parameters work alongside conditions on collection and similar tags:

| Parameter | Description | Example |
|---|---|---|
| `limit` | Maximum number of results | `limit="10"` |
| `sort` | Sort field and direction | `sort="date:desc"` |
| `offset` | Skip the first N results | `offset="3"` |
| `paginate` | Enable pagination with N per page | `paginate="10"` |
| `as` | Alias the results variable | `as="posts"` |
| `scope` | Scope outer variables to prevent conflicts | `scope="entry"` |

## Full Example

```antlers
{{ collection:blog
   limit="12"
   sort="date:desc"
   taxonomy:tags="featured"
   status:is="published"
   date:is_before="{ now }"
   author:isnt="admin"
   paginate="6"
   as="posts"
}}
  {{ if no_results }}
    <p>No posts found.</p>
  {{ /if }}

  {{ posts }}
    <article>
      <h2><a href="{{ url }}">{{ title }}</a></h2>
      <time>{{ date format="M d, Y" }}</time>
      <p>{{ content | truncate(150, '...') }}</p>
    </article>
  {{ /posts }}

  {{ paginate }}
    {{ if prev_page }}<a href="{{ prev_page }}">Previous</a>{{ /if }}
    {{ if next_page }}<a href="{{ next_page }}">Next</a>{{ /if }}
  {{ /paginate }}
{{ /collection:blog }}
```
