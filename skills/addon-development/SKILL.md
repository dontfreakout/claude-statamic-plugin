---
name: addon-development
description: >-
  This skill should be used when the user asks to "create addon", "custom tag",
  "custom modifier", "custom fieldtype", "service provider", "addon scaffold",
  "make:tag", "make:modifier", "make:fieldtype", "make:action", "make:scope",
  "make:widget", "make:dictionary", or works with Statamic's AddonServiceProvider.
version: 0.1.0
---

# Statamic Addon Development

## Creating an Addon

Scaffold a new addon with `php please make:addon`:

```bash
php please make:addon acme/my-addon
```

This creates the addon scaffold at `addons/acme/my-addon/` with a service provider, `composer.json`, and standard directories. The addon is automatically symlinked into the application via a Composer path repository entry in the root `composer.json`.

The vendor/package name (e.g., `acme/my-addon`) determines both the Composer package name and the PSR-4 namespace (`Acme\MyAddon`). Choose a vendor name that matches the organization or developer publishing the addon.

After scaffolding, register any extension classes in the service provider or rely on auto-registration from standard directories (see Extension Points below). Install addon dependencies by running `composer install` inside the addon directory.

See `references/addon-scaffold.md` for the complete addon package structure, directory layout, route files, Vite configuration, asset publishing, and testing setup.

## Service Provider Structure

Every addon has a service provider extending `Statamic\Providers\AddonServiceProvider`. Use the protected property arrays to register extension classes and configure routes and assets.

```php
namespace Acme\MyAddon;

use Statamic\Providers\AddonServiceProvider;

class ServiceProvider extends AddonServiceProvider
{
    protected $tags = [Tags\MyTag::class];
    protected $modifiers = [Modifiers\MyModifier::class];
    protected $fieldtypes = [Fieldtypes\MyFieldtype::class];
    protected $widgets = [Widgets\MyWidget::class];
    protected $commands = [Commands\MyCommand::class];
    protected $routes = [
        'cp' => __DIR__.'/../routes/cp.php',
        'actions' => __DIR__.'/../routes/actions.php',
        'web' => __DIR__.'/../routes/web.php',
    ];
    protected $vite = [
        'input' => ['resources/js/addon.js', 'resources/css/addon.css'],
        'publicDirectory' => 'resources/dist',
    ];

    public function bootAddon()
    {
        // Boot logic (runs AFTER Statamic boots)
    }
}
```

### Registration Arrays

The following protected arrays are available on `AddonServiceProvider`:

- `$tags` -- Tag classes to register
- `$modifiers` -- Modifier classes to register
- `$fieldtypes` -- Fieldtype classes to register
- `$widgets` -- Dashboard widget classes to register
- `$commands` -- Artisan command classes to register
- `$scopes` -- Query scope classes to register
- `$actions` -- Action classes to register
- `$routes` -- Route file paths keyed by type (`cp`, `actions`, `web`)
- `$vite` -- Vite build configuration with `input` array and `publicDirectory`
- `$scripts` -- JavaScript files to load in the CP
- `$stylesheets` -- CSS files to load in the CP
- `$externalScripts` -- External JS URLs to load in the CP
- `$externalStylesheets` -- External CSS URLs to load in the CP
- `$publishables` -- Files/directories to publish via `vendor:publish`
- `$viewNamespace` -- Namespace for addon views

### Routes Configuration

The `$routes` array maps route types to file paths:

- `cp` -- Routes prefixed with the CP URL (default `/cp`) and protected by CP authentication middleware. Use for admin interfaces and settings pages.
- `actions` -- Routes prefixed with `/!/` for front-end form submissions. Not protected by CP auth, making them suitable for public-facing form handlers.
- `web` -- Standard Laravel web routes with no automatic prefix. Use for public pages served by the addon.

### Vite Configuration

The `$vite` array configures asset compilation for the Control Panel:

- `input` -- Array of JS and CSS entry points to compile.
- `publicDirectory` -- Directory where compiled assets are placed (typically `resources/dist`).

