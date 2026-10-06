---
name: review-python-flask
description: >
  Code review rules for Python 3.11 + Flask 3.x / FastAPI projects. Covers SQLAlchemy 2.x
  patterns, Flask app-factory correctness, security (secret key, debug mode), and dependency
  hygiene. Activate for detected_stack = "python-flask" or any Python diff.
metadata:
  stack: python-flask
  role: reviewer
  version: "1.0"
---

# Review Skill — Python / Flask

## Trigger
`detected_stack = "python-flask"` OR diff contains `requirements.txt` or `.py` files.

## Rule Set

### SECURITY (prefix: py-sec)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| py-sec-001 | critical | `SECRET_KEY` hardcoded as a non-env string literal (`app.config["SECRET_KEY"] = "hardcoded"`) |
| py-sec-002 | critical | `FLASK_DEBUG=True` or `app.run(debug=True)` without env-guard — must be `os.getenv("FLASK_DEBUG", "0") == "1"` |
| py-sec-003 | critical | SQL built by string concatenation passed to `db.session.execute()` — use parameterized queries |
| py-sec-004 | major    | Route missing `@login_required` or JWT guard when SecurityConfig or JWT extension is configured |
| py-sec-005 | major    | User-controlled input used directly in `os.system()`, `subprocess.run()`, or `eval()` |

### CORRECTNESS (prefix: py-cor)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| py-cor-001 | major    | POST route returning `200` — should return `201` for resource creation |
| py-cor-002 | major    | `db.session.execute(db.select(Entity))` result used without `.scalars().all()` — returns `CursorResult` not list |
| py-cor-003 | major    | Old-style `Model.query.filter_by()` used — SQLAlchemy 2.x requires `db.session.execute(db.select(...))` |
| py-cor-004 | major    | `db.Column` used instead of `Mapped[type]` + `mapped_column()` — SQLAlchemy 1.x syntax in a 2.x project |
| py-cor-005 | major    | Bare `except:` clause — always `except SpecificError as e:` |
| py-cor-006 | minor    | `app.run()` called outside `if __name__ == "__main__":` guard |
| py-cor-007 | minor    | Missing `__tablename__` on SQLAlchemy model — table name won't be deterministic |

### DEPENDENCY (prefix: py-dep)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| py-dep-001 | critical | Flask version pinned with `==` or version < 3.1 in requirements.txt |
| py-dep-002 | major    | `gunicorn` missing from requirements.txt (needed for production) |
| py-dep-003 | major    | `gunicorn<26.0.0` pinned — must be `>=26.0.0` |
| py-dep-004 | major    | `FLASK_ENV` used — removed in Flask 2.3+; use `FLASK_DEBUG=1` |
| py-dep-005 | minor    | `Flask-SQLAlchemy` present but `Flask-Migrate` missing (needed for schema migrations) |

### PERFORMANCE (prefix: py-perf)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| py-perf-001 | major    | Query inside a Python loop — classic N+1 pattern |
| py-perf-002 | minor    | `db.session.execute(db.select(Entity))` without `.limit()` on a collection endpoint — unbounded query |

### MAINTAINABILITY (prefix: py-maint)

| Rule ID    | Severity | Description |
|------------|----------|-------------|
| py-maint-001 | major    | `pass` as the only statement in a route handler or service method — stub not implemented |
| py-maint-002 | minor    | Business logic directly in route handler — should delegate to a service function |
| py-maint-003 | minor    | Missing `__init__.py` in a package directory |
| py-maint-004 | suggestion | `jsonify(item.to_dict())` can be replaced with `return item.to_dict(), 201` in Flask 3.x |

## Positive patterns (do NOT flag)
- App factory pattern `create_app(env="default")`
- `Mapped[str]` + `mapped_column(String(255))` — correct SQLAlchemy 2.x syntax
- `db.session.execute(db.select(Entity)).scalars().all()`
- `raise ValueError` / `raise LookupError` in service methods for 400/404 responses
- `Blueprint` + `register_blueprint` pattern

## Quick checklist for Flask diffs
1. Is Flask >= 3.1.0 in requirements.txt?
2. Is `SECRET_KEY` read from environment, not hardcoded?
3. Are all models using `Mapped[type]` + `mapped_column()`?
4. Does every POST return 201?
5. Is `gunicorn>=26.0.0` in requirements.txt?
6. Is the app factory (`create_app`) pattern used?