# TASK CONTRACT

Task ID: R1-001
Phase: R1 — Canonical Identity + Local Persistence Contract (web/IndexedDB track, per ACR-001)
Title: Bootstrap the HTML/CSS/JS application shell with a working IndexedDB persistence layer implementing the canonical entity/identifier model

## Objective

Produce a runnable, offline-capable, single-codebase web application that:
1. Opens directly as a local file (`file://`) with zero server and zero build step, AND
2. Is installable as a PWA when served over http(s) (manifest + service worker present and correct),
and that implements, with full referential-integrity enforcement in application code, the `entity` and `entity_identifier` object stores exactly as defined in `04_DATA/INDEXEDDB_SCHEMA_R0.md`, satisfying `03_DOMAIN/CANONICAL_ENTITY_CONTRACT.md` in full (including the resolved `context` field).

This is the smallest vertical slice that proves the platform shift (ACR-001) end-to-end: UI shell → data-access layer → IndexedDB → back to UI, with RTL Arabic support live from the first screen.

## Inputs

- `00_CONTROL/ACR/ACR-001-web-platform-shift.md` (approved)
- `03_DOMAIN/CANONICAL_ENTITY_CONTRACT.md`
- `04_DATA/INDEXEDDB_SCHEMA_R0.md`
- `04_DATA/SQLITE_SCHEMA_R0.sql` (reference only, for field fidelity)
- `MASTER_PROJECT_CONTRACT.md` Principles 1–13 (Arabic-first RTL, traceability, no silent invention, ACR gating, etc.)

## Allowed files

New directory: `30_WEB_APP/` (parallel to `21_ANDROID_STARTER/`, does not touch it)
- `30_WEB_APP/index.html`
- `30_WEB_APP/styles.css`
- `30_WEB_APP/app.js` (or ES module split: `app.js`, `db.js`, `ui.js`)
- `30_WEB_APP/manifest.webmanifest`
- `30_WEB_APP/service-worker.js`
- `30_WEB_APP/README.md`
- `00_CONTROL/PROJECT_STATE.md` (status update only)

## Constraints

- No build step required to run (`file://` must work with no `npm install`, no bundler).
- No external CDN dependency for core functionality — the file:// mode must work fully offline with zero network access (Principle: offline-first). Any font/icon may be embedded or system-default; no remote fetch on load.
- All entity/entity_identifier writes go through a single transaction per logical operation (per `INDEXEDDB_SCHEMA_R0.md` referential-integrity policy).
- UI default direction: `dir="rtl"`, `lang="ar"`; layout must not visually break if content is Latin-script.
- No fabricated sample "real-world" engineering data — seed/demo data, if any, must be clearly marked as demo, never presented as verified evidence-backed content (Principle: no silent invention / evidence traceability).
- Do not modify `21_ANDROID_STARTER/*` or any file outside `30_WEB_APP/` and the two control-log files listed above.

## Acceptance tests

1. Opening `30_WEB_APP/index.html` directly via `file://` in a mobile browser loads a functional UI with no console errors related to missing resources.
2. Creating an entity via the UI persists it in IndexedDB (`entity` store) with all contract fields (`entity_id`, `entity_type`, `canonical_name`, `display_name`, `status`, `context`, `created_at`, `updated_at`) populated or explicitly null where optional.
3. Adding two identifiers to the same entity, one marked primary, persists two rows in `entity_identifier`, both referencing the correct `entity_id`.
4. Attempting to insert a duplicate `[namespace, identifier]` pair is rejected by the unique index and surfaced as a user-visible, non-crashing error.
5. Reloading the page (closing and reopening the file) preserves all previously entered data (IndexedDB persistence survives reload).
6. `manifest.webmanifest` and `service-worker.js` are present, structurally valid (valid JSON manifest; service worker registers and caches the app shell), and are inert/non-blocking when the app is opened via plain `file://` (no dependency on them for core functionality in that mode).
7. Page renders correctly right-to-left with Arabic labels for all primary actions.

## Forbidden changes

- No changes to `03_DOMAIN/*`, `04_DATA/SQLITE_SCHEMA_R0.sql`, `04_DATA/LOGICAL_SCHEMA.md` (reference contracts stay stable; only `INDEXEDDB_SCHEMA_R0.md`, already written under ACR-001, governs the new store shapes).
- No introduction of a build toolchain (webpack/vite/etc.) at this stage — out of scope for R1; may be proposed later as a separate, non-architectural tooling ACR-lite if justified by codebase size.
- No native Android file changes.

## Deliverables

Per objective above: `30_WEB_APP/` directory (index.html, styles.css, app.js/db.js, manifest.webmanifest, service-worker.js, README.md), plus an updated `PROJECT_STATE.md` and a `TASK_RESULT` document for R1-001.

## Dependencies

ACR-001 (approved) and `04_DATA/INDEXEDDB_SCHEMA_R0.md` (written) — both satisfied prior to starting this task.