During development, run `npm run dev` inside the addon directory for hot module replacement. For production, run `npm run build` to compile assets. The service provider automatically handles loading the correct assets based on whether Vite's dev server is running.

### The bootAddon Method

Place addon boot logic in `bootAddon()` instead of `boot()`. This method runs after Statamic has fully booted, ensuring all Statamic services are available. Use it for:

- Registering event listeners
- Extending navigation or CP menus
- Publishing config, views, and assets via `$this->publishes()`
- Merging default configuration via `$this->mergeConfigFrom()`
- Loading migrations via `$this->loadMigrationsFrom()`
- Loading translations via `$this->loadTranslationsFrom()`
- Registering permissions

## Extension Points and Auto-Registration

When building extensions directly in a Statamic application (not in an addon package), place classes in the standard directories for automatic registration -- no service provider entry needed:

- `app/Tags/` -- Custom tags
- `app/Modifiers/` -- Custom modifiers
- `app/Fieldtypes/` -- Custom fieldtypes
- `app/Actions/` -- Custom actions
- `app/Scopes/` -- Query scopes
- `app/Dictionaries/` -- Custom dictionaries

For addon packages, register classes explicitly in the service provider's property arrays. Auto-registration only applies to the main application's `app/` directory, not to addon `src/` directories.

## Building Custom Tags

Generate a tag with `php please make:tag MyTag`. Tags extend `Statamic\Tags\Tags`.

```php
namespace App\Tags;

use Statamic\Tags\Tags;

class MyTag extends Tags
{
    public function index()  // {{ my_tag }}
    {
        return $this->params->get('greeting', 'Hello');
    }

    public function world()  // {{ my_tag:world }}
    {
        return ['items' => [['name' => 'Earth'], ['name' => 'Mars']]];
    }

    public function wildcard($method)  // {{ my_tag:* }}
    {
        return "Called: {$method}";
    }
}
```

### Tag Method Resolution

- **Index method** -- `index()` handles `{{ my_tag }}` (the tag with no method suffix).
- **Named methods** -- A public method like `world()` handles `{{ my_tag:world }}`.
- **Wildcard method** -- `wildcard($method)` catches any method not explicitly defined, receiving the method name as `$method`.

### Returning Data from Tags

- Return a **string** for single-value output: `{{ my_tag }}` renders the string directly.
- Return an **array** to make variables available between tag pairs: `{{ my_tag:world }}{{ items }}{{ name }}{{ /items }}{{ /my_tag:world }}`.
- Return a **collection or query builder** to let Statamic handle pagination and iteration.
- Access tag parameters via `$this->params->get('key', 'default')`.
- Access the current context via `$this->context`.
- Use `$this->isPair` to check whether the tag is used as a pair (has closing tag) or a single tag.

## Building Custom Modifiers

Generate a modifier with `php please make:modifier Repeat`. Modifiers extend `Statamic\Modifiers\Modifier`.

```php
namespace App\Modifiers;

use Statamic\Modifiers\Modifier;

class Repeat extends Modifier
{
    public function index($value, $params, $context)
    {
        return str_repeat($value, $params[0] ?? 2);
    }
}
// Usage: {{ title | repeat:3 }}
```

### Modifier Parameters

- `$value` -- The current value being modified.
- `$params` -- Array of parameters passed after the colon (e.g., `repeat:3` yields `$params[0] = 3`). Multiple parameters are separated by colons: `{{ title | truncate:50:... }}` yields `$params = [50, '...']`.
- `$context` -- The full template context (all available variables).

### Modifier Naming

The modifier handle is derived from the class name in snake_case. A class named `Repeat` becomes the modifier `repeat`. A class named `SentenceCase` becomes `sentence_case`. Use the modifier in Antlers with the pipe syntax: `{{ variable | modifier_name:param }}`. Modifiers can be chained: `{{ title | upper | truncate:50 }}`.

## Building Custom Fieldtypes

Generate a fieldtype with `php please make:fieldtype TogglePassword`. Fieldtypes have two parts: a PHP class and a Vue component.

