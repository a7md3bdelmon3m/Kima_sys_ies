# TASK CONTRACT

Task ID: R1-002
Phase: R1
Title: Relationship + Evidence CRUD, wired into the ledger panel

## Objective

Extend the existing `30_WEB_APP` (built and PASS-verified in R1-001) so
that the ledger panel for a selected entity shows real, persisted
`relationship` and `evidence` records — not just identifiers — with
`validation_state` and `evidence_id` linkage working end to end, per
`03_DOMAIN/RELATIONSHIP_CONTRACT.md` and the `relationship`/`evidence`
stores already defined (schema-only, no UI/CRUD yet) in
`04_DATA/INDEXEDDB_SCHEMA_R0.md` and `30_WEB_APP/db.js`.

## Inputs

- `00_CONTROL/MASTER_PROJECT_CONTRACT.md` (v3.0, 14 principles)
- `00_CONTROL/ACR/ACR-001-web-platform-shift.md`
- `00_CONTROL/tasks/R1-001-TASK_RESULT.md` (baseline this task builds on)
- `14_LLM_OS/roles/web_engineer.md`, `14_LLM_OS/roles/data_engineer.md`
- `03_DOMAIN/RELATIONSHIP_CONTRACT.md`, `03_DOMAIN/CANONICAL_ENTITY_CONTRACT.md`
- `04_DATA/INDEXEDDB_SCHEMA_R0.md` (`relationship`, `evidence`, `source`
  stores + referential integrity policy)
- Existing code: `30_WEB_APP/db.js`, `app.js`, `index.html`, `styles.css`

## Allowed files

Only inside `30_WEB_APP/`:
- `db.js` (extend — add relationship/evidence/source functions; do not
  touch `openDb`'s store/index definitions, they are already correct
  and match `INDEXEDDB_SCHEMA_R0.md` exactly)
- `app.js` (extend the ledger panel rendering + wire two new forms)
- `index.html` (no structural change expected — ledger panel is already
  a single mount point rendered from `app.js`; touch only if a new
  top-level section is genuinely needed)
- `styles.css` (extend — add `<select>` styling and any new block
  needed for relationship/evidence lists, reusing existing
  chip/badge/inline-form/field-row patterns, no new visual language)
- `README.md` (update feature list if user-facing behavior changes)

Nothing outside `30_WEB_APP/` is touched by IMPLEMENT. Governance files
(`PROJECT_STATE.md`, `DECISION_LOG.md`, this task's own contract/result)
are updated only in the STATE UPDATE step, per protocol.

## Constraints

1. `entity_id` referenced by a new `relationship` or `evidence` record
   must be verified to exist in the same IndexedDB transaction before
   the write commits (Referential integrity policy,
   `INDEXEDDB_SCHEMA_R0.md`) — mirrors the pattern already used in
   `addEntityIdentifier`.
2. Relationship `relationship_type` restricted to the 12 categories in
   `RELATIONSHIP_CONTRACT.md`: containment, connection, dependency,
   reference, process_flow, piping, instrumentation, function,
   equipment_association, document_reference, drawing_reference,
   standard_mapping. Enforced via a `<select>` (not free text) so an
   invalid category cannot be submitted from the UI.
3. `validation_state` defaults to `'UNREVIEWED'` for both `relationship`
   and `evidence`, per schema.
4. No relationship edge is ever presented as an implied physical
   connection beyond what `relationship_type` states (topology rule,
   `06_GRAPH/`) — this task only lists/creates relationships, it does
   not yet render a graph, so this is a labeling discipline (the UI
   must show the type as-is, not paraphrase or imply more).
5. No fuzzy-matching entity merge logic introduced anywhere — target
   entity selection for a relationship is by explicit entity_id (a
   `<select>` sourced from `KimaDB.listEntities()`), never free text
   matched against names.
6. `evidence` requires a parent `source` record. Since no import
   pipeline exists yet (R2, not built), this task adds a minimal
   manual "register a source" affordance sufficient to satisfy the
   `evidence.source_id` foreign-key-equivalent honestly (a real
   `source` row with `checksum_sha256` computed from user-entered
   locator text, not a fabricated placeholder) — this is scoped
   narrowly and documented as a stand-in for the future R2 import
   pipeline, not a redesign of it.
7. No build toolchain introduced (still vanilla ES modules, no
   bundler/transpiler/npm dependency).
8. No IndexedDB schema change (no new store, no new index, no field
   rename) — `db.js`'s existing `openDb` already defines everything
   this task needs. If a genuine gap is found during implementation,
   stop and report it rather than silently extending the schema.
9. Existing R1-001 functionality (`entity`/`entity_identifier` CRUD)
   must keep working exactly as before — verified by re-running the
   original Suite A/B checks, not just the new ones.

## Acceptance tests

Real functional tests against an actual Chromium browser (headless via
Puppeteer/CDP, same method as R1-001), covering:
- Create a relationship between two existing entities → appears in both
  entities' ledger panels (or at least the source entity's, documented
  either way).
- Attempt to create a relationship referencing a non-existent entity_id
  → rejected with a clear, non-crashing error (mirrors T006 intent).
- Register a source, then create evidence referencing it, then link
  that evidence to a relationship → evidence appears with correct
  `source_id` linkage (mirrors T007 intent).
- Attempt to create evidence referencing a non-existent source_id →
  rejected with a clear, non-crashing error.
- `validation_state` defaults correctly and is visible in the UI for
  both relationship and evidence records.
- Re-run of R1-001's original Suite A (A1–A8) and Suite B (F1–F3) to
  confirm zero regression.

## Forbidden changes

- No edits to `21_ANDROID_STARTER/` (historical, out of scope).
- No edits to `15_BUILD/*`, role files, or any `00_CONTROL/` governance
  file during IMPLEMENT (only during STATE UPDATE, and only
  `PROJECT_STATE.md` + this task's own records).
- No redefinition of `RELATIONSHIP_CONTRACT.md` categories or fields.
- No silent IndexedDB schema/version bump.

## Deliverables

- Modified `30_WEB_APP/db.js`, `app.js`, `index.html` (if needed),
  `styles.css`, `README.md`.
- `00_CONTROL/tasks/R1-002-TASK_CONTRACT.md` (this file).
- `00_CONTROL/tasks/R1-002-TASK_RESULT.md` with real test evidence.
- `00_CONTROL/PROJECT_STATE.md` updated (Next task advanced).

## Dependencies

R1-001 (Status: PASS) — this task builds directly on its `db.js`/`app.js`
code and must not regress it.
