# TASK CONTRACT

Task ID: R3-001
Phase: R3
Title: Explorer + Search — persistent Plant/Area/Unit/System context, exact/prefix/normalized search, filter, sort, pagination

## Objective

Build the context browser and search layer inside the existing
`30_WEB_APP` per `01_PRODUCT/MOBILE_UX_REQUIREMENTS.md` and
`13_UI_UX/DESIGN_SYSTEM.md`: a persistent hierarchical context bar
(Plant/Area/Unit/System), and exact/prefix/normalized search with
type/status filters, sort, and pagination — replacing the current
unfiltered, unpaginated flat entity list in `app.js` with a real
explorer, without touching the domain contracts or `db.js`'s existing
write paths.

## Inputs

- `00_CONTROL/MASTER_PROJECT_CONTRACT.md` (v3.0, 14 principles)
- `00_CONTROL/ACR/ACR-001-web-platform-shift.md`
- `00_CONTROL/PROJECT_STATE.md` (confirms R2-001 + R2-002 both PASS)
- `01_PRODUCT/MOBILE_UX_REQUIREMENTS.md`, `13_UI_UX/DESIGN_SYSTEM.md`
- `03_DOMAIN/CANONICAL_ENTITY_CONTRACT.md` (matching rule: exact/
  explicit-mapping/normalized/reviewed-equivalence only, never fuzzy)
- `14_LLM_OS/roles/web_engineer.md`
- Existing code: `30_WEB_APP/db.js`, `app.js`, `index.html`, `styles.css`

## Allowed files

Only inside `30_WEB_APP/`:
- `db.js` (extend only — add a `searchEntities` function; no existing
  function signature changes, no schema/index change)
- `app.js` (extend — replace the flat unfiltered list render with a
  search/filter/sort/paginate-aware explorer; existing `selectEntity`
  ledger-panel logic, existing entity/relationship/evidence/source/
  import forms all remain untouched and fully working)
- `index.html` (extend — add explorer controls: search input, context
  bar, type/status filter selects, sort select, pagination controls)
- `styles.css` (extend, reusing existing tokens/patterns)
- `README.md` (update feature list)

## Constraints

