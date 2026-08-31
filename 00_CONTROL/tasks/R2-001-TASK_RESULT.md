# TASK RESULT

Task ID: R2-001
Status: PASS

## Files created

- `30_WEB_APP/import.js` — the full import pipeline
  (fingerprint → parse → structural validation → header/schema
  recognition → normalization → candidate identity resolution →
  relationship extraction (stage present, no-op for this CSV schema) →
  evidence attachment (stage folded into commit's `source` recording) →
  conflict report → preview → commit).
- `00_CONTROL/tasks/R2-001-TASK_CONTRACT.md`
- `00_CONTROL/tasks/R2-001-TASK_RESULT.md` (this file)
- `00_CONTROL/tasks/R2-001-test-harness/` (cdp.js copy, both test
  scripts, three CSV fixtures, README)

## Files modified

- `30_WEB_APP/db.js` — added `sha256HexBytes` (hash raw file bytes,
  distinct from the existing text-hashing `sha256Hex`), `getEntity`
  (was already present from R1-002, unchanged), `updateEntity` (new —
  the idempotent re-import "update path" needs to mutate an existing
  entity's fields without changing its `entity_id`/`created_at`), and
  `findEntityIdentifier` (new — exact `[namespace, identifier]` lookup
  against the existing unique index, used by candidate identity
  resolution). `openDb`'s object stores/indexes are completely
  untouched — zero schema/version change.
- `30_WEB_APP/app.js` — added an Import section: file input handler,
  preview renderer (fingerprint, unmapped-columns report, per-row
  insert/update classification, conflict list with severity badges),
  and a commit handler. Existing entity/relationship/evidence/source
  manual-form logic is unchanged.
- `30_WEB_APP/index.html` — added the Import section markup (file
  input, result container, commit button) above the existing entity
  list/ledger panel sections.
- `30_WEB_APP/styles.css` — added `input[type="file"]` styling and a
  small amount of layout CSS for the import section, reusing existing
  chip/badge tokens for conflict severity (no new visual language).

## Tests executed

Same dependency-free raw-CDP harness pattern established in R1-002
(Chrome 131.0.6778.204, headless, driven via Node 22's built-in
`WebSocket`; Puppeteer remains unavailable in this sandbox — no network,
no cached package).

**New R2-001 suite — hosted mode** (17 checks, covering
`16_TESTING/TEST_MATRIX.md` T001–T003 plus supporting acceptance
criteria from the task contract):

| ID | Check | Result |
|---|---|---|
| R2-0-load | Page loads, zero console errors | PASS |
| R2-0-baseline | Zero entities before any import (clean starting state) | PASS |
| T001-fingerprint-stable | SHA-256 over identical file bytes is stable across two separate `File` instances | PASS |
| T001-fingerprint-differs | A byte-different file produces a different fingerprint | PASS |
| R2-preview-shape | Preview of a clean 3-row CSV yields 3 planned rows, 0 conflicts, 0 unmapped columns | PASS |
| R2-preview-no-mutation | Entity/source store counts identical before and after preview (zero writes) | PASS |
| T002-deterministic-import | Committing the 3-row CSV creates exactly 3 entities, 0 skipped | PASS |
| T002-fields-and-provenance | Committed entity has all 8 `CANONICAL_ENTITY_CONTRACT.md` fields; original `context`/`display_name` values preserved verbatim; identifier recorded correctly | PASS |
| T003-idempotent-reimport-no-dup-entities | Re-importing the identical file a second time: entity count stays at 3 (not 6) | PASS |
| T003-idempotent-reimport-no-dup-identifiers | Identifier count stays at 3 after re-import (no duplicate `entity_identifier` rows) | PASS |
| T003-idempotent-reimport-update-path | All 3 re-imported rows resolve via the `update` path (matched existing entity), not `insert` | PASS |
| R2-conflict-blocker-detected | A file with two rows sharing one identifier is classified BLOCKER | PASS |
| R2-conflict-blocker-not-committed | Neither of the two BLOCKER-conflicting rows is committed (entity count unchanged) | PASS *(after fix — see Decisions)* |
| R2-row-position-independence | Reordering an already-imported file's rows still resolves all 3 via `update`, entity count stays 3 | PASS |
| R2-ui-elements-present | File input, result container, and commit button exist; commit button starts hidden | PASS |
| R2-regression-manual-create-still-works | The pre-existing manual entity-creation form still works after adding Import (4 entities: 3 imported + 1 manual) | PASS |
| R2-no-console-errors-overall | Zero console errors/exceptions across the entire run | PASS |

17/17 passed.

**New R2-001 suite — raw `file://` mode** (4 checks, no server):

| ID | Check | Result |
|---|---|---|
| F-R2-1-load | `file://` load, zero console errors | PASS |
| F-R2-2-import-under-file | Full preview→commit cycle works under `file://` with an in-memory `File` object (2 rows, 0 conflicts, 2 committed) | PASS |
| F-R2-3-persists-after-reload | Imported data survives a full page reload under `file://` | PASS |
| F-R2-4-no-console-errors | Zero console errors under `file://` | PASS |

4/4 passed.

**Regression re-run — R1-001/R1-002's original suites, against the
R2-updated app** (confirms zero regression from adding the import
pipeline):
- Hosted suite (A1–A8, R002-1–R002-13): **22/22 passed**
- `file://` suite (F1–F5): **5/5 passed**

**Total this task: 48/48 real browser checks passed** (17 + 4 new,
22 + 5 regression).

## Test evidence

Raw output, new R2-001 hosted suite (port 9226, app served on 8900,
after the fix described in Decisions):

```
PASS — R2-0-load: console errors=0 []
PASS — R2-0-baseline: entities before any import=0
PASS — T001-fingerprint-stable: {"fp1":"ebdd11aabc51cee0f82a139fc189ee0bc29ad1c749e37f669ce274ab50cec299","fp1b":"ebdd11aabc51cee0f82a139fc189ee0bc29ad1c749e37f669ce274ab50cec299","fp2":"ece5c5f2605fd915f29679fc1fab1d526cb21c6d8f91014bc590efc5fcfde1a5","sameFileStable":true,"differentFileDiffers":true,"fp1Length":64}
PASS — T001-fingerprint-differs: fp1=ebdd11aabc51 fp2=ece5c5f2605f
PASS — R2-preview-shape: {"checksum":"ebdd11aabc51...","rowCount":3,"conflictCount":0,"unmapped":0}
PASS — R2-preview-no-mutation: before={"entities":0,"sources":0} after={"entities":0,"sources":0}
PASS — T002-deterministic-import: {"committedCount":3,"skippedCount":0,"totalEntities":3}
PASS — T002-fields-and-provenance: {"hasAllFields":true,"contextPreserved":true,"displayNamePreserved":true,"identifierRecorded":true}
PASS — T003-idempotent-reimport-no-dup-entities: entities after 2nd import=3
PASS — T003-idempotent-reimport-no-dup-identifiers: identifiers after 2nd import=3
PASS — T003-idempotent-reimport-update-path: actions=["update","update","update"]
PASS — R2-conflict-blocker-detected: ["BLOCKER","BLOCKER"]
PASS — R2-conflict-blocker-not-committed: before=3 after=3
PASS — R2-row-position-independence: {"actions":["update","update","update"],"totalEntities":3}
PASS — R2-ui-elements-present: {"fileInput":true,"resultHost":true,"commitBtn":true,"commitBtnHiddenInitially":true}
PASS — R2-regression-manual-create-still-works: entities after manual create=4 (expected 4: 3 imported + 1 manual)
PASS — R2-no-console-errors-overall: errors=0 []

=== SUMMARY ===
17/17 passed
ALL TESTS PASSED
```

Raw output, new R2-001 file:// suite (port 9227):
```
PASS — F-R2-1-load: console errors=0 []
PASS — F-R2-2-import-under-file: {"rowCount":2,"conflictCount":0,"committedCount":2,"totalEntities":2}
PASS — F-R2-3-persists-after-reload: entities after reload=2
PASS — F-R2-4-no-console-errors: errors=0

=== SUMMARY (file://, R2) ===
4/4 passed
ALL FILE:// R2 TESTS PASSED
```

Raw output, regression re-run of R1-001/R1-002 suites (ports 9228/9229,
app served on 8901, from a clean `30_WEB_APP` copy without test
fixtures — i.e. the actual delivery state):
```
[hosted: 22/22 passed, ALL TESTS PASSED]
[file://: 5/5 passed, ALL FILE:// TESTS PASSED]
```
(Full per-check lines identical in form to R1-002-TASK_RESULT.md's
originals; omitted here for brevity since every line individually
re-printed PASS — reproducible via
`00_CONTROL/tasks/R1-002-test-harness/`.)

Syntax/format validation (re-run, unchanged method):
```
$ node --check app.js && node --check db.js && node --check import.js \
    && node --check service-worker.js && echo OK
OK
$ python3 -m json.tool manifest.webmanifest > /dev/null && echo OK
OK
```

## Decisions

- **CSV instead of Excel for R2-001 (explicit scoping decision, not a
  silent limitation):** no `.xlsx`-parsing library is available in this
  sandbox and none can be fetched (no network egress); introducing an
  unvetted dependency without a build toolchain would violate the
  project's established "no build toolchain by default" posture. The R2
  phase prompt itself names manual CSV parsing as the explicit fallback
  for exactly this situation. `import.js`'s `parseCsv` is a
  dependency-free RFC-4180-style parser (quoted fields, embedded commas/
  newlines/escaped quotes). `.xlsx` support is deferred to a follow-up
  task (R2-002) once a vetted, locally-vendored parser is available —
  tracked in Unresolved issues, not silently dropped.
- **A real bug was found and fixed by the test suite, not silently
  worked around:** the first implementation of duplicate-identifier
  detection only flagged the *second and later* occurrences of a
  repeated `[namespace, identifier]` key within one file as BLOCKER,
  silently letting the *first* occurrence commit. Test `R2-conflict-
  blocker-not-committed` caught this (entity count went 3→4 instead of
  staying at 3). On inspection, silently accepting the first occurrence
  is itself an unreviewed identity decision — there is no objective way
  to know which of two rows sharing one identifier is "the real one"
  without human review — which conflicts with
  `MASTER_PROJECT_CONTRACT.md`'s matching-rule principle (no merge
  without exact match / explicit mapping / reviewed equivalence).
  Fixed by a two-pass approach in `buildImportPlan`: first count
  occurrences of each identity key across the whole file, then flag
  **every** row sharing a key that occurs more than once, not just the
  later ones. Re-ran the full 17-test suite from a clean state after
  the fix; all 17 passed, including the corrected check. This sequence
  (bug found by real test → fixed → full re-verification) is reported
  in full rather than only showing the final green run, per
  `MASTER_PROJECT_CONTRACT.md`'s "no release without objective evidence"
  and the anti-hallucination protocol's TEST-VERIFIED standard.
- **Idempotency implementation:** re-import identity resolution is keyed
  strictly off the existing `ux_entity_identifier` unique compound index
  (`[namespace, identifier]`), via the new `findEntityIdentifier`
  exact-lookup helper — never off `canonical_name`, row position, or any
  fuzzy comparison. A matched row takes the `update` path
  (`KimaDB.updateEntity`, preserving `entity_id`/`created_at`); an
  unmatched row takes the `insert` path
  (`KimaDB.createEntity` + `addEntityIdentifier`). A `source` row is
  still recorded on every commit attempt (import provenance/audit trail
  of "when was this file last imported"), even when zero new entities
  result — this is intentional per `IMPORT_PIPELINE.md`'s "every
  committed source-derived record retains provenance" invariant applied
  to the act of importing itself, not only to the resulting entities.
- **Relationship extraction stage is present but a documented no-op:**
  this CSV schema (`entity_type, canonical_name, display_name, context,
  namespace, identifier`) does not express relationships between rows.
  Rather than silently skip the stage, `buildImportPlan`'s returned plan
  includes a `relationshipExtraction` object naming the stage and
  stating explicitly why nothing was extracted, so a future schema
  revision (or reviewer) can see the stage was considered, not
  forgotten.
- **Structural validation vs. conflict report as separate concerns:**
  ragged rows (wrong column count) are caught at the structural-
  validation stage (MAJOR, row excluded from the plan entirely) rather
  than flowing into candidate identity resolution and being reported
  there — keeping the stage boundaries from `IMPORT_PIPELINE.md`
  meaningful rather than collapsing everything into one generic
  conflict list.

## Warnings

- `.xlsx` (actual Excel binary format) is not supported yet — only CSV.
  This is a real, current functional gap for anyone whose source files
  are `.xlsx`, not merely a documentation note.
- The header-recognition map (`KNOWN_HEADERS`) is a fixed, small set of
  English/Arabic aliases for six fields. A source file using column
  names outside this set will have those columns reported as "unmapped"
  (INFO-level, visible in the preview) but not imported into any field —
  this is safe (no silent data loss, no silent misassignment) but means
  real-world workbooks will likely need either a header-mapping UI
  (not built in this task) or conforming their headers to the known
  aliases.
- No relationship data is imported from CSV yet (see Decisions,
  relationship extraction no-op) — only entities and their identifiers.
- Large-file / large-row-count performance was not tested (out of scope
  for R2-001; belongs with T015/R9 per the roadmap).
- The import UI does not yet show a distinct "already up to date, no
  changes" state distinctly from "N rows will update" when a re-import
  changes nothing substantive — it currently always reports rows as
  `update` if matched, without diffing whether the update is a no-op.
  Not a correctness issue (an identical no-op `put` is harmless) but a
  minor UX clarity gap worth revisiting.

## Unresolved issues

- (Carried over, unchanged) `file://` behavior verified only on
  Chromium; Firefox for Android / Samsung Internet still unverified.
- (Carried over, unchanged) No backup/export mechanism exists yet.
- (Carried over, unchanged) No in-repo/on-phone test runner; harness
  remains external-sandbox-only, now with two accumulated harness
  directories (`R1-002-test-harness/`, `R2-001-test-harness/`) sharing
  the same `cdp.js` — worth consolidating into one shared harness
  location in a future housekeeping task rather than continuing to copy
  `cdp.js` per task.
- **New:** `.xlsx` support (see Warnings) — recommend a dedicated
  R2-002 task once a vetted, locally-vendorable pure-JS `.xlsx` parser
  can be sourced and reviewed (network access permitting, or supplied
  by the user directly as a vendored file).
- **New:** no header-mapping UI for source files whose column names
  don't match `KNOWN_HEADERS`' fixed alias list.
- `standard_mapping` and `change_event` stores still have no UI/CRUD
  (unchanged — in scope for R6 and R8 respectively).

## State changes

`PROJECT_STATE.md` to be updated (see next step in this session): phase
advances to R2, "Next task" set per Recommended next task below.

## Recommended next task

R2-001 delivers CSV-only import. Two reasonable next steps, either
acceptable:
1. **R2-002 — `.xlsx` support**, once a vetted local parser is
   available, extending `import.js`'s `parse` stage with a second
   format branch (fingerprint/validation/normalization/etc. stages are
   already format-agnostic and would not need to change).
2. **R3 — Explorer + Search**, per
   `KIMA_EIS_PHASE_PROMPTS_R1_R15.md`, building the context browser now
   that a real (if CSV-only) bulk-population path exists to populate
   test/demo data at scale beyond one-row-at-a-time manual entry.

No strict dependency forces one over the other; recommend R3 first
since it delivers more end-user-visible value sooner, with R2-002
picked up whenever an `.xlsx` parser becomes available.
