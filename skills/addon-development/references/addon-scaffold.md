# Addon Scaffold Reference

Complete reference for Statamic addon package structure, configuration, and tooling.

## Complete Directory Layout

```
addons/acme/my-addon/
├── composer.json
├── package.json
├── vite.config.js
├── phpunit.xml
├── README.md
├── LICENSE
├── config/
│   └── my-addon.php              # Publishable config file
├── database/
│   └── migrations/               # Addon migrations
├── resources/
│   ├── dist/                     # Compiled Vite assets (gitignored in dev)
│   ├── js/
│   │   ├── addon.js              # JS entry point
│   │   └── components/
│   │       └── fieldtypes/
│   │           └── MyFieldtype.vue
│   ├── css/
│   │   └── addon.css             # CSS entry point
│   ├── lang/
│   │   └── en/
│   │       └── messages.php      # Translatable strings
│   └── views/
│       └── widget.blade.php      # Blade views
├── routes/
│   ├── cp.php                    # Control Panel routes
│   ├── actions.php               # Front-end form action routes
│   └── web.php                   # Public web routes
├── src/
│   ├── ServiceProvider.php       # Main addon service provider
│   ├── Tags/
│   │   └── MyTag.php
│   ├── Modifiers/
│   │   └── MyModifier.php
│   ├── Fieldtypes/
│   │   └── MyFieldtype.php
│   ├── Actions/
│   │   └── MyAction.php
│   ├── Scopes/
│   │   └── MyScope.php
│   ├── Widgets/
│   │   └── MyWidget.php
│   ├── Commands/
│   │   └── MyCommand.php
│   ├── Dictionaries/
│   │   └── MyDictionary.php
│   ├── Http/
│   │   └── Controllers/
│   │       └── MyController.php
│   └── Listeners/
│       └── MyListener.php
└── tests/
    ├── TestCase.php
    ├── Feature/
    └── Unit/
```

## composer.json

```json
{
    "name": "acme/my-addon",
    "description": "My Statamic addon",
    "type": "statamic-addon",
    "autoload": {
        "psr-4": {
            "Acme\\MyAddon\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Acme\\MyAddon\\Tests\\": "tests/"
        }
    },
    "require": {
        "statamic/cms": "^5.0"
    },
    "require-dev": {
        "orchestra/testbench": "^9.0",
        "phpunit/phpunit": "^11.0"
    },
    "extra": {
        "statamic": {
            "name": "My Addon",
            "description": "Description of My Addon"
        },
        "laravel": {
            "providers": [
                "Acme\\MyAddon\\ServiceProvider"
            ]
        }
    }
}
```

The `extra.laravel.providers` array enables Laravel's package auto-discovery, so the service provider is registered automatically without manual configuration.

## Service Provider with All Registration Arrays

```php
<?php

namespace Acme\MyAddon;

use Statamic\Providers\AddonServiceProvider;

class ServiceProvider extends AddonServiceProvider
{
    // --- Extension registration arrays ---

    protected $tags = [
        Tags\MyTag::class,
    ];

    protected $modifiers = [
        Modifiers\MyModifier::class,
    ];

    protected $fieldtypes = [
        Fieldtypes\MyFieldtype::class,
    ];

    protected $widgets = [
        Widgets\MyWidget::class,
    ];

    protected $commands = [
        Commands\MyCommand::class,
    ];

    protected $scopes = [
        Scopes\MyScope::class,
    ];

    protected $actions = [
        Actions\MyAction::class,
    ];

    // --- Route files ---

    protected $routes = [
        'cp' => __DIR__.'/../routes/cp.php',
        'actions' => __DIR__.'/../routes/actions.php',
        'web' => __DIR__.'/../routes/web.php',
    ];

    // --- Asset management ---

    protected $vite = [
        'input' => [
            'resources/js/addon.js',
            'resources/css/addon.css',
        ],
        'publicDirectory' => 'resources/dist',
    ];

    // Alternative: register pre-built scripts/stylesheets directly
    // protected $scripts = [__DIR__.'/../resources/dist/js/addon.js'];
    // protected $stylesheets = [__DIR__.'/../resources/dist/css/addon.css'];

    // External assets loaded from CDN or other URLs
    // protected $externalScripts = ['https://cdn.example.com/lib.js'];
    // protected $externalStylesheets = ['https://cdn.example.com/lib.css'];

    // --- Publishable assets ---

    protected $publishables = [
        __DIR__.'/../resources/dist' => 'vendor/my-addon',
    ];

    // --- View namespace ---

    protected $viewNamespace = 'my-addon';

    // --- Boot logic ---

    public function bootAddon()
    {
        // Runs AFTER Statamic has fully booted.
        // Use for event listeners, nav extensions, permissions, etc.

        $this->publishes([
            __DIR__.'/../config/my-addon.php' => config_path('my-addon.php'),
        ], 'my-addon-config');

        $this->mergeConfigFrom(
            __DIR__.'/../config/my-addon.php', 'my-addon'
        );

        $this->loadMigrationsFrom(__DIR__.'/../database/migrations');

        $this->loadTranslationsFrom(__DIR__.'/../resources/lang', 'my-addon');
    }

    public function register()
    {
        parent::register();

        // Bind services into the container if needed.
    }
}
```

## Route Files

### Control Panel Routes (routes/cp.php)

CP routes are automatically prefixed with the CP URL (default `/cp`) and protected by the CP authentication middleware.

