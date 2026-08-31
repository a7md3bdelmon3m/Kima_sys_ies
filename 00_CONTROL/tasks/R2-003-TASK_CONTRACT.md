# TASK CONTRACT

Task ID: R2-003
Source: 17_ROADMAP/ROADMAP.md ("R2 — Import + Diff"), ADR-013 (recommended
this exact task, tentatively numbered R2-003), V3_GAP_REGISTER.txt M-16
(OPEN, P2), 03_DOMAIN/RELATIONSHIP_CONTRACT.md.

## Objective

Add a second, parallel import pipeline stage that reads relationship-
shaped spreadsheet data (`From_Tag`/`To_Tag` or `From_ID`/`To_ID` +
`Relationship_Type` columns, matching the real K103 workbook's
`09_Piping_Connections` (144 rows) and `10_Relationships` (35 rows)
sheets per ADR-013) and creates `relationship` records via the existing
`db.js#createRelationship`, without duplicating its validation logic.

## Pre-implementation findings (critical — read before reviewing the diff)

1. **No exact, unambiguous entity lookup by `canonical_name` exists in
   `db.js` today.** `findEntityIdentifier(namespace, identifier)` looks up
   the `entity_identifier` store's unique `[namespace, identifier]` index
   — but ADR-013's own fix mapped the real workbook's `Tag_ID` column to
   `canonical_name`, NOT to `namespace`/`identifier` (K103's entity sheets
   have no namespace/identifier columns at all). This means entities
   already imported from K103 have `canonical_name` populated but, in the
   common case, no corresponding `entity_identifier` row — so
   `findEntityIdentifier` cannot resolve `From_Tag`/`To_Tag` values against
   them. `searchEntities` (the only other lookup available) is a
   multi-result substring search — unsuitable and unsafe for resolving a
   single relationship endpoint (a substring match could silently attach
   a relationship to the wrong entity).
2. **`entity_type`+`canonical_name` is not enforced unique anywhere** —
   `db.js` has an index (`ix_entity_type_name`) on that pair but no
   function uses it, and `entityRecord`/`createEntity` perform no
   uniqueness check before insert. Two entities can legitimately (or by
   import error) share the same `canonical_name`. A relationship-import
   row whose `From_Tag`/`To_Tag` resolves to more than one entity is
   therefore a real, reachable case, not a theoretical one — and must be
   handled the same way `buildImportPlan` already handles duplicate
   identifiers within one file: flagged BLOCKER, not silently resolved to
   "the first match."
3. **Consequence for this task's design**: a new, narrowly-scoped lookup
   function is required in `db.js` — `findEntitiesByCanonicalName(entity_type
   | null, canonical_name)` — using the existing `ix_entity_type_name`
   index when `entity_type` is supplied (exact, fast) and falling back to
   a full `canonical_name`-only match via `listEntities()` + filter when
   it is not (the real workbook's relationship sheets, per ADR-013's own
   description, carry `From_Tag`/`To_Tag` values but do not necessarily
   state each endpoint's entity_type inline). This function returns an
   array (never assumes uniqueness) — the caller (this task's new plan
   builder) is responsible for treating a non-singleton result as a
   BLOCKER conflict, exactly like the existing duplicate-identifier
   handling in `import.js#buildImportPlan`.
4. **`createRelationship`'s existing constraints must not be
   re-implemented, only respected**: it already validates
   `relationship_type` membership in `RELATIONSHIP_TYPES`, direction
   against `RELATIONSHIP_RULES[type].allowedDirections`, requires both
   endpoints to exist, and rejects self-relationships except for
   `reference`. This task's plan-builder must pre-check these same
   conditions to produce accurate preview conflicts (so a user sees "row
   12: invalid direction" at preview time, not a silent skip at commit
   time), but the actual write path calls `createRelationship` itself as
   the single source of truth for the write — never a parallel
   `tx.objectStore('relationship').add(...)` in the new code.
5. **Direction value in the real data is not guaranteed to already be
   one of `forward`/`reverse`/`bidirectional`.** ADR-013 described the
   real column as `Relationship_Type`, not `Direction` — the real
   workbook may have no explicit direction column at all, or one using
   different vocabulary. Per M-19's own lesson (a mismatched direction
   vocabulary caused a real, live defect), this task must NOT assume a
   direction alias list will match the real file's vocabulary — it must
   default unmapped/unrecognized direction values to `'forward'` (the
   same default `assertRelationshipSemantics` itself already applies
   when `direction` is omitted), and surface the raw source value in the
   row's report either way, rather than guessing at new aliases the way
   M-19 shows can silently break.

