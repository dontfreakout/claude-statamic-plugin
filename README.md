# Statamic Dev — Claude Code Plugin

A comprehensive development assistant for building websites with **Statamic v6**, the flat-file-first CMS built on Laravel.

## Features

### Knowledge Skills (auto-activate)
- **antlers-templating** — Antlers syntax, variables, loops, conditionals, modifiers, tags, partials, layouts
- **content-modeling** — Collections, taxonomies, globals, blueprints, fieldsets, all 46 fieldtypes
- **addon-development** — Custom tags, modifiers, fieldtypes, service providers, Vue components
- **statamic-php-api** — Facades, query builder, events, computed values, view models
- **routing-and-caching** — Collection routes, static routes, Stache, static caching, deployment
- **assets-forms-search** — Asset containers, Glide image manipulation, forms, search configuration

### Actionable Skills (slash commands)
- `/statamic-dev:create-collection` — Scaffold a collection with config, blueprint, and templates
- `/statamic-dev:create-blueprint` — Generate blueprint YAML from natural language description
- `/statamic-dev:create-addon` — Scaffold custom tags, modifiers, or fieldtypes

### Agents
- **statamic-validator** — Validates Statamic project structure, blueprints, configs, and conventions

## Installation

### From Marketplace (recommended)

Add the marketplace and install:
```
/plugin marketplace add dontfreakout/statamic-dev
/plugin install statamic-dev@statamic-dev
```

### Direct Plugin Directory

```bash
claude --plugin-dir /path/to/statamic-dev
```

Or add to your project's `.claude/settings.json`:
```json
{
  "plugins": ["/path/to/statamic-dev"]
}
```

## Requirements

- A Statamic v6 project (PHP 8.3+, Laravel 12+)
- Claude Code CLI

## Author

Martin Vlcek (info@dontfreakout.eu)
