---
name: backend-python-flask
description: >
  Generates a complete Python Flask backend. DEFAULTS ONLY — resolved_config takes precedence.
metadata:
  stack: python-flask
  version: "4.0"
---

# Backend Skill — Python / Flask

## Section 1 — Metadata & Trigger
**Trigger:** `resolved_config.technology.language` contains "Python" OR `detected_stack = "python-flask"`.
**Role:** Defaults only — resolved_config overrides every field in Section 2.

## Section 2 — Defaults (null resolved_config field → use these)

| Field | Default |
|---|---|
| `technology.framework` | Flask 3.x (`>=3.1.0`) |
| `technology.version` | Python 3.11 |
| `technology.orm` | Flask-SQLAlchemy 3.x |
| `technology.package_manager` | pip |
| `architecture.pattern` | App factory (create_app) |
| `architecture.custom_structure` | `[]` → models/, routes/, services/ |
| `database.primary` | SQLite (`sqlite:///dev.db`) |
| `database.driver` | built-in sqlite3 |
| `api.auth_mechanism` | null (no auth) |
| `docker.port` | 5000 |

## Section 3 — Override Protocol

**Framework override:**
- `FastAPI` → **REPLACE the entire requirements.txt** with the following (do NOT keep Flask packages):
  ```
  fastapi>=0.115.0
  uvicorn[standard]>=0.34.0
  sqlalchemy>=2.0.0
  alembic>=1.13.0
  pydantic>=2.0.0
  python-dotenv>=1.0.1
  ```
  **REMOVE:** `Flask`, `Flask-SQLAlchemy`, `Flask-Migrate`, `marshmallow` (all Flask-specific — keeping them causes import conflicts)
  **Also update:** `config.py` to use `sqlalchemy` directly (no `flask_sqlalchemy`); models to use `sqlalchemy.orm` declarative base; routes to use `APIRouter` instead of Blueprint; app.py to be a FastAPI app object (`app = FastAPI()`)

  **database.py — generate this file as part of the FastAPI override (Bug 12 fix):**
  ```python
  import os
  from sqlalchemy import create_engine
  from sqlalchemy.orm import DeclarativeBase, sessionmaker

  DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./dev.db")
  engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
  SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

  class Base(DeclarativeBase):
      pass
  ```

  **Also generate these Alembic bootstrap files:**

  **`alembic.ini`** (minimal):
  ```
  [alembic]
  script_location = alembic
  # URL is set programmatically in env.py from DATABASE_URL environment variable
  sqlalchemy.url = placeholder
  ```

  **`alembic/env.py`** (autogenerate-enabled):
  ```python
  import os
  from logging.config import fileConfig
  from sqlalchemy import engine_from_config, pool
  from alembic import context
  from database import Base  # import your declarative Base

  config = context.config

  # Bug 14 fix: Override with runtime DATABASE_URL environment variable
  # ConfigParser will NOT read %(DATABASE_URL)s from the OS environment
  database_url = os.getenv("DATABASE_URL", "sqlite:///./dev.db")
  config.set_main_option("sqlalchemy.url", database_url)

  if config.config_file_name is not None:
      fileConfig(config.config_file_name)
  target_metadata = Base.metadata

  def run_migrations_online():
      connectable = engine_from_config(
          config.get_section(config.config_ini_section, {}),
          prefix="sqlalchemy.",
          poolclass=pool.NullPool,
      )
      with connectable.connect() as connection:
          context.configure(connection=connection, target_metadata=target_metadata)
          with context.begin_transaction():
              context.run_migrations()

  run_migrations_online()
  ```

  **`alembic/versions/` directory** — empty, created as a placeholder

**Database override:**
- `PostgreSQL` → requirements.txt: add `psycopg2-binary>=2.9.9`; DATABASE_URL: `postgresql://user:password@localhost:5432/dbname`
- `MySQL` → add `PyMySQL>=1.1.0`; DATABASE_URL: `mysql+pymysql://user:password@localhost:3306/dbname`
- Use `resolved_config.database.driver` in SQLAlchemy connection string if specified

**Auth override:**
- `JWT` → add `PyJWT>=2.8.0`, `Flask-JWT-Extended>=4.6.0`; generate `auth_routes.py`, `AuthController`; add JWT init in `app.py`
- `OAuth2` → add `authlib>=1.3.0`; generate OAuth2 blueprint

**Architecture override:**
- `resolved_config.architecture.custom_structure[]` NOT empty → replace default folder layout; adjust Python package `__init__.py` files accordingly
- `hexagonal` → domain/, application/, infrastructure/ layers

