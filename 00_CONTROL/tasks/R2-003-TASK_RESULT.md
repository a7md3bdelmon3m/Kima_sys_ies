# TASK RESULT

Task ID: R2-003
Status: PARTIAL — logic/regression-verified via Node (real bundled SheetJS
+ a contract-matched mock), NOT real-IndexedDB or real-browser-verified
(same standing sandbox limitation as every task since R3-002 — no
Chromium binary, no npm registry access, no general network egress,
reconfirmed directly before this task)

## Objective (recap)

Add a relationship-import pipeline stage, parallel to the existing
entity-import pipeline, to cover the real K103 workbook's
`09_Piping_Connections` (144 rows) and `10_Relationships` (35 rows)
sheets — relationship-shaped data (`From_Tag`/`To_Tag`,
`Relationship_Type`) that ADR-013 identified as having no import path.

## Pre-implementation findings that shaped the design (see
R2-003-TASK_CONTRACT.md for the full reasoning)

1. Entities imported from K103 have `canonical_name` populated (matched
   from the `Tag_ID` column per ADR-013) but, in the common case, no
   `entity_identifier` row — so `findEntityIdentifier` cannot resolve
   `From_Tag`/`To_Tag` values against them. A new lookup was required.
2. `entity_type`+`canonical_name` has no uniqueness constraint anywhere
   in this schema (`entityRecord`/`createEntity` perform no such check).
   A relationship endpoint resolving to 2+ entities is therefore a real,
   reachable case, not a theoretical one, and had to be designed for
   explicitly from the start rather than discovered later as a defect.

## What was built

- `db.js#findEntitiesByCanonicalName(entityType, canonicalName)` — new,
  additive. Exact match only (never substring/fuzzy). Uses the existing
  `ix_entity_type_name` composite index when `entityType` is given;
  falls back to a full `listEntities()` + filter when it is not (real
  relationship sheets don't always state each endpoint's entity_type
  inline). Always returns an array — ambiguity is the caller's problem
  to flag, never resolved here to a single guess.
- `db.js#commitImportRelationshipRow({row, source_id})` — new. Creates
  an `evidence` record first (unattached — the relationship doesn't
  exist yet to attach it to), then calls the existing
  `createRelationship` with that `evidence_id`. Two separate
  transactions, deliberately matching this project's existing per-row
  commit pattern (`commitImport` already isolates each row's write so
  one failure never aborts siblings) rather than introducing a new,
  stronger atomicity guarantee this codebase doesn't use elsewhere.
- `import.js` — a fully parallel stage: `RELATIONSHIP_KNOWN_HEADERS`
  (separate alias map from entity `KNOWN_HEADERS`), `recognizeRelationshipHeaders`,
  `buildRelationshipImportPlan`, `previewRelationshipImport`,
  `commitRelationshipImport`, exported as `RelationshipImportPipeline`.
  The existing entity pipeline (`buildImportPlan`/`commitImport`/
  `KNOWN_HEADERS`/`recognizeHeaders`) is completely untouched — confirmed
  by diff, not just by intent.
- `app.js`/`index.html` — an import-mode switch (كيانات / علاقات) above
  the existing import panel, with a fully separate second file input,
  preview host, and commit button, wired to fully separate state
  variables (`currentRelPreview`/`currentRelFile` vs the existing
  `currentPreview`/`currentFile`) so switching modes can never clobber
  an in-progress preview in the other mode.

## Design decisions worth recording

- **relationship_type/direction validity is checked at preview time**,
  against the real `RELATIONSHIP_TYPES`/`RELATIONSHIP_RULES` imported
  from `db.js` (not re-declared) — so a BLOCKER shown in the preview is
  guaranteed to match what `createRelationship` would have rejected at
  commit, never a narrower or looser check.
- **Direction defaults to `'forward'` when the column is missing or its
  value isn't recognized** — the same default `assertRelationshipSemantics`
  itself applies — rather than guessing at new direction-vocabulary
  aliases. This was a deliberate choice informed by M-19 (a mismatched
  direction vocabulary — `backward` vs `reverse` — was a real, live
  defect elsewhere in this project): guessing aliases for unfamiliar
  workbook vocabulary is exactly the pattern that produced M-19.
