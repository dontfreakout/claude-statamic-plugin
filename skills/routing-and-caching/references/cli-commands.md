# Statamic CLI Command Reference

Complete reference for all `php please` and related Statamic CLI commands.

## Project Management

```bash
statamic new project-name              # Create new Statamic project
php please install                     # Install Statamic into existing Laravel app
php please multisite                   # Convert single-site to multisite setup
php please site:clear                  # Wipe all content from the site
php please pro:enable                  # Enable Statamic Pro in .env
php please license:set                 # Set the license key
php please support:details             # List environment and support details
```

## Content Creation

```bash
php please make:user                   # Create a new user account (interactive)
```

## Cache Management

```bash
php please stache:clear                # Clear the Stache content cache
php please stache:warm                 # Build/rebuild the Stache content cache
php please stache:refresh              # Clear + rebuild Stache in one step
php please stache:doctor               # Diagnose Stache issues
php please glide:clear                 # Clear the Glide image manipulation cache
php please static:clear                # Clear all static page cache files
php please static:warm                 # Crawl and warm the static cache for all URLs
php please static:warm --queue         # Warm static cache via background queue
php please assets:clear-cache          # Clear asset container caches
php please assets:meta                 # Regenerate asset metadata files
php please assets:generate-presets     # Generate all configured image presets
php artisan cache:clear                # Clear the Laravel application cache
```

## Search

```bash
php please search:update               # Update search index (interactive picker)
php please search:update default       # Update a specific named search index
php please search:update --all         # Update all configured search indexes
php please search:insert               # Insert an item into search indexes
```

## Code Generation (make: commands)

```bash
php please make:addon vendor/package   # Create addon scaffold with service provider
php please make:tag MyTag              # Create a custom Antlers tag class
php please make:modifier MyMod         # Create a custom modifier class
php please make:fieldtype MyField      # Create a custom fieldtype class
php please make:action MyAction        # Create a custom action class
php please make:filter MyFilter        # Create a custom filter class
php please make:scope MyScope          # Create a custom query scope class
php please make:widget MyWidget        # Create a custom CP widget class
php please make:dictionary MyDict      # Create a custom dictionary class
php please make:user-migration         # Create the user database migration file
```

## Git and Deployment

```bash
php please git:commit                  # Trigger a manual git commit (Git Automation)
php please install:ssg                 # Install the Static Site Generator package
php please install:eloquent-driver     # Install the Eloquent database driver
php please install:collaboration       # Install the real-time collaboration addon
```

## Imports and Migrations

```bash
php please auth:migration              # Generate authentication migration files
php please eloquent:import-users       # Import flat-file users to database
php please eloquent:import-roles       # Import flat-file roles to database
php please eloquent:import-groups      # Import flat-file groups to database
php please migrate-dates-to-utc        # Migrate date fields to UTC format
php please nocache:migration           # Generate nocache tag migration files
php please updates:run                 # Run pending Statamic update scripts
```

## Starter Kits

```bash
php please starter-kit:init            # Initialize starter kit configuration
php please starter-kit:export          # Export current project as a starter kit
php please starter-kit:install         # Install a starter kit into current project
```

## Updating

```bash
composer update statamic/cms --with-dependencies   # Update Statamic via Composer
statamic update                                    # Update Statamic via CLI shorthand
```