```php
<?php

use Illuminate\Support\Facades\Route;
use Acme\MyAddon\Http\Controllers\MyController;

Route::name('my-addon.')->prefix('my-addon')->group(function () {
    Route::get('/', [MyController::class, 'index'])->name('index');
    Route::get('/{id}', [MyController::class, 'show'])->name('show');
    Route::post('/', [MyController::class, 'store'])->name('store');
    Route::put('/{id}', [MyController::class, 'update'])->name('update');
    Route::delete('/{id}', [MyController::class, 'destroy'])->name('destroy');
});
```

### Action Routes (routes/actions.php)

Action routes handle front-end form submissions. They are prefixed with `/!/` and do not require CP authentication.

```php
<?php

use Illuminate\Support\Facades\Route;
use Acme\MyAddon\Http\Controllers\ActionController;

Route::name('my-addon.')->prefix('my-addon')->group(function () {
    Route::post('/submit', [ActionController::class, 'submit'])->name('submit');
});
```

The resulting URL is `/!/my-addon/submit`.

### Web Routes (routes/web.php)

Standard Laravel web routes with no automatic prefix.

```php
<?php

use Illuminate\Support\Facades\Route;
use Acme\MyAddon\Http\Controllers\WebController;

Route::name('my-addon.')->prefix('my-addon')->group(function () {
    Route::get('/page', [WebController::class, 'page'])->name('page');
});
```

## Vite Configuration for Vue Components

### vite.config.js

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/js/addon.js',
                'resources/css/addon.css',
            ],
            publicDirectory: 'resources/dist',
            hotFile: 'resources/dist/hot',
        }),
        vue(),
    ],
});
```

### package.json

```json
{
    "private": true,
    "scripts": {
        "dev": "vite",
        "build": "vite build"
    },
    "devDependencies": {
        "@vitejs/plugin-vue": "^5.0",
        "laravel-vite-plugin": "^1.0",
        "vite": "^5.0",
        "vue": "^3.4"
    }
}
```

### JS Entry Point (resources/js/addon.js)

```js
import MyFieldtype from './components/fieldtypes/MyFieldtype.vue';
import MyWidget from './components/widgets/MyWidget.vue';

// Register fieldtype component: name must be {snake_case_handle}-fieldtype
Statamic.$components.register('my_fieldtype-fieldtype', MyFieldtype);

// Register widget component
Statamic.$components.register('my-widget', MyWidget);
```

### Building Assets

```bash
# Development with hot reload
cd addons/acme/my-addon && npm run dev

# Production build
cd addons/acme/my-addon && npm run build
```

The `$vite` array in the service provider tells Statamic to load the Vite assets in the CP. During development with `npm run dev`, hot module replacement is active. For production, run `npm run build` to compile assets into `resources/dist/`.

## Publishing Assets, Config, and Views

### From the Service Provider

```php
public function bootAddon()
{
    // Publish config
    $this->publishes([
        __DIR__.'/../config/my-addon.php' => config_path('my-addon.php'),
    ], 'my-addon-config');

    // Publish views
    $this->publishes([
        __DIR__.'/../resources/views' => resource_path('views/vendor/my-addon'),
    ], 'my-addon-views');

    // Publish assets (compiled JS/CSS)
    $this->publishes([
        __DIR__.'/../resources/dist' => public_path('vendor/my-addon'),
    ], 'my-addon-assets');

    // Merge default config (so it works without publishing)
    $this->mergeConfigFrom(
        __DIR__.'/../config/my-addon.php', 'my-addon'
    );

    // Load views with namespace
    $this->loadViewsFrom(__DIR__.'/../resources/views', 'my-addon');
    // Usage in Blade: @include('my-addon::widget')
    // Usage in Antlers: {{ view:my-addon::widget }}
}
```

### Publishing from the Command Line

```bash
# Publish everything for the addon
php artisan vendor:publish --provider="Acme\MyAddon\ServiceProvider"

# Publish only config
php artisan vendor:publish --tag=my-addon-config

# Publish only views
php artisan vendor:publish --tag=my-addon-views
```

## Testing Setup

### phpunit.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
    </testsuites>
    <source>
        <include>
            <directory>src</directory>
        </include>
    </source>
</phpunit>
```

### Base TestCase (tests/TestCase.php)

```php
<?php

namespace Acme\MyAddon\Tests;

use Acme\MyAddon\ServiceProvider;
use Statamic\Testing\AddonTestCase;

abstract class TestCase extends AddonTestCase
{
    protected string $addonServiceProvider = ServiceProvider::class;

    // Optionally override to configure the test environment
    protected function resolveApplicationConfiguration($app)
    {
        parent::resolveApplicationConfiguration($app);

        // Custom config for tests
        // $app['config']->set('my-addon.key', 'value');
    }
}
```

### Example Test

```php
<?php

namespace Acme\MyAddon\Tests\Feature;

use Acme\MyAddon\Tests\TestCase;

class MyTagTest extends TestCase
{
    public function test_tag_renders_greeting()
    {
        $this->assertEquals(
            'Hello',
            (string) $this->tag('{{ my_tag }}')
        );
    }

    public function test_tag_accepts_custom_greeting()
    {
        $this->assertEquals(
            'Howdy',
            (string) $this->tag('{{ my_tag greeting="Howdy" }}')
        );
    }
}
```

### Running Tests

```bash
cd addons/acme/my-addon
composer install
./vendor/bin/phpunit
```
