---
name: frontend-vanilla
description: >
  Generates a complete vanilla HTML/CSS/JS frontend that connects to any REST API backend.
  Activated ONLY when full-stack generation is explicitly requested or user mentions "generate frontend", "build UI", or "include frontend".
  No frameworks, no build tools, no npm. Reads detected_stack to place files in the
  correct static root for the paired backend.
metadata:
  stack: vanilla-html
  version: "2.1"
---

# Frontend generation — vanilla HTML / CSS / JS

## Philosophy
Generate the MINIMUM files needed for a complete, working UI.
Every screen implied by the acceptance criteria gets a page.
Every API endpoint gets a matching fetch call in JS.
Zero frameworks. Zero build tools. Zero placeholders.

## Static root by backend stack

| detected_stack       | Static root                          | How it's served                    |
|----------------------|--------------------------------------|------------------------------------|
| `java-springboot`    | `src/main/resources/static/`         | Spring Boot auto-serves            |
| `python-flask`       | `static/`                            | Flask auto-serves                  |
| `node-express`       | `public/`                            | `express.static('public')`         |

All file paths below use `static_root` — replace with the correct value above.

## Full file structure to generate

```
static_root/
├── index.html          # main page — always required
├── style.css           # all styles — always required
├── app.js              # all JS for index.html — always required
└── page.html           # one additional file per extra screen (if story needs it)
└── page.js             # companion JS for each extra page (if needed)
```

Generate an extra `.html` + `.js` pair for each distinct screen in the story.
Example: a story with "list products" and "product detail" needs `index.html` + `detail.html`.

## HTML + CSS templates

Use `assets/html-template.html` as the structural base for every `.html` file.
Use `assets/style-template.css` as the base for every `style.css` — copy it in full,
then add any story-specific overrides at the bottom.

### Replacing `PLACEHOLDER` tokens in html-template.html

| Token | Replace with |
|---|---|
| `PROJECT_NAME` | project name from story_discovery |
| `PAGE_TITLE` | page heading (e.g. "Orders", "Products") |
| `PAGE_SUBTITLE` | short description (e.g. "Manage all orders") |
| `CTA_LABEL` | action verb (e.g. "New Order", "Add Product") |
| `FORM_TITLE` | form heading (e.g. "Create Order") |
| `FIELD_1_LABEL` / `FIELD_1_PLACEHOLDER` | first form field |
| `FIELD_2_LABEL` / `FIELD_2_PLACEHOLDER` | second form field |
| `SUBMIT_LABEL` | submit button text |
| `LIST_TITLE` | table section heading |
| `COL_1` / `COL_2` | column headers from endpoint response |
| `ITEMS_LABEL` | plural noun for empty state (e.g. "orders") |
| `STAT_2_LABEL` / `STAT_3_LABEL` | stat card labels for the story |

Add more `<div class="field">` blocks for additional form fields.
Add more `<th>`/`<td>` columns for additional entity fields.
Remove `id="stats-row"` if the story has no summary metrics.
Remove the form panel if the story is read-only.
Never leave a `PLACEHOLDER` token in the final output.

## Reasoning steps (execute before writing any file)

1. Read acceptance criteria — identify every UI screen and interaction.
2. Read `generated_files` (backend output) — extract all API endpoints, request/response shapes.
3. Determine static root from `detected_stack`.
4. Plan file list: one HTML+JS per screen, shared `style.css`.
5. For each screen: map UI interactions → `fetch()` calls to backend endpoints.
6. Generate all files in full — real HTML structure, real CSS, real JS logic.
7. Self-check: every backend endpoint has a `fetch()` call. No `PLACEHOLDER` tokens remain.

## Code standards

### HTML
- Semantic tags: `<header>`, `<main>`, `<section>`, `<article>`, `<form>`, `<table>`
- Always: `<meta charset="UTF-8">` and `<meta name="viewport" ...>`
- Link stylesheet: `<link rel="stylesheet" href="style.css">`
- Load script deferred: `<script src="app.js" defer></script>`
- Navigation links between pages if more than one HTML file

### CSS
- CSS custom properties at `:root` for all colors, spacing, radii
- Mobile-first responsive using `@media (min-width: 768px)`
- Spacing in `rem`. No inline styles anywhere.
- Style: clean, neutral, works without a design system
- Minimal required classes: `.container`, `.card`, `.btn`, `.btn-primary`, `.btn-secondary`,
  `.form-group`, `.data-table`, `.message`, `.error`, `.loading`, `.hidden`, `.empty-state`

### JavaScript
- `const`/`let` only — no `var`
- `async/await` for all fetch — no `.then()` chains
- API base URL constant at top: `const API_BASE = '/api/v1';`
- Always `try/catch` around fetch calls — show user-visible error in `.message.error` div
- DOM via `getElementById`/`querySelector` — use `textContent` not `innerHTML` for user data
- Loading state: show `.loading` div before fetch, hide after
- Empty state: show `.empty-state` div when list is empty

### Fetch pattern (always follow this exact structure)

> **CRITICAL — JavaScript template literals require `${...}` curly braces.**
> `$variable` without curly braces is a literal dollar sign followed by text — it does NOT
> interpolate. Every variable inside a backtick string MUST use `$variable` syntax.

```js
async function loadItems() {
  const loading = document.getElementById('loading');
  const errorBanner = document.getElementById('error-banner');
  loading.classList.remove('hidden');
  errorBanner.classList.add('hidden');
  try {
    const res = await fetch(`${API_BASE}/resource`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const data = await res.json();
    renderItems(data);
  } catch (err) {
    errorBanner.textContent = `Failed to load: ${err.message}`;
    errorBanner.classList.remove('hidden');
  } finally {
    loading.classList.add('hidden');
  }
}
```

**Hard JS template literal rules — apply to every single generated file:**
- ALWAYS: `` `${API_BASE}/resource` `` — never `` `$API_BASE/resource` ``
- ALWAYS: `` `HTTP ${res.status}` `` — never `` `HTTP $res.status` ``
- ALWAYS: `` `${err.message}` `` — never `` `$err.message` ``
- ALWAYS: `` `${item.id}` `` — never `` `$item.id` ``
- Every `${...}` interpolation MUST have the curly braces

## Output format

```json
{
  "frontend_files": [
    { "path": "static_root/index.html",   "content": "..." },
    { "path": "static_root/style.css",    "content": "..." },
    { "path": "static_root/app.js",       "content": "..." }
  ]
}
```

Add additional `{ "path": ..., "content": ... }` entries for each extra page.

## Hard rules
- NEVER use a JS framework (React, Vue, Angular, Svelte, jQuery, Bootstrap JS)
- NEVER use inline event handlers (`onclick="..."`) — `addEventListener` only
- NEVER use `document.write()` or `eval()`
- NEVER leave a `PLACEHOLDER` token in any output file
- NEVER use `$variable` without curly braces inside template literals — always `$variable`
- CSS framework (Bootstrap CSS via CDN) only if explicitly requested
- Generate as many HTML/JS files as the story's screens require — no artificial file cap