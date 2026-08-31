# TASK CONTRACT

Task ID: R1-003
Phase: R1 — Canonical Identity + Local Persistence Contract (web/IndexedDB track, per ACR-001)
Title: Close the three MAJOR/MINOR data-integrity gaps found by R1-002's adversarial review

## Objective

Fix, narrowly and only, the gaps identified and live-confirmed in
`00_CONTROL/tasks/R1-002-REVIEW.md` (option "b" of its Recommendation),
per explicit operator instruction (2026-08-27):

1. **R002-REV-03 (self-referencing relationship):** `createRelationship`
   must reject `source_entity_id === target_entity_id`.
2. **R002-REV-04 (duplicate relationship):** `createRelationship` must
   reject an exact duplicate of `[source_entity_id, target_entity_id,
   relationship_type]` against an existing relationship — application-
   enforced (matching the `entity_identifier` → `ux_entity_identifier`
   pattern already established in R1-001/R1-002), not a new IndexedDB
   index, so no schema/version change and no ACR is required.
3. **R002-REV-05 (no double-submit guard):** all four R1-002 forms
   (`entity-form` is R1-001 and out of scope; `relationship-form`,
   `evidence-form`, `link-evidence-form` are R1-002 and in scope) must
   disable their submit button for the duration of the in-flight
   `await`, re-enabling on success or failure, to prevent a rapid
   double-tap from firing two submissions before the first completes.

Also close R002-REV-06 (test-coverage gap) as a direct consequence: the
new rejections in (1) and (2) must be covered by real, executed
browser tests — not just implemented.

This task does **not** address R002-REV-01 (evidence panel
directionality) or R002-REV-02 (duplicated RELATIONSHIP_TYPES list) or
R002-REV-07 (unrendered `source_reference` field) — those remain
OBSERVATION/MINOR items explicitly out of this narrow scope, left open
in `R1-002-REVIEW.md` for a future task if ever prioritized.

## Inputs

- `00_CONTROL/tasks/R1-002-REVIEW.md` (source of every finding fixed here)
- `00_CONTROL/tasks/R1-002-TASK_RESULT.md` (baseline this builds on)
- `03_DOMAIN/RELATIONSHIP_CONTRACT.md`
- `04_DATA/INDEXEDDB_SCHEMA_R0.md` (`relationship` store + referential
  integrity policy sections)
- `30_WEB_APP/db.js`, `30_WEB_APP/app.js` (current, post-R1-002 state —
  read in full before editing, not rewritten from scratch)

## Allowed files

Only within `30_WEB_APP/` plus the two control-log files:
- `30_WEB_APP/db.js` (extend `createRelationship` with the two new
  rejection checks; no other function's signature or behavior changes)
- `30_WEB_APP/app.js` (add submit-button disable/re-enable to
  `handleCreateRelationship`, `handleCreateEvidence`,
  `handleLinkEvidence`; surface the two new rejection messages via the
  existing `toast` mechanism — no new UI mechanism introduced)
- `30_WEB_APP/README.md` (document the fix, one short addition)
- `00_CONTROL/PROJECT_STATE.md` (status update only)
- `00_CONTROL/DECISION_LOG.md` (append-only, if a decision is recorded)

No new files outside `30_WEB_APP/` (this task produces its own
`TASK_CONTRACT`/`TASK_RESULT` under `00_CONTROL/tasks/`, same pattern as
R1-001/R1-002). No new object stores or indexes; `DB_VERSION` stays `1`.

## Constraints

- The self-loop check and the duplicate check are both enforced inside
  `createRelationship`'s existing single `readwrite` transaction (over
  `entity`, `evidence`, `relationship`) — no new transaction, no
  possibility of a race between the check and the write within one call.
- The duplicate check reads existing relationships via the
  `ix_relationship_source` index (already defined since R1-001's schema)
  filtered by `target_entity_id` and `relationship_type` in application
  code — matching the "no native compound uniqueness for this
  combination" reality noted in the review; this is an
  application-enforced check, not a new IndexedDB index, per
  `INDEXEDDB_SCHEMA_R0.md`'s "no schema mutation without ACR" rule (a
  compound unique index would be a schema change; a pre-write
  application check is not).
- Error messages for both new rejections must be as specific and
  non-generic as the existing referential-integrity errors (e.g. name
  the offending entity/relationship-type combination), consistent with
  R1-001/R1-002's established error-message style.
- Submit-button disable must not break the existing "disabled until an
  entity is selected" gating already on `#relationship-submit` — the two
  conditions (no entity selected vs. in-flight submission) must compose
  correctly, not fight each other.
- No change to `entity`, `entity_identifier`, `evidence`, or `source`
  creation logic — this task is scoped to `relationship` creation and
  submit-button UX only.
- No IndexedDB schema mutation, no new ACR, no build toolchain, no
  changes to `21_ANDROID_STARTER/*` or any file outside the allowed list.
- No regression to any R1-001 or R1-002 acceptance test.

## Acceptance tests

1. Attempting `createRelationship` with `source_entity_id === target_entity_id`
   (a real, existing entity) is rejected with a clear, non-crashing
   error naming the entity.
2. Attempting `createRelationship` with the same
   `[source_entity_id, target_entity_id, relationship_type]` as an
   already-existing relationship is rejected with a clear error; a
   *different* `relationship_type` between the same two entities is
   still allowed (this is not a full edge-uniqueness ban, only exact
   duplicates).
3. Rapid double-submission (two `submit` events fired before the first
   `await` resolves) on the relationship form results in at most one
   persisted relationship, verified by checking `relationship` store
   count before/after, not just by UI appearance.
4. Same double-submission protection verified for the evidence-creation
   and evidence-linking forms.
5. All R1-001 and R1-002 acceptance tests still pass unmodified (full
   regression, not a subset).
6. `node --check` (as ES module) passes on modified `db.js` and `app.js`.

## Forbidden changes

- No IndexedDB schema/index/version change.
- No relaxation or removal of any existing referential-integrity check.
- No change to `RELATIONSHIP_TYPES`, `RELATIONSHIP_CONTRACT.md`, or the
  meaning of `direction`/`validation_state`.
- No fix attempted for R002-REV-01, R002-REV-02, or R002-REV-07 (out of
  scope — see Objective).
- No touching `21_ANDROID_STARTER/*`.

## Deliverables

Modified `30_WEB_APP/db.js`, `30_WEB_APP/app.js`, `30_WEB_APP/README.md`;
updated `00_CONTROL/PROJECT_STATE.md`; this task's own
`00_CONTROL/tasks/R1-003-TASK_RESULT.md` and (per the now-established
R1-002 pattern) an adversarial `00_CONTROL/tasks/R1-003-REVIEW.md`.

## Dependencies

R1-002 (Status: PASS) and its review (`R1-002-REVIEW.md`, findings
R002-REV-03/04/05/06) — this task exists specifically to close those
findings, not to redesign relationship semantics.
