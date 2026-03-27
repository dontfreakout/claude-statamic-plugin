# Antlers Modifier Reference

Complete list of approximately 150 modifiers available in Statamic Antlers templates. Apply with the pipe syntax: `{{ value | modifier }}` or `{{ value | modifier(arg1, arg2) }}`.

## String Modifiers

| Modifier | Description |
|---|---|
| `upper` | Convert to uppercase |
| `lower` | Convert to lowercase |
| `title` | Convert to title case |
| `ucfirst` | Capitalize first character |
| `lcfirst` | Lowercase first character |
| `slugify` | Convert to URL-safe slug |
| `deslugify` | Convert slug back to readable text |
| `camelize` | Convert to camelCase |
| `studly` | Convert to StudlyCase |
| `snake` | Convert to snake_case |
| `kebab` | Convert to kebab-case |
| `dashify` | Convert to dash-separated string |
| `trim` | Remove whitespace from both ends |
| `ltrim` | Remove whitespace from left |
| `rtrim` | Remove whitespace from right |
| `replace` | Replace occurrences of a substring |
| `regex_replace` | Replace using a regular expression |
| `remove_left` | Remove a prefix string |
| `remove_right` | Remove a suffix string |
| `ensure_left` | Ensure string starts with a given prefix |
| `ensure_right` | Ensure string ends with a given suffix |
| `insert` | Insert a string at a given position |
| `substr` | Extract a substring by position and length |
| `truncate` | Truncate to a character limit with suffix |
| `safe_truncate` | Truncate at word boundary with suffix |
| `backspace` | Remove N characters from the end |
| `surround` | Wrap string with a given string on both sides |
| `wrap` | Wrap with different left/right strings |
| `widont` | Prevent widows by adding non-breaking space before last word |
| `entities` | Encode HTML entities |
| `sanitize` | Strip tags and encode entities |
| `ascii` | Transliterate to ASCII characters |
| `markdown` | Parse Markdown to HTML |
| `smartypants` | Convert quotes and dashes to typographic equivalents |
| `strip_tags` | Remove HTML tags |
| `nl2br` | Convert newlines to `<br>` tags |
| `collapse_whitespace` | Collapse consecutive whitespace to single space |
| `spaceless` | Remove whitespace between HTML tags |
| `tidy` | Clean and fix malformed HTML |
| `embed_url` | Convert video URL to embeddable URL |
| `gravatar` | Convert email to Gravatar image URL |
| `md5` | Generate MD5 hash |
| `urlencode` | URL-encode the string |
| `urldecode` | URL-decode the string |
| `rawurlencode` | Raw URL-encode (RFC 3986) |
| `parse_url` | Parse URL into components |
| `pathinfo` | Parse file path into components |
| `full_urls` | Convert relative URLs to absolute in HTML |
| `read_time` | Estimate reading time for content |
| `word_count` | Count the number of words |
| `excerpt` | Extract an excerpt from content |
| `headline` | Convert to human-readable headline |
| `mark` | Wrap a substring with `<mark>` tags |
| `regex_mark` | Wrap regex matches with `<mark>` tags |
| `antlers` | Parse string as Antlers template |

## Array Modifiers

| Modifier | Description |
|---|---|
| `count` / `length` | Return the number of items |
| `first` | Return the first item |
| `last` | Return the last item |
| `join` | Join items into a string with a delimiter |
| `limit` | Take the first N items |
| `offset` | Skip the first N items |
| `reverse` | Reverse the order of items |
| `shuffle` | Randomize the order of items |
| `sort` | Sort items by a key or value |
| `unique` | Remove duplicate items |
| `flatten` | Flatten a multi-dimensional array |
| `collapse` | Collapse an array of arrays into a single array |
| `compact` | Remove null/empty values |
| `filter_empty` | Remove empty values |
| `pluck` | Extract values for a given key |
| `select` | Select specific keys from each item |
| `where` | Filter items matching a condition |
| `where-in` | Filter items where key is in a list of values |
| `group_by` | Group items by a key |
| `flip` | Swap keys and values |
| `keys` | Return all keys |
| `values` | Return all values (re-indexed) |
| `pad` | Pad array to a given length with a value |
| `random` | Return a random item |
| `sum` | Sum numeric values (optionally by key) |
| `merge` | Merge with another array |

## Date Modifiers