### PHP Class

```php
namespace App\Fieldtypes;

use Statamic\Fields\Fieldtype;

class TogglePassword extends Fieldtype
{
    protected $icon = 'lock';
    public $categories = ['text'];

    protected function configFieldItems(): array
    {
        return [
            'mode' => [
                'display' => 'Mode',
                'type' => 'select',
                'options' => ['plain' => 'Plain', 'masked' => 'Masked'],
            ],
        ];
    }

    public function preProcess($value) { return $value; }
    public function process($value) { return $value; }
    public function augment($value) { return $value; }
}
```

### Key Fieldtype Methods

- `configFieldItems()` -- Define configuration fields shown in the blueprint editor.
- `preProcess($value)` -- Transform the stored value before sending to the Vue component.
- `process($value)` -- Transform the Vue component value before storing.
- `augment($value)` -- Transform the stored value for frontend template output.

### Vue Component (Statamic v6 -- Composition API)

```vue
<script setup>
import { Fieldtype } from '@statamic/cms';

const emit = defineEmits(Fieldtype.emits);
const props = defineProps(Fieldtype.props);
const { expose, update, meta } = Fieldtype.use(emit, props);
defineExpose(expose);
</script>

<template>
    <input :value="value" @input="update($event.target.value)">
</template>
```

Register the Vue component in the addon's JS entry point:

```js
import Component from './components/fieldtypes/TogglePassword.vue';
Statamic.$components.register('toggle_password-fieldtype', Component);
```

The component name must follow the pattern `{handle}-fieldtype` where the handle is the snake_case version of the class name. For example, `TogglePassword` becomes `toggle_password-fieldtype`.

### Fieldtype Vue Component Helpers

The `Fieldtype.use(emit, props)` composable provides:

- `update(value)` -- Emit the updated value back to the parent form. Call this whenever the field value changes.
- `meta` -- Reactive object containing server-side meta data passed from the PHP fieldtype's `preProcessMeta()` method. Use to pass options, configuration, or computed data from PHP to the Vue component.
- `expose` -- Object to pass to `defineExpose()` for parent component access to fieldtype methods.

## Other Make Commands

Statamic provides artisan commands to scaffold various extension types:

| Command | Creates | Directory |
|---|---|---|
| `php please make:tag` | Custom tag | `app/Tags/` |
| `php please make:modifier` | Custom modifier | `app/Modifiers/` |
| `php please make:fieldtype` | Custom fieldtype | `app/Fieldtypes/` |
| `php please make:action` | Custom action | `app/Actions/` |
| `php please make:filter` | Custom query filter | `app/Filters/` |
| `php please make:scope` | Custom query scope | `app/Scopes/` |
| `php please make:widget` | Dashboard widget | `app/Widgets/` |
| `php please make:dictionary` | Custom dictionary | `app/Dictionaries/` |

When creating extensions inside an addon, use the `--addon` flag:

```bash
php please make:tag MyTag --addon=acme/my-addon
```

This places the generated class in the addon's `src/` directory with the correct namespace and registers it in the addon's service provider.

## Addon Development Workflow

### Local Development

1. Scaffold the addon: `php please make:addon vendor/addon-name`
2. Generate extensions with `--addon` flag as needed
3. Register extensions in the service provider's property arrays
4. If the addon has Vue components, run `npm install && npm run dev` in the addon directory for hot reload
5. Run `npm run build` before committing to compile production assets

### Distributing an Addon

Prepare an addon for distribution by ensuring the `composer.json` has correct metadata in `extra.statamic` (name, description) and `extra.laravel.providers` for auto-discovery. Tag a release and publish to Packagist. Users install with `composer require vendor/addon-name`.

### Testing

Use `Statamic\Testing\AddonTestCase` as the base test class. It boots the addon's service provider in an isolated Statamic test environment. Run tests with PHPUnit from the addon directory. See `references/addon-scaffold.md` for the full testing setup including `phpunit.xml` and example tests.
