# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Source content for **developer.supervisely.com** — a GitBook-hosted documentation site for the Supervisely platform (Python SDK, REST API, app framework, UI widgets). Pure Markdown + assets, no application code, no build/test/lint pipeline. GitBook ingests this repo directly; there is no local preview command in-tree.

Authoritative configuration:
- `.gitbook.yaml` — GitBook root + redirects. When renaming or moving a page, add a redirect here from the old path so external links keep working.
- `SUMMARY.md` — the site's table of contents. Every new page MUST be linked from here, or GitBook will not render it in the sidebar.
- `README.md` — landing page of the site (not a typical project README).

## Repository layout

Three top-level documentation sections, each mirrored as a folder:

- `getting-started/` — installation, auth, env vars, Supervisely annotation JSON format, Python SDK tutorials (images / videos / point-clouds / volumes / common), CLI.
- `app-development/` — app framework: `basics/` (config.json, public/private apps), `apps-with-gui/`, `create-import-app/`, `create-export-app/`, `neural-network-integration/` (inference, serving, training), `advanced/` (debugging, custom widgets, MLOps, multi-user sessions, in-depth chapters), `widgets/` (reference for every UI widget, grouped by category).
- `advanced-user-guide/` — objects binding, automating Supervisely via SDK/API (start/stop apps, user management, labeling jobs).

Section folders typically contain a `README.md` that acts as the section index in GitBook.

## Doc conventions (match existing pages)

- **GitBook flavored Markdown.** Pages may use GitBook-specific syntax: `{% hint style="info" %}`, `{% embed url="..." %}`, `<details><summary>…</summary>`, and YAML front matter with `description:`. Preserve these when editing.
- **Widget reference pages** (`app-development/widgets/**/*.md`) follow a fixed template: Introduction → Function signature (Python code block) → Parameters table → per-parameter subsections → usage example. Keep this shape when adding a new widget page.
- **Images / GIFs** live on external hosts (GitHub releases, user-images.githubusercontent.com) rather than the repo. Local assets live under `.gitbook/assets/`. Prefer the external pattern already used in nearby pages.
- **Code samples** are Python SDK examples assuming `import supervisely as sly` and `api = sly.Api.from_env()`. Don't invent new auth patterns — mirror the snippets already in `getting-started/`.
- **Cross-links between pages** use relative `.md` paths (e.g. `../widgets/input/input.md`). GitBook resolves these to clean URLs at build time.

## When adding or moving pages

1. Create the `.md` under the right section folder (a section index goes in that folder's `README.md`).
2. Add an entry to `SUMMARY.md` at the correct nesting depth — indentation matters and controls the sidebar tree.
3. If renaming/moving an existing page, add a redirect line under `redirects:` in `.gitbook.yaml` from the old path to the new one.
4. Don't edit `.DS_Store` files — they are macOS junk that shows up in `git status` but should not be committed (already present in the working tree from prior commits, leave alone unless cleaning).

## What NOT to do

- Don't add build tooling, package.json, linters, or CI configs — this is a content repo, not code.
- Don't restructure the top-level sections; external docs, the Supervisely UI, and `developer.supervisely.com` URLs are linked to these paths.
- Don't remove the GitBook-specific syntax tokens when "cleaning up" Markdown — they render as rich blocks on the live site.