**ORM override:**
- `SQLAlchemy` (explicit) → keep Flask-SQLAlchemy
- `asyncpg` → use async pattern with `asyncpg>=0.29.0` + `SQLAlchemy[asyncio]>=2.0.0`
- `Tortoise` → replace Flask-SQLAlchemy with Tortoise ORM

**Extra libraries:**
- Merge ALL `resolved_config.technology.extra_libraries[]` into `requirements.txt`

**Port override:**
- `resolved_config.docker.port` NOT null → update `PORT=<port>` in `.env` and `.env.example`; update gunicorn `--bind` in CMD

## Section 4 — Compliance Overlays (additive)

- **HIPAA** → `FLASK_DEBUG=0`; add request logging middleware stripping PII fields; audit log on write operations
- **GDPR** → add `X-Data-Classification` response header middleware; no PII in logs
- **SOC2** → all write endpoints require auth; audit log entries in service layer

## Section 5 — Hard Rules (non-overridable)

- NEVER use `FLASK_ENV` in any file — removed in Flask 2.3+; use `FLASK_DEBUG=1`
- NEVER use `db.Column` — use `Mapped` + `mapped_column` (SQLAlchemy 2.x)
- NEVER use `Model.query` or `db.Query` — use `db.session.execute(db.select(...))`
- NEVER pin exact versions with `==` — use `>=` bounds
- NEVER use `gunicorn<26.0.0` — minimum `gunicorn>=26.0.0`
- NEVER use TODO, TBD, FIXME, bare `pass`, or empty bodies
- NEVER use bare `except:` — always `except SpecificError as e:`
- NEVER omit `__init__.py` in each package directory
- ALWAYS use the app factory pattern in `app.py`
- Generate as many entity files as the story requires — no cap

---

## Philosophy
Minimum files for the app to actually run. Zero placeholders. Zero stubs.

## Stack (defaults — overridable per Section 2)
Python 3.11+, **Flask 3.x** (3.1.3), Flask-SQLAlchemy 3.x + SQLAlchemy 2.x style,
Flask-Migrate, marshmallow, python-dotenv, gunicorn>=26.0.0

## Default project structure
```
project-name/
├── requirements.txt / .env / .env.example / .gitignore / README.md
├── config.py / app.py
├── models/__init__.py + entity.py
├── routes/__init__.py + entity_routes.py
└── services/__init__.py + entity_service.py
```
> Override when `resolved_config.architecture.custom_structure[]` is NOT empty.

## Reasoning steps
1. Apply all Section 3 overrides from resolved_config.
2. Extract entities from acceptance_criteria.
3. Map each entity → HTTP method, URL rule, request/response shape, status codes.
4. Design SQLAlchemy 2.x models: `Mapped` + `mapped_column`.
5. Generate model + routes + service per entity.
6. Self-check: every AC has a route; every route calls a service.

## File content rules

### requirements.txt (defaults)
```
Flask>=3.1.0
Flask-SQLAlchemy>=3.1.1
Flask-Migrate>=4.0.7
marshmallow>=3.21.0
python-dotenv>=1.0.1
gunicorn>=26.0.0
```
Merge `extra_libraries[]` into this file.

### config.py — Config/DevelopmentConfig/ProductionConfig with DATABASE_URL env var.
### app.py — factory pattern `create_app(env="default")`, register blueprints, error handlers.
### .env — `FLASK_DEBUG=1, FLASK_APP=app.py, SECRET_KEY=dev-secret-key, DATABASE_URL=sqlite:///dev.db, PORT=5000`
### Models — SQLAlchemy 2.x: `Mapped[type]`, `mapped_column()`, explicit `__tablename__`, `to_dict()`
### Services — `db.session.execute(db.select(...))`, raise `ValueError`/`LookupError` for 400/404
### Routes — Blueprint pattern, `jsonify()`, explicit status codes

## Output format
```json
{
  "status": "success",
  "data": {
    "story_id": "<from story_discovery>",
    "detected_stack": "python-flask",
    "resolved_config": {},
    "extracted_requirements": {},
    "generated_files": [
      { "path": "requirements.txt", "content": "..." },
      { "path": ".env", "content": "..." },
      { "path": ".env.example", "content": "..." },
      { "path": ".gitignore", "content": "..." },
      { "path": "README.md", "content": "..." },
      { "path": "config.py", "content": "..." },
      { "path": "app.py", "content": "..." },
      { "path": "models/__init__.py", "content": "..." },
      { "path": "models/entity.py", "content": "..." },
      { "path": "routes/__init__.py", "content": "..." },
      { "path": "routes/entity_routes.py", "content": "..." },
      { "path": "services/__init__.py", "content": "..." },
      { "path": "services/entity_service.py", "content": "..." }
    ],
    "files_generated": 13,
    "language": "python"
  }
}
```