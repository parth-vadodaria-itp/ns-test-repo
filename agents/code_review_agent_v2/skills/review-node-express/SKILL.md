---
name: review-node-express
description: >
  Code review rules for Node.js 20 + Express 5.x projects. Covers Express 5 migration
  pitfalls, ESM module patterns, Sequelize ORM usage, async/await correctness, and
  security (JWT, CORS, input validation).
  Activate for detected_stack = "node-express" or any JS/TS diff.
metadata:
  stack: node-express
  role: reviewer
  version: "1.0"
---

# Review Skill — Node.js / Express

## Trigger
`detected_stack = "node-express"` OR diff contains `package.json`, `.js`, or `.ts` files.

## Rule Set

### SECURITY (prefix: node-sec)

| Rule ID      | Severity | Description |
|--------------|----------|-------------|
| node-sec-001 | critical | JWT secret or API key hardcoded as string literal in source file (not `.env`) |
| node-sec-002 | critical | `cors({ origin: "*" })` in production — restrict to known origins |
| node-sec-003 | critical | `eval()` or `Function()` constructor called with user input |
| node-sec-004 | major    | Route missing auth middleware when JWT is configured for the project |
| node-sec-005 | major    | SQL string concatenation with `req.body` / `req.params` values |

### CORRECTNESS (prefix: node-cor)

| Rule ID      | Severity | Description |
|--------------|----------|-------------|
| node-cor-001 | major    | Express 5: `router.del()` used — method was removed; use `router.delete()` |
| node-cor-002 | major    | Express 5: optional route param written as `/:id?` — must be `{/:id}` |
| node-cor-003 | major    | `require()` used instead of `import` — project uses ES modules (`"type": "module"`) |
| node-cor-004 | major    | `try/catch` wrapped around async route handler body — unnecessary in Express 5 (rejected promises auto-propagate) |
| node-cor-005 | major    | POST route sending `res.status(200)` — should be `res.status(201)` for resource creation |
| node-cor-006 | minor    | `.then()` chain used instead of `async/await` — inconsistent with project style |
| node-cor-007 | minor    | `var` declaration used — project should use `const` / `let` only |

### DEPENDENCY (prefix: node-dep)

| Rule ID      | Severity | Description |
|--------------|----------|-------------|
| node-dep-001 | critical | `"express": "^4.x"` in package.json — must be `"express": "^5.1.0"` |
| node-dep-002 | major    | `nodemon` in `"dependencies"` instead of `"devDependencies"` |
| node-dep-003 | minor    | `"type": "module"` missing in package.json when ESM imports are used |

### PERFORMANCE (prefix: node-perf)

| Rule ID      | Severity | Description |
|--------------|----------|-------------|
| node-perf-001 | major   | `findAll()` (Sequelize) called with no `limit` on a list endpoint — unbounded query |
| node-perf-002 | minor   | `await` inside a `for` loop — serial async calls; use `Promise.all()` |

### MAINTAINABILITY (prefix: node-maint)

| Rule ID      | Severity | Description |
|--------------|----------|-------------|
| node-maint-001 | major   | Route handler contains direct DB calls — logic should be in a service layer |
| node-maint-002 | minor   | Global error handler not the last middleware in `server.js` |
| node-maint-003 | minor   | `process.exit(1)` called without logging the error first |
| node-maint-004 | suggestion | Async controller uses `res.json(data)` without checking if `data` is null first |

## Positive patterns (do NOT flag)
- Async route handlers without try/catch — Express 5 feature
- `const router = Router()` + `export default router`
- `{/:id}` optional param syntax — correct Express 5
- `Promise.all([...])` for parallel async operations
- `const API_BASE = '/api/v1'` constant at top of JS files

## Quick checklist for Node.js diffs
1. Is Express `^5.1.0` in package.json?
2. Are all route files using `import`/`export` (no `require`)?
3. Do POST routes return 201?
4. Is `cors({ origin: "*" })` absent (or restricted)?
5. Is the global error handler the LAST `app.use()` in server.js?