1. **A pre-existing schema gap must be named, not silently worked
   around:** `CANONICAL_ENTITY_CONTRACT.md` lists a single free-text
   `context` field on `entity` — there is no separate structured
   `plant`/`area`/`unit`/`system` field in `04_DATA/INDEXEDDB_SCHEMA_R0.md`,
   and adding one would be a schema change requiring an ACR, which is
   out of this task's scope. This task therefore implements the
   persistent context bar as an **optional, best-effort parse** of the
   existing `context` string using a documented separator convention
   (`"Plant / Area / Unit / System"`, matching the UX requirement
   doc's own hierarchy notation) — an entity whose `context` doesn't
   follow this convention still displays and searches normally, just
   without a parsed hierarchy breadcrumb. This is reported explicitly
   in TASK_RESULT as a scoping decision, not hidden as if a real
   structured field existed.
2. **No duplicated matching/search logic.** `searchEntities` lives in
   `db.js` — `app.js` calls it, never re-implements filtering inline
   against `state.entities` by hand. (`state.entities` remains the
   in-memory cache `db.js`'s `listEntities()` already populates;
   `searchEntities` operates over that cache to avoid a redundant
   IndexedDB round-trip per keystroke, but the actual match/filter/sort
   predicate logic is centralized in `db.js`, not re-typed in `app.js`.)
3. **Search matching stays within `CANONICAL_ENTITY_CONTRACT.md`'s
   allowed matching rule** — this is *search* (finding candidates for
   a human to look at), not *identity resolution* (auto-merging
   records), so substring/prefix matching on display fields is fine
   and is NOT the fuzzy-merge the contract forbids; that prohibition
   is specifically about entity-identity matching during import, not
   this task. This distinction is stated explicitly in TASK_RESULT so
   it isn't misread as a contract violation later.
4. Engineering identifiers remain visually dominant and selectable/
   copyable plain text — no canvas-rendered text, no truncation that
   loses copyability (ellipsis via CSS `text-overflow` is fine, the
   underlying text node must remain intact).
5. RTL layout mandatory throughout new markup (`dir="rtl"` inherited
   from `<html>`, logical CSS properties, no hardcoded `left`/`right`),
   with engineering identifiers kept LTR within RTL context (same
   pattern already verified in R1-001's test A7, extended to new
   screens here).
6. The UI never invents missing data — an entity with no parsed
   context hierarchy shows "context not structured" or equivalent, not
   a guessed breadcrumb.
7. Pagination and result counts must reflect the actual filtered set,
   not the full unfiltered entity count.
8. No IndexedDB schema/version change. No build toolchain introduced.
9. Existing R1-001/R1-002/R2-001/R2-002 functionality must keep working
   unmodified — entity creation, relationship/evidence CRUD, CSV/xlsx
   import, ledger panel — this task only changes how the entity list
   above the ledger panel is browsed/filtered.

## Acceptance tests

Real functional tests against an actual Chromium browser (same
dependency-free raw-CDP harness pattern, network monitoring included
per R2-002's precedent since this task also touches how data loads):

- Exact search: typing a full `canonical_name` returns exactly that
  entity.
- Prefix search: typing the first few characters of a `display_name`
  returns all entities whose display name starts with (or contains,
  documented precisely in TASK_RESULT based on actual implementation)
  that substring.
- Normalized search: search is case-insensitive and
  whitespace-trimmed (e.g. searching `"  pump-101  "` matches
  `PUMP-101`).
- Type filter: selecting an `entity_type` shows only matching entities.
- Status filter: selecting a `status` shows only matching entities.
- Combined filter + search: both apply together (AND, not OR).
- Sort: at least by `updated_at` (existing default) and by
  `canonical_name`, both directions.
- Pagination: with more entities than one page's worth, page controls
  correctly slice the filtered/sorted set with no duplicate or missing
  entities across pages.
- Empty result state: a query matching nothing shows a clear
  "no results" state, not a blank/broken screen.
- Context hierarchy: an entity with `context` following the
  `"Plant / Area / Unit / System"` convention displays a parsed
  breadcrumb; an entity without it displays cleanly with no invented
  values.
- Zero regression: full re-run of all prior real-browser suites
  (R1-001 11, R1-002 16 extra, R2-001 17, R2-002 14, plus their
  `file://` counterparts — 67 total from R2-002-TASK_RESULT.md) still
  pass unmodified.
- Zero new CDN/network dependency introduced by this task (verified by
  the same network-monitoring harness from R2-002).

## Forbidden changes

- No edits to `03_DOMAIN/*.md`, `04_DATA/INDEXEDDB_SCHEMA_R0.md`, or any
  other domain/schema contract.
- No new IndexedDB object store or index.
- No edits to `21_ANDROID_STARTER/`, `15_BUILD/*`, role files, or other
  `00_CONTROL/` governance files during IMPLEMENT.
- No fuzzy auto-merge logic introduced anywhere near identity
  resolution (search filtering is explicitly out of that rule's scope,
  per Constraint 3, but nothing here may be wired into
  `createRelationship`'s or the import pipeline's identity-matching
  paths).

## Deliverables

- Modified `30_WEB_APP/db.js`, `app.js`, `index.html`, `styles.css`,
  `README.md`.
- `00_CONTROL/tasks/R3-001-TASK_CONTRACT.md` (this file).
- `00_CONTROL/tasks/R3-001-TASK_RESULT.md` with real test evidence.
- `00_CONTROL/tasks/R3-001-test-harness/` (reusing R2-002's `cdp.js`).
- `00_CONTROL/PROJECT_STATE.md` updated.
- `00_CONTROL/FILE_MANIFEST.md` regenerated.

## Dependencies

R1-001, R1-002, R2-001, R2-002 (all PASS) — extends `app.js`'s entity
list rendering and `db.js` with a new read-only search function; does
not modify any existing write path.
