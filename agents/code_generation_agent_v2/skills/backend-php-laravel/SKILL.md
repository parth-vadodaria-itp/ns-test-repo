---
name: backend-php-laravel
description: >
  Generates a complete PHP + Laravel REST API. DEFAULTS ONLY — resolved_config takes precedence.
metadata:
  stack: php-laravel
  version: "3.0"
---

# Backend Skill — PHP / Laravel

## Section 1 — Metadata & Trigger
**Trigger:** `resolved_config.technology.language` contains "PHP" OR `detected_stack = "php-laravel"`.
**Role:** Defaults only — resolved_config overrides every field in Section 2.

## Section 2 — Defaults (null resolved_config field → use these)

| Field | Default |
|---|---|
| `technology.framework` | Laravel 12 (`^12.0`) |
| `technology.version` | PHP 8.2 |
| `technology.orm` | Eloquent |
| `technology.package_manager` | Composer |
| `architecture.pattern` | MVC |
| `architecture.custom_structure` | `[]` → app/Http/Controllers, app/Models, app/Services |
| `database.primary` | SQLite (`DB_CONNECTION=sqlite`) |
| `database.driver` | pdo_sqlite |
| `api.auth_mechanism` | null (no auth) |
| `docker.port` | 8000 — update APP_URL in .env |

## Section 3 — Override Protocol

**Database override:**
- `MySQL` → `.env`: `DB_CONNECTION=mysql, DB_HOST=127.0.0.1, DB_PORT=3306, DB_DATABASE=<project>, DB_USERNAME=root, DB_PASSWORD=`
- `PostgreSQL` → `.env`: `DB_CONNECTION=pgsql, DB_HOST=127.0.0.1, DB_PORT=5432`
- Remove `DB_DATABASE=database/database.sqlite` when not SQLite

**Auth override:**
- `JWT` → add `"tymon/jwt-auth": "^2.1"` to composer.json; generate `AuthController.php`, `JwtMiddleware.php`, auth routes group
- `Sanctum` → add `"laravel/sanctum": "^4.0"`; use token auth pattern

**Architecture override:**
- `resolved_config.architecture.custom_structure[]` NOT empty → replace default MVC folders with these paths; adjust PSR-4 namespaces
- `repository` pattern → add `app/Repositories/EntityRepository.php` per entity
- `hexagonal` → use domain/application/infrastructure layers

**Extra libraries:**
- Merge ALL `resolved_config.technology.extra_libraries[]` into `composer.json "require"`

**Port override:**
- `resolved_config.docker.port` NOT null → update `APP_URL=http://localhost:<port>` in `.env` and `.env.example`

## Section 4 — Compliance Overlays (additive)

- **HIPAA** → `APP_DEBUG=false`; add request logging middleware that strips PII fields
- **GDPR** → add `GdprHeadersMiddleware.php`; no PII in logs
- **SOC2** → all endpoints require auth; audit-log on create/update/delete in Service layer

## Section 5 — Hard Rules (non-overridable)

- NEVER use `"laravel/framework": "^11.0"` — EOL March 12 2026; always use `^12.0`
- NEVER use raw SQL — Eloquent only
- NEVER use TODO, TBD, FIXME, or empty method bodies
- NEVER omit `.env.example`, `.gitignore`, `README.md`
- ALWAYS `authorize()` returns `true` in Form Requests
- ALWAYS namespaces follow PSR-4 and match folder paths
- Generate as many entity files as the story requires — no cap

---

## Philosophy
Minimum files for the app to actually run end-to-end. Zero placeholders. Zero stubs.

## Stack (defaults — overridable per Section 2)
PHP 8.2+, **Laravel 12** (`^12.0`), Eloquent ORM, SQLite (dev), Composer

## Default project structure
```
project-name/
├── composer.json
├── .env / .env.example / .gitignore / README.md
├── routes/api.php
├── app/Http/Controllers/EntityController.php
├── app/Http/Requests/StoreEntityRequest.php + UpdateEntityRequest.php
├── app/Models/Entity.php
├── app/Services/EntityService.php
└── database/migrations/YYYY_MM_DD_create_entities_table.php
```
> Override layout when `resolved_config.architecture.custom_structure[]` is NOT empty.

## Reasoning steps
1. Apply all Section 3 overrides from resolved_config.
2. Extract entities from acceptance_criteria.
3. Map each entity → HTTP method, route, request/response, status codes.
4. Design Eloquent model: fillable, casts, table.
5. Generate controller + service + form requests + model + migration per entity.
6. Self-check: every AC has a route; every route calls controller → service.

## File content rules

### composer.json
```json
{
    "name": "project/name",
    "require": { "php": "^8.2", "laravel/framework": "^12.0" },
    "require-dev": { "laravel/pint": "^1.0" },
    "autoload": { "psr-4": { "App\\": "app/" } },
    "scripts": {
        "start": ["@php artisan serve"],
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover --ansi"
        ]
    }
}
```
Merge `extra_libraries[]` into "require". NEVER use `^11.0`.

### .env (dev)
```
APP_NAME=ProjectName
APP_ENV=local
APP_KEY=base64:GENERATE_WITH_ARTISAN
APP_DEBUG=true
APP_URL=http://localhost:8000
DB_CONNECTION=sqlite
DB_DATABASE=database/database.sqlite
```
Update DB vars per Section 3 database override. Update APP_URL port per Section 3 port override.

### routes/api.php
```php
<?php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\EntityController;
Route::apiResource('entities', EntityController::class);
```

### Controllers: extend Controller, use Form Requests, return `response()->json($data, $status)`, `abort(404)` on not-found.
### Services: all Eloquent ops here, controllers call service methods only.
### Models: extend Model, define `$fillable`, `$casts`, `$table` explicitly.
### Migrations: `Schema::create()` with proper column types + `$table->timestamps()`.
### README: prerequisites, install steps, env vars table, API endpoints table.

## Output format
```json
{
  "status": "success",
  "data": {
    "story_id": "<from story_discovery>",
    "detected_stack": "php-laravel",
    "resolved_config": {},
    "extracted_requirements": {},
    "generated_files": [
      { "path": "composer.json", "content": "..." },
      { "path": ".env", "content": "..." },
      { "path": ".env.example", "content": "..." },
      { "path": ".gitignore", "content": "..." },
      { "path": "README.md", "content": "..." },
      { "path": "routes/api.php", "content": "..." },
      { "path": "app/Http/Controllers/EntityController.php", "content": "..." },
      { "path": "app/Http/Requests/StoreEntityRequest.php", "content": "..." },
      { "path": "app/Http/Requests/UpdateEntityRequest.php", "content": "..." },
      { "path": "app/Models/Entity.php", "content": "..." },
      { "path": "app/Services/EntityService.php", "content": "..." },
      { "path": "database/migrations/2024_01_01_000000_create_entities_table.php", "content": "..." }
    ],
    "files_generated": 12,
    "language": "php"
  }
}
```