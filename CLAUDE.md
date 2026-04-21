# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-file static reference site for AWS services, intended as study material for AWS certifications (CLF-C02 etc.). Deployed to GitHub Pages — the file **must** remain named `index.html` at the repo root.

There is no build system, no package manager, no tests, no dependencies to install. Tailwind CSS is loaded from its CDN at runtime; all JavaScript is inline.

## Running locally

Open `index.html` directly in a browser, or serve the directory:

```bash
python3 -m http.server 8000     # then visit http://localhost:8000/
```

## Architecture (everything is inside `index.html`)

The page has three tabs driven by `currentTab` and `setTab()`:

- **Services** — card grid rendered by `render()`
- **Matrix** — same data as a table with GCP/Azure columns, rendered by `renderMatrix()`
- **Acronyms** — separate section rendered by `renderAcronyms()`

Three inline data structures are the single source of truth — edits go here:

- `CAT` (~line 98): category → `{bg, tc, dkbg, dktc}` color scheme for light/dark mode. A service's category **must** exist as a key in `CAT` or the badge colors fall back to a default.
- `S` (~line 123): array of services. Required fields: `r` (AWS name), `h` (headline), `d` (description), `c` (category, must match a `CAT` key). Optional: `gcp`, `az` (cross-cloud equivalents shown in the Matrix tab and in collapsible `<details>` on cards).
- `ACRONYMS` (~line 571): array of `{a, f, d}` (acronym, full name, description). Sorted alphabetically at load via `.sort(...)` at the end of the array literal.

`deduped` is built from `S` using `r+c` as the dedup key — duplicate service+category combinations are silently dropped. Pills for category filtering are built from `Object.keys(CAT)` with `"All"` prepended, not from categories actually present in `S`, so orphan pills are possible if you add a category to `CAT` but no services use it.

## Adding content

- **New service**: append one object to `S` in the relevant `// ── <Category> ──` section. Keep the aligned-column formatting convention used by the surrounding entries.
- **New category**: add an entry to `CAT` (both light and dark colors) *before* using it in any service's `c` field.
- **New acronym**: append `{a, f, d}` to `ACRONYMS`. The trailing `.sort(...)` handles ordering.

## Reading large files

**Always use Gemini CLI** for large files (PDFs, docs > 50KB) to save Claude tokens:

```bash
gemini -p "Extract X from this document" < large-file.pdf
gemini -p "..." < iam-ug.pdf > /tmp/out.json   # save output for Claude to read
```

Use the `gemini-read` skill — it has more usage patterns.

## Conventions

- Tone in service descriptions is plain-English and slightly irreverent (e.g. "The OG cloud service", "Managed MongoDB without running MongoDB"). Match the existing voice rather than AWS marketing copy.
- Keep `h` (headline) to one short line. `d` (description) is 1–2 sentences.
- Dark mode state persists via `localStorage['dark']`; test both modes when changing colors.
- Keyboard: `/` focuses the active tab's search input, `Esc` clears/blurs.
