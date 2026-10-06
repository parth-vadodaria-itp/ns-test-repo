---
name: review-php-laravel
description: >
  Code review rules for PHP 8.2 + Laravel 12 projects. Covers Eloquent ORM patterns,
  Form Request validation, route model binding, service-layer separation, and
  end-of-life version detection.
  Activate for detected_stack = "php-laravel" or any PHP diff.
metadata:
  stack: php-laravel
  role: reviewer
  version: "1.0"
---

# Review Skill — PHP / Laravel

## Trigger
`detected_stack = "php-laravel"` OR diff contains `composer.json`, `.php`, or `.blade.php` files.

## Rule Set

### SECURITY (prefix: php-sec)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| php-sec-001 | critical | `APP_KEY` not using `base64:` prefix or set to a short non-random string |
| php-sec-002 | critical | `APP_DEBUG=true` in non-development `.env` file — exposes stack traces |
| php-sec-003 | critical | Raw SQL via `DB::statement("SELECT ... " . $input)` — string concatenation with user input |
| php-sec-004 | major    | Route missing auth middleware (`auth:sanctum`, `auth:api`, or JWT guard) when auth is configured |
| php-sec-005 | major    | `$fillable` not defined on Eloquent model — mass assignment vulnerability |

### CORRECTNESS (prefix: php-cor)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| php-cor-001 | major    | Controller `store()` method returning `response()->json($data, 200)` — must return 201 |
| php-cor-002 | major    | `authorize()` method in Form Request returns `false` — should return `true` for authenticated routes |
| php-cor-003 | major    | `Model::find()` result used without null check — `$model->field` will throw on null |
| php-cor-004 | minor    | Business logic directly in Controller — should be in a Service class |
| php-cor-005 | minor    | Missing `$table` property on Eloquent model — relies on auto-pluralization which can be wrong |

### DEPENDENCY (prefix: php-dep)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| php-dep-001 | critical | `"laravel/framework": "^11.0"` — EOL March 2026; must be `^12.0` |
| php-dep-002 | major    | `"php": "^8.1"` — minimum should be `^8.2` for Laravel 12 |
| php-dep-003 | minor    | `tymon/jwt-auth` version < `^2.1` — may have breaking changes |

### PERFORMANCE (prefix: php-perf)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| php-perf-001 | major    | Eloquent query inside a `foreach` loop — N+1 pattern; use `with()` eager loading |
| php-perf-002 | minor    | `Model::all()` returned from an API endpoint without pagination |

### MAINTAINABILITY (prefix: php-maint)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| php-maint-001 | major   | Migration missing `$table->timestamps()` — Eloquent `created_at`/`updated_at` won't work |
| php-maint-002 | minor   | Controller method > 50 lines — fat controller; extract to Service |
| php-maint-003 | minor   | PSR-4 namespace doesn't match folder path |
| php-maint-004 | suggestion | `route()` helper not used for URL generation — hardcoded `/api/v1/` path |

## Positive patterns (do NOT flag)
- `authorize()` returning `true` in Form Requests — correct
- `Schema::create()` with `$table->timestamps()` in migrations
- `protected $fillable = [...]` on all models
- Service class injected into Controller via constructor

## Quick checklist for PHP diffs
1. Is `"laravel/framework": "^12.0"` in composer.json?
2. Is `$fillable` defined on every Eloquent model?
3. Do `store()` methods return 201?
4. Is `authorize()` returning `true` in all Form Requests?
5. Does every migration have `$table->timestamps()`?