- **Exact-duplicate rows within one file are flagged INFO, not
  BLOCKER**, and every occurrence of a duplicated tuple is flagged (not
  just the second-and-later ones) — mirroring `buildImportPlan`'s
  existing duplicate-identifier handling exactly. Reasoning: an exact
  repeated relationship statement across sheets in a real DEXPI/CFIHOS
  workbook (e.g. the same physical connection listed once on a P&ID
  sheet and again on a piping-connections sheet) is plausible real-world
  data, not obviously an error — INFO surfaces it for awareness without
  blocking an otherwise-valid row.

## Verification performed (executed, not just read)

Built a realistic `.xlsx` fixture using this project's own bundled
SheetJS library (`30_WEB_APP/lib/xlsx.full.min.js`, the same one
`import.js` uses in production) — not a synthetic row array — with 6
data rows deliberately covering: a valid unambiguous row, an ambiguous
endpoint (ties to the central risk from finding #2 above), a
nonexistent endpoint, an invalid `relationship_type`, a type-valid-but-
direction-invalid combination, and an exact duplicate of the valid row.

Ran the real pipeline against this file through a byte-for-byte
identical copy of `import.js` (verified via `diff`: the only difference
is the `db.js` import path, swapped for a test-only mock implementing
the same function contracts — signatures, return shapes, and
`RELATIONSHIP_TYPES`/`RELATIONSHIP_RULES` values copied verbatim from
the real file). 18/18 assertions passed:

- Header auto-recognition of `From_Tag`/`To_Tag`/`Relationship_Type`/`Direction`.
- The ambiguous-canonical_name row correctly BLOCKER, not silently
  resolved to one of the two matches.
- The nonexistent-endpoint row correctly BLOCKER.
- The invalid-`relationship_type` row correctly BLOCKER.
- The valid-type-invalid-direction row (`containment`+`reverse`)
  correctly BLOCKER, matching `RELATIONSHIP_RULES` exactly.
- The duplicate row correctly INFO on both occurrences, not BLOCKER.
- Commit correctly persisted exactly the 2 BLOCKER-free rows and
  skipped the other 4, each committed relationship paired with its own
  evidence record (provenance invariant maintained).

One test-authoring error was caught and corrected during this process:
an initial assertion wrongly expected the first of two duplicate rows
to have zero conflicts, contradicting the code's own deliberate
"flag every occurrence of a duplicated tuple" design (mirroring
`buildImportPlan`'s existing behavior) — the test's expectation was
fixed, not the code, once the discrepancy was traced to its source.

`node --check` clean on every file in `30_WEB_APP` after all edits.
`decision.test.mjs` re-run: PASS (unaffected by this task, run as a
general regression check). Every `KimaDB.*` call site across
`app.js`/`decision.js`/`impact.js`/`import.js` cross-checked against
`db.js`'s actual exports: zero dangling calls introduced.

## Not verified

- Real IndexedDB transactions (`findEntitiesByCanonicalName`'s
  `ix_entity_type_name` index query, `commitImportRelationshipRow`'s
  two-transaction sequence) — the mock's index-query simulation matches
  IndexedDB's documented compound-index semantics but was not run
  against a real IndexedDB engine.
- Real browser DOM: the import-mode radio switch, the new relationship
  panel's rendering, and file-input change events were written and
  traced by reading, not executed in a browser.
- Real `.xlsx` files with more exotic structure (merged cells, multiple
  relevant sheets, `From_ID`/`To_ID` numeric-style columns rather than
  tag strings) beyond what the test fixture covered.

## Recommendation

Real-browser verification remains the single outstanding gate, same as
every task since R3-002. When available, this task's UI (mode switch,
both file inputs, both preview panels) should be included in the same
combined verification pass already recommended in
R9-MERGE-001-TASK_RESULT.md's own recommendation section, rather than
opening yet another separate verification round.

## State changes

Per `SYSTEM_OF_RECORD_MATRIX.md`'s precedence rule: that file updated
first (M-16's row rewritten with full verification detail), then
`V3_GAP_REGISTER.txt` (M-16's STATE column: OPEN -> FIXED, with the
ambiguous-match risk explicitly recorded as an addressed finding, not
just the originally-logged gap), then `PROJECT_STATE.md` (this entry).