## Constraints

- New pipeline stage only — `import.js`'s existing `buildImportPlan`/
  `commitImport`/`KNOWN_HEADERS` (entity import) are NOT modified. A
  second header-recognition map and a second plan-builder are added,
  parallel to the existing ones, selected by the caller (a new "import
  mode" choice in the UI: "كيانات" vs "علاقات"), per ADR-013's own
  guidance that this is a new pipeline stage, not a `KNOWN_HEADERS`
  extension.
- Reuse `db.js#createRelationship` for every write — no parallel
  relationship-creation logic (`RELATIONSHIP_CONTRACT.md`'s rule:
  relationships are canonical data with one write path).
- Every imported relationship must carry evidence, per this project's
  established import invariant (every `commitImportRow`-created entity
  gets an `observation`+`evidence` pair) — this task's
  `commitImportRelationshipRow` must do the equivalent: create an
  `evidence` record (`evidence_type: 'IMPORT_ROW'`, locator naming the
  source row) and pass its `evidence_id` into `createRelationship`,
  which already accepts one.
- Ambiguous endpoint resolution (0 or 2+ matching entities for a
  `From_Tag`/`To_Tag` value) is a BLOCKER conflict at preview time, never
  a best-guess pick.
- Must not regress the existing entity-import pipeline (R2-001/R2-002/
  ADR-012/ADR-013) — verified by re-running its existing Node-executable
  checks unchanged.

## Acceptance criteria

1. A relationship-shaped file (columns recognizable as endpoint-pair +
   relationship type, by a header-alias map analogous to but separate
   from `KNOWN_HEADERS`) can be previewed: each row resolves to zero, one,
   or many matching entities per endpoint, with zero/many flagged as a
   named BLOCKER conflict.
2. A row whose `relationship_type` is not in `RELATIONSHIP_TYPES`, or
   whose (explicit or defaulted) `direction` is not in that type's
   `allowedDirections`, is flagged as a BLOCKER at preview time with the
   same message shape `assertRelationshipSemantics` would throw, not
   discovered only at commit.
3. Committing a preview with zero BLOCKER rows creates one `relationship`
   record per row via `createRelationship` (never a parallel write path),
   each with its own `evidence` record referencing the source row.
4. Rows with any BLOCKER conflict are skipped at commit (same skip/report
   shape as the existing entity pipeline's `commitImport`), never
   partially written.
5. Re-importing the same relationship file a second time does not create
   duplicate relationships silently — at minimum, exact duplicate
   (source, target, type, direction) rows are flagged as a conflict at
   preview time (INFO or BLOCKER, to be decided by evidence found during
   implementation of what's least surprising, and documented either way)
   rather than silently creating a second identical relationship record.
6. No regression to the entity-import pipeline: existing recognizeHeaders/
   buildImportPlan/commitImport behavior and header-alias lists are
   byte-for-byte unchanged.
7. `findEntitiesByCanonicalName` is a genuinely new, additive `db.js`
   function — it does not alter `findEntityIdentifier`, `searchEntities`,
   or any existing index definition.

## Verification approach (same standing sandbox limitation)

No Chromium/browser available in this session (independently reconfirmed
before this task, not assumed from any prior task's finding). Verification
will be, in order of strength available:
1. Node-executable tests against the real, extracted `db.js` functions
   (`findEntitiesByCanonicalName`, `createRelationship`,
   `assertRelationshipSemantics`) and the new plan-builder logic, using a
   synthetic dataset modeled directly on ADR-013's documented real-column
   names (`From_Tag`/`To_Tag`/`Relationship_Type`), including the
   ambiguous-match and invalid-type/direction cases explicitly.
2. `node --check` on every modified/new file.
3. Manual DOM trace (not executed) for the new UI mode-switch/preview
   rendering, following R3-002-MANUAL_AUDIT.md's established method.
4. Explicit non-silent flagging of what remains unverified (real
   IndexedDB transactions, real file upload/parse of an actual .xlsx,
   real browser DOM) — reported as PARTIAL, not PASS, consistent with
   every task in this sandbox class since R3-002.

## State changes

Per `SYSTEM_OF_RECORD_MATRIX.md`'s precedence rule: that file updated
first (new row for R2-003, M-16's state changed from OPEN to reference
this task), then `V3_GAP_REGISTER.txt` (M-16's STATE column updated), then
`PROJECT_STATE.md` (Completed tasks log entry + Next task update).
