---
name: create-addon
description: This skill should be used when the user asks to "create a custom tag", "create a modifier", "create a fieldtype", "scaffold addon", "make:tag", "make:modifier", "make:fieldtype", "make:action", "make:scope", or wants to create any Statamic extension point (tag, modifier, fieldtype, action, filter, scope, widget, dictionary).
argument-hint: "custom tag called EventList that queries upcoming events"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
version: 0.1.0
---

# Create Statamic Addon Component

Scaffold Statamic extension points — custom tags, modifiers, fieldtypes, actions, scopes, and more — with correct boilerplate and registration.

## Workflow

### 1. Determine Extension Type

Identify what the user wants to create:

| Type | Directory | Base Class | Template Tag |
|------|-----------|------------|--------------|
| Tag | `app/Tags/` | `Statamic\Tags\Tags` | `{{ my_tag }}` |
| Modifier | `app/Modifiers/` | `Statamic\Modifiers\Modifier` | `{{ value \| my_mod }}` |
| Fieldtype | `app/Fieldtypes/` | `Statamic\Fields\Fieldtype` | CP field |
| Action | `app/Actions/` | `Statamic\Actions\Action` | CP bulk action |
| Scope | `app/Scopes/` | `Statamic\Query\Scopes\Scope` | `query_scope="name"` |
| Filter | `app/Scopes/Filters/` | `Statamic\Query\Scopes\Filter` | CP filter |
| Widget | `app/Widgets/` | `Statamic\Widgets\Widget` | CP dashboard |
| Dictionary | `app/Dictionaries/` | `Statamic\Dictionaries\BasicDictionary` | Dictionary fieldtype |

All directories under `app/` are auto-registered by Statamic — no manual service provider registration needed for project-level extensions.

### 2. Generate the PHP Class

#### Custom Tag

```php
<?php

namespace App\Tags;

use Statamic\Tags\Tags;

class {ClassName} extends Tags
{
    /**
     * {{ {tag_name} }}
     */
    public function index()
    {
        // Return a string for single values
        // Return an array for tag pairs (loops)
        return [];
    }

    /**
     * {{ {tag_name}:{method} }}
     */
    public function {methodName}()
    {
        return [];
    }

    /**
     * {{ {tag_name}:* }}
     */
    public function wildcard($method)
    {
        return "Called: {$method}";
    }
}
```

Key tag patterns:
- `$this->params->get('key', 'default')` — access tag parameters
- `$this->context->get('key')` — access template context/cascade
- `$this->isPair` — check if used as tag pair
- `$this->content` — get content between tag pair
- Return string for output, array for loopable data

#### Custom Modifier

```php
<?php

namespace App\Modifiers;

use Statamic\Modifiers\Modifier;

class {ClassName} extends Modifier
{
    /**
     * {{ value | {modifier_name}:param1:param2 }}
     */
    public function index($value, $params, $context)
    {
        return $value;
    }
}
```

Parameters:
- `$value` — the value being modified
- `$params` — array of colon-separated parameters
- `$context` — full template context array

#### Custom Fieldtype

```php
<?php

namespace App\Fieldtypes;

use Statamic\Fields\Fieldtype;

class {ClassName} extends Fieldtype
{
    protected $icon = '{icon}';
    public $categories = ['{category}'];

    protected function configFieldItems(): array
    {
        return [
            'option_name' => [
                'display' => 'Option Label',
                'instructions' => 'Help text for this option.',
                'type' => 'select',
                'options' => [
                    'value1' => 'Label 1',
                    'value2' => 'Label 2',
                ],
                'default' => 'value1',
            ],
        ];
    }

    public function preProcess($value)
    {
        // Transform value before sending to Vue component
        return $value;
    }

    public function process($value)
    {
        // Transform value before saving to file
        return $value;
    }

    public function augment($value)
    {
        // Transform value for template rendering
        return $value;
    }
}
```

#### Custom Action

```php
<?php

namespace App\Actions;

use Statamic\Actions\Action;

class {ClassName} extends Action
{
    public static function title()
    {
        return '{Title}';
    }

    public function visibleTo($item)
    {
        return $item instanceof \Statamic\Entries\Entry;
    }

    public function authorize($user, $item)
    {
        return $user->can('edit', $item);
    }

    public function run($items, $values)
    {
        foreach ($items as $item) {
            // Perform action on each item
        }

        return trans_choice('{count} item processed|{count} items processed', $items->count());
    }
}
```

#### Custom Query Scope

```php
<?php

namespace App\Scopes;

use Statamic\Query\Scopes\Scope;

class {ClassName} extends Scope
{
    public function apply($query, $values)
    {
        $query->where('{field}', '{value}');
    }
}
```

Usage in templates: `{{ collection:blog query_scope="{scope_name}" }}`

#### Custom Widget

```php
<?php

namespace App\Widgets;

use Statamic\Widgets\Widget;

class {ClassName} extends Widget
{
    public function html()
    {
        return view('{view_name}', [
            'data' => $this->config('data_source', 'default'),
        ]);
    }
}
```

### 3. Naming Conventions

- **Class name**: StudlyCase (e.g., `EventList`, `ReadTime`)
- **Tag handle**: snake_case of class name (e.g., `{{ event_list }}`)
- **Modifier handle**: snake_case of class name (e.g., `{{ value | read_time }}`)
- **Fieldtype handle**: snake_case of class name (e.g., `toggle_password`)
- **Scope handle**: snake_case for template use (e.g., `query_scope="featured"`)

### 4. Create the File

Write the PHP class to the correct auto-registered directory. Verify the directory exists first:

```bash
mkdir -p app/{Type}s
```

### 5. Vue Component (Fieldtypes Only)

For custom fieldtypes, create a Vue 3 component using Composition API:

```vue
<script setup>
import { Fieldtype } from '@statamic/cms';

const emit = defineEmits(Fieldtype.emits);
const props = defineProps(Fieldtype.props);
const { expose, update, meta } = Fieldtype.use(emit, props);

defineExpose(expose);
</script>

<template>
  <div>
    <input :value="props.value" @input="update($event.target.value)" />
  </div>
</template>
```

Register in addon JS: `Statamic.$components.register('{handle}-fieldtype', Component);`

### 6. Summary

After creating the extension, display:
- File path of the created class
- How to use it (template tag syntax, modifier syntax, etc.)
- Reminder that `app/` directory extensions are auto-registered
- For addon packages: mention `php please make:addon` for full scaffold

## Full Addon Package

For a distributable addon package (not just a project-level extension), use:

```bash
php please make:addon vendor/package-name
```

This creates a full scaffold in `addons/vendor/package-name/` with:
- ServiceProvider extending `AddonServiceProvider`
- composer.json with auto-discovery
- Standard directory structure for all extension types

Register extensions in the service provider's protected arrays: `$tags`, `$modifiers`, `$fieldtypes`, `$widgets`, `$commands`, `$routes`, `$vite`.
