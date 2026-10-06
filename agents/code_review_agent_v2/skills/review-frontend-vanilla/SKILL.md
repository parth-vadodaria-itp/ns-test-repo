---
name: review-frontend-vanilla
description: >
  Code review rules for Vanilla HTML/CSS/JS frontends. Covers template literal syntax,
  fetch() correctness, DOM safety (XSS via innerHTML), accessibility basics, and
  API integration completeness. Activate for detected_stack = "frontend-vanilla".
metadata:
  stack: frontend-vanilla
  role: reviewer
  version: "1.0"
---

# Review Skill — Vanilla HTML / CSS / JS

## Trigger
`detected_stack = "frontend-vanilla"` OR diff contains `.html`, `.css`, or frontend `.js` files.

## Rule Set

### SECURITY (prefix: fe-sec)

| Rule ID   | Severity | Description |
|-----------|----------|-------------|
| fe-sec-001 | critical | `element.innerHTML = userInput` — XSS vector; use `element.textContent` for user data |
| fe-sec-002 | critical | API key or token hardcoded as JS string literal in frontend source |
| fe-sec-003 | major    | `eval()` called with any variable input |
| fe-sec-004 | minor    | `document.write()` used — deprecated and XSS-risky |

### CORRECTNESS (prefix: fe-cor)

| Rule ID   | Severity | Description |
|-----------|----------|-------------|
| fe-cor-001 | critical | Template literal uses `$variable` without curly braces — e.g. `` `$API_BASE/path` `` instead of `` `${API_BASE}/path` `` — will not interpolate |
| fe-cor-002 | major    | `fetch()` call without checking `response.ok` before using the data |
| fe-cor-003 | major    | Async function using `.then()` chains — inconsistent with project style (should use `async/await`) |
| fe-cor-004 | major    | Inline event handler (`onclick="..."`) used instead of `addEventListener` |
| fe-cor-005 | minor    | `var` declaration used — use `const`/`let` |
| fe-cor-006 | minor    | Missing `try/catch` around `fetch()` call — network errors will be uncaught |

### ACCESSIBILITY (prefix: fe-a11y)

| Rule ID   | Severity | Description |
|-----------|----------|-------------|
| fe-a11y-001 | minor   | `<img>` missing `alt` attribute |
| fe-a11y-002 | minor   | Form input missing associated `<label>` with `for` attribute |
| fe-a11y-003 | suggestion | `<button>` with no text content and no `aria-label` |

### MAINTAINABILITY (prefix: fe-maint)

| Rule ID   | Severity | Description |
|-----------|----------|-------------|
| fe-maint-001 | major   | `PLACEHOLDER` token remaining in any output HTML/JS file — was not replaced |
| fe-maint-002 | minor   | Inline styles used instead of CSS classes |
| fe-maint-003 | minor   | CSS custom property (design token) defined but never used |
| fe-maint-004 | suggestion | Magic string URL (`"/api/v1/..."`) in JS — should use `API_BASE` constant |

## Positive patterns (do NOT flag)
- `` `${API_BASE}/resource` `` — correct template literal interpolation
- `element.textContent = userValue` — safe DOM insertion
- `async function loadItems()` with `try/catch` + `finally` loading state
- `addEventListener("click", handler)` — correct event binding

## Quick checklist for frontend diffs
1. Are all template literals using `${variable}` not `$variable`?
2. Is every `fetch()` checking `response.ok`?
3. Is user data inserted via `textContent`, never `innerHTML`?
4. Are `PLACEHOLDER` tokens absent from all files?
5. Is `API_BASE` a constant at the top of every JS file?