---
name: backend-node-express
description: >
  Generates a complete Node.js Express backend. DEFAULTS ONLY — resolved_config takes precedence.
metadata:
  stack: node-express
  version: "4.0"
---

# Backend Skill — Node.js / Express

## Section 1 — Metadata & Trigger
**Trigger:** `resolved_config.technology.language` contains "Node" or "JavaScript" OR `detected_stack = "node-express"`.
**Role:** Defaults only — resolved_config overrides every field in Section 2.

## Section 2 — Defaults (null resolved_config field → use these)

| Field | Default |
|---|---|
| `technology.framework` | Express 5.x (`^5.1.0`) |
| `technology.version` | Node.js 20 LTS |
| `technology.orm` | Sequelize 6.x |
| `technology.package_manager` | npm |
| `architecture.pattern` | MVC (routes/controllers/services) |
| `architecture.custom_structure` | `[]` → config/, models/, routes/, controllers/, services/ |
| `database.primary` | SQLite (`sqlite:./dev.db`) |
| `database.driver` | sqlite3 |
| `api.auth_mechanism` | null (no auth) |
| `docker.port` | 3000 |

## Section 3 — Override Protocol

**Framework override:**
- `Fastify` → replace Express with `fastify@^4.0.0`; use `fastify.get/post/put/delete()` pattern; replace `express.json()` with built-in JSON parsing; adjust server.js structure
- `Koa` → replace with `koa@^2.15.0`, `koa-router`, `koa-body`

**Database override:**
- `PostgreSQL` → replace `sqlite3` with `pg@^8.12.0`; DATABASE_URL: `postgresql://user:password@localhost:5432/dbname`
- `MySQL` → replace with `mysql2@^3.9.0`; DATABASE_URL: `mysql://user:password@localhost:3306/dbname`
- `MongoDB` → replace Sequelize with `mongoose@^8.4.0`; use Schema/Model pattern

**ORM override:**
- `Prisma` → replace Sequelize with `@prisma/client`; generate `schema.prisma`; use `prisma.entity.findMany()` style
- `TypeORM` → add `typeorm@^0.3.20`; use entity class decorators; generate `data-source.ts`
- `Knex` → add `knex@^3.1.0`; use query builder style

**Auth override:**
- `JWT` → add `jsonwebtoken@^9.0.0`, `bcrypt@^5.1.1`; generate `auth.routes.js`, `AuthController.js`, JWT middleware; add auth middleware to protected routes
- `Passport` → add `passport@^0.7.0`, `passport-jwt@^4.0.1`; generate `passport.config.js`

**Architecture override:**
- `resolved_config.architecture.custom_structure[]` NOT empty → replace default folder layout; update imports accordingly
- `hexagonal` → domain/, application/, infrastructure/ layers

**Extra libraries:**
- Merge ALL `resolved_config.technology.extra_libraries[]` into package.json `"dependencies"`

**Port override:**
- `resolved_config.docker.port` NOT null → update `PORT=<port>` in `.env` and `.env.example`

## Section 4 — Compliance Overlays (additive)

- **HIPAA** → add request logging middleware stripping PII fields; no sensitive data in error messages
- **GDPR** → add `X-Data-Classification` response header middleware; audit log on data write routes
- **SOC2** → all write endpoints require auth; structured audit log in service layer

## Section 5 — Hard Rules (non-overridable)

- NEVER use `"express": "^4.x"` — always `"express": "^5.1.0"`
- NEVER use `try/catch` + `next(err)` in async route handlers — Express 5 handles this
- NEVER use `router.del()` — use `router.delete()`
- NEVER use optional param syntax `/:param?` — use `{/:param}` in Express 5
- NEVER use `var` or `require()` — ES modules (`import`/`export`) only
- NEVER use `.then()` chains — `async/await` only
- NEVER omit `config/database.js`, `.env.example`, `.gitignore`, `README.md`
- NEVER use `nodemon` in CMD — dev tool only; use `node server.js`
- Global error handler MUST be the last middleware in `server.js`
- Generate as many entity files as the story requires — no cap

---

## Philosophy
Minimum files for the app to run. Zero placeholders. Zero stubs. No empty `{}` bodies.

## Stack (defaults — overridable per Section 2)
Node.js 20 LTS, **Express 5.x** (stable since March 2025), Sequelize 6.x, SQLite3, dotenv, Joi, cors

## Default project structure
```
project-name/
├── package.json / .env / .env.example / .gitignore / README.md
├── server.js
├── config/database.js
├── models/index.js + entity.js
├── routes/index.js + entity.routes.js
├── controllers/entity.controller.js
└── services/entity.service.js
```
> Override when `resolved_config.architecture.custom_structure[]` is NOT empty.

## Reasoning steps
1. Apply all Section 3 overrides from resolved_config.
2. Extract entities from acceptance_criteria.
3. Map each entity → HTTP method, path, request/response shape, status codes.
4. Design Sequelize models or ORM-specific models per override.
5. Generate routes + controller + service per entity.
6. Self-check: every AC has a route; every route calls controller → service.

## Key Express 5 rules
- **Async routes — NO try/catch needed** (Express 5 catches rejected promises automatically)
- **Async controller pattern:** `export const getById = async (req, res) => { const item = await service.findById(req.params.id); res.json(item); };`
- **Optional params:** use `{/:param}` syntax, NOT `/:param?`
- **No regex sub-expressions** in routes (ReDoS risk, removed in Express 5)

## Output format
```json
{
  "status": "success",
  "data": {
    "story_id": "<from story_discovery>",
    "detected_stack": "node-express",
    "resolved_config": {},
    "extracted_requirements": {},
    "generated_files": [
      { "path": "package.json", "content": "..." },
      { "path": ".env", "content": "..." },
      { "path": ".env.example", "content": "..." },
      { "path": ".gitignore", "content": "..." },
      { "path": "README.md", "content": "..." },
      { "path": "server.js", "content": "..." },
      { "path": "config/database.js", "content": "..." },
      { "path": "models/index.js", "content": "..." },
      { "path": "models/entity.js", "content": "..." },
      { "path": "routes/index.js", "content": "..." },
      { "path": "routes/entity.routes.js", "content": "..." },
      { "path": "controllers/entity.controller.js", "content": "..." },
      { "path": "services/entity.service.js", "content": "..." }
    ],
    "files_generated": 13,
    "language": "javascript"
  }
}
```