| Modifier | Description |
|---|---|
| `format` | Format date using PHP date format string |
| `format_translated` | Format date with translated month/day names |
| `iso_format` | Format date using ISO format tokens |
| `relative` | Display as relative time (e.g., "2 hours ago") |
| `modify_date` | Modify date by adding/subtracting intervals |
| `timezone` | Convert to a specific timezone |
| `days_ago` | Number of days since the date |
| `hours_ago` | Number of hours since the date |
| `minutes_ago` | Number of minutes since the date |
| `weeks_ago` | Number of weeks since the date |
| `months_ago` | Number of months since the date |
| `years_ago` | Number of years since the date |
| `is_future` | Return true if date is in the future |
| `is_past` | Return true if date is in the past |
| `is_today` | Return true if date is today |
| `is_tomorrow` | Return true if date is tomorrow |
| `is_yesterday` | Return true if date is yesterday |
| `is_weekday` | Return true if date is a weekday |
| `is_weekend` | Return true if date is on a weekend |
| `is_leap_year` | Return true if date is in a leap year |
| `is_before` | Return true if date is before a given date |
| `is_after` | Return true if date is after a given date |
| `is_between` | Return true if date is between two dates |

## Math Modifiers

| Modifier | Description |
|---|---|
| `add` | Add a number |
| `subtract` | Subtract a number |
| `multiply` | Multiply by a number |
| `divide` | Divide by a number |
| `mod` | Return the remainder (modulo) |
| `round` | Round to N decimal places |
| `ceil` | Round up to the nearest integer |
| `floor` | Round down to the nearest integer |
| `format_number` | Format with decimal places and separators |

## Markup Modifiers

| Modifier | Description |
|---|---|
| `ul` | Render array as an unordered list (`<ul>`) |
| `ol` | Render array as an ordered list (`<ol>`) |
| `dl` | Render array as a definition list (`<dl>`) |
| `list` | Render array as a generic list |
| `ampersand_list` | Join items with commas and "&" before last |
| `sentence_list` | Join items with commas and "and" before last |
| `piped` | Join items with pipe characters |
| `option_list` | Render as `<option>` elements for `<select>` |
| `link` | Wrap value in an `<a>` tag |
| `mailto` | Wrap email in a `mailto:` link |
| `image` | Wrap value in an `<img>` tag |
| `table` | Render array of arrays as an HTML table |
| `obfuscate` | Obfuscate text using HTML entities |
| `obfuscate_email` | Obfuscate email with JavaScript protection |
| `favicon` | Generate favicon URL from a domain |
| `chunk` | Split array into smaller arrays of N size |

## Utility Modifiers

| Modifier | Description |
|---|---|
| `to_json` | Convert value to JSON string |
| `to_qs` | Convert array to query string |
| `bool_string` | Convert boolean to "true"/"false" string |
| `raw` | Output without escaping |
| `dump` | Dump value for debugging |
| `console_log` | Output as JavaScript `console.log()` |
| `ray` | Send value to Ray debugging tool |
| `repeat` | Repeat the string N times |
| `decode` | Decode HTML entities |
| `partial` | Render value through a partial template |
| `macro` | Apply a named macro (reusable modifier chain) |
| `as` | Alias the value as a named variable |
| `get` | Get a nested value by key |
| `output` | Force output of a value |
| `to_spaces` | Convert tabs to spaces |
| `to_tabs` | Convert spaces to tabs |
| `hex_to_rgb` | Convert hex color to RGB values |

## Condition Modifiers

Use in conditionals to test values. Return boolean results.

| Modifier | Description |
|---|---|
| `contains` | True if string/array contains a value |
| `contains_all` | True if value contains all given items |
| `contains_any` | True if value contains any of the given items |
| `starts_with` | True if string starts with a given prefix |
| `ends_with` | True if string ends with a given suffix |
| `in_array` | True if value exists in a given array |
| `is_array` | True if value is an array |
| `is_empty` | True if value is empty |
| `is_blank` | True if value is blank (empty, null, whitespace) |
| `is_json` | True if value is valid JSON |
| `is_url` | True if value is a valid URL |
| `is_email` | True if value is a valid email address |
| `is_numeric` | True if value is numeric |
| `is_alpha` | True if value contains only letters |
| `is_alphanumeric` | True if value contains only letters and numbers |
| `is_uppercase` | True if value is entirely uppercase |
| `is_lowercase` | True if value is entirely lowercase |
| `has_upper_case` | True if value contains at least one uppercase letter |
| `has_lower_case` | True if value contains at least one lowercase letter |
| `is_external_url` | True if URL points to an external domain |
| `is_embeddable` | True if URL is embeddable (YouTube, Vimeo, etc.) |
