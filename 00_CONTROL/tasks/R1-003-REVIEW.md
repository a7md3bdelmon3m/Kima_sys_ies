# REVIEW

Task ID: R1-003
Reviewer role: Adversarial Reviewer (per `14_LLM_OS/roles/adversarial_reviewer.md`),
applying `14_LLM_OS/protocols/REVIEW_PROTOCOL.md`'s seven dimensions.
Scope: the R1-003 diff only (`30_WEB_APP/{db.js,app.js}` changes closing
`R1-002-REVIEW.md` findings R002-REV-03/04/05/06).

## 1. Requirements

All four acceptance tests in `00_CONTROL/tasks/R1-003-TASK_CONTRACT.md`
pass — TEST-VERIFIED, raw output in `R1-003-TASK_RESULT.md`. The negative
case (a *different* `relationship_type` between the same two entities
must remain allowed, i.e. this is not a blanket multi-edge ban) was
explicitly tested (R003-4) and confirmed correct, not just assumed from
reading the code.

## 2. Architecture

- No schema/index/version change — TEST-VERIFIED (`DB_VERSION` unchanged;
  the duplicate check reuses the existing `ix_relationship_source` index
  from R1-001's schema, confirmed by diff).
- Both new checks (self-loop, duplicate) execute inside
  `createRelationship`'s existing single `readwrite` transaction, before
  the write — TEST-VERIFIED by code read and confirmed behaviorally: a
  rejected call leaves the `relationship` count unchanged (verified via
  R003-3's `totalRels` check landing at exactly 2, not 3, after the
  rejected duplicate attempt).

## 3. Source / data integrity

This is where this review specifically re-tested R1-002-REVIEW's
findings to confirm they are actually closed, not just superficially
patched — and went one step further than the original task contract
required, probing for a race condition the fix might have missed.

- **R002-REV-03 (self-loop) — CONFIRMED CLOSED.** Live re-test: calling
  `createRelationship` with `source_entity_id === target_entity_id` on a
  real entity now throws, naming the entity ID in the message.
  TEST-VERIFIED.
- **R002-REV-04 (duplicate) — CONFIRMED CLOSED for the exact-match case.**
  Live re-test: identical `[source, target, relationship_type]` is
  rejected, naming the existing `relationship_id`. A *different*
  `relationship_type` between the same pair is correctly still allowed
  (R003-4). TEST-VERIFIED.
- **New probe, not required by the R1-003 task contract (R003-REV-01,
  OBSERVATION): cross-tab concurrency was tested against the duplicate
  check**, since an application-level "read-then-write inside one
  transaction" pattern is only safe against concurrency if the
  transaction itself is atomic across browsing contexts sharing the same
  IndexedDB database (two tabs of the same origin) — not just within a
  single call. Two separate Playwright pages against the same origin
  fired the identical `createRelationship` call concurrently
  (`Promise.all` across both pages, no artificial delay). Result:
  ```
  { "r1": { "threw": false, "id": "rel_mtb4lwnd_..." },
    "r2": { "threw": true, "message": "Referential integrity violation: source_entity_id \"...\" does not exist" },
    "finalCount": 1 }
  ```
  Only one relationship persisted — **the race is not exploitable**,
  TEST-VERIFIED. However, the specific rejection reason for the losing
  call (`r2`) was a referential-integrity error about the source entity,
  not a duplicate-relationship error — most likely because `r2`'s
  browsing context had not yet observed `r1`'s entity-creation write at
  the moment it ran, an artifact of this specific test's timing (both
  entities were created via `page1` immediately before the concurrent
  call, leaving a narrow window for `page2` to read stale state), not
  evidence that the duplicate-check path itself was exercised under true
  concurrency. Classification: TEST-VERIFIED (no duplicate was
  persisted, confirmed by `finalCount === 1`) but MODEL-INFERENCE
  regarding *why* — IndexedDB's `readwrite` transaction serialization is
  the standard-guaranteed reason duplicates can't land, not something
  this specific test's rejection message directly proves. A cleaner
  reproduction (pre-seed both entities and let both contexts observe
  them before firing the concurrent identical relationship call) would
  isolate the duplicate-check path specifically; not done in this
  review, left as UNRESOLVED if ever worth re-verifying precisely.
- **Finding R003-REV-02 (OBSERVATION):** `handleCreateRelationship`'s
  `finally` block re-enables the submit button via
  `submitBtn.disabled = !state.selectedEntityId`, but on the success path
  `selectEntity()` (called inside `try`) already sets
  `relSubmit.disabled = false` (same DOM element) before `finally` runs.
  Confirmed by code read that both paths compute the same final value
  when `state.selectedEntityId` is still set (the common case), so this
  is not a bug — but it is redundant logic worth simplifying if this
  function is touched again. Not fixed here (cosmetic, zero behavioral
  difference confirmed).

## 4. Tests

- 7/7 new R1-003 checks pass, including the double-submit protection
  tests (R003-5, R003-6) which measured actual persisted-record counts
  before/after concurrent clicks, not just UI appearance — TEST-VERIFIED,
  this is a stronger verification method than checking the DOM alone
  (a UI could look fine while silently creating duplicate records
  underneath; this suite specifically guards against that).
- Full regression: 19/19 R1-001/R1-002 checks re-ran and passed against
  the modified files — TEST-VERIFIED, zero regression.
- The new cross-tab race probe (§3) is additional, review-only test
  coverage, not part of the committed R1-003 test suite; if this
  scenario should be permanently guarded against regressions, it should
  be formalized into the test harness as its own task, not left as a
  one-off review artifact. UNRESOLVED (left as an explicit open item,
  not silently assumed covered going forward).

## 5. Performance

- The duplicate check adds one extra `getAll()` on the
  `ix_relationship_source` index per `createRelationship` call, scoped to
  the calling entity's existing outgoing relationships only (not a full
  table scan) — TEST-VERIFIED by code read to be correctly scoped.
  Cost grows with a single entity's own outgoing-relationship count, not
  with total relationship count — acceptable at R1's realistic personal-
  use data volumes, consistent with R1-002-REVIEW's existing performance
  OBSERVATION. No new performance concern introduced.

## 6. Security

- No new user input surface introduced; error messages for both new
  rejections interpolate only IDs already generated by `genId()` or
  supplied by the caller and validated to already exist as real records
  — no new unescaped-rendering risk (these messages go through the
  existing `toast()` path, whose text-only rendering was already
  reviewed safe in R1-002-REVIEW).

## 7. Release impact

- No manifest/service-worker change — TEST-VERIFIED unchanged.
- This task closes 3 of the 5 open findings from `R1-002-REVIEW.md`
  (R002-REV-03, R002-REV-04, R002-REV-05, and by direct consequence
  R002-REV-06's test-coverage gap). R002-REV-01 (evidence panel
  directionality) and R002-REV-02 (duplicated type list) and R002-REV-07
  (unrendered field) remain open, unchanged, out of this task's declared
  scope — not silently resolved, not silently forgotten.

## Summary by severity

| Severity | Count | IDs |
|---|---|---|
| BLOCKER | 0 | — |
| MAJOR | 0 | — |
| MINOR | 0 | — |
| OBSERVATION | 2 | R003-REV-01 (race-probe result attributed to timing, not conclusively isolated to the duplicate-check path specifically), R003-REV-02 (redundant but harmless disabled-state double-write) |

**No BLOCKER, MAJOR, or MINOR finding.** R1-003 successfully closes the
three MAJOR/MINOR findings it targeted from R1-002's review, confirmed
by live re-test rather than by re-reading the original findings and
assuming the fix matches. The two OBSERVATION-level notes here are both
about test-methodology precision, not about any defect in the shipped
behavior.

## Recommendation

No further action required to close R1-003. The two OBSERVATION items
are optional future polish (R003-REV-01: write a more precisely isolated
concurrency test if this area is revisited; R003-REV-02: simplify the
redundant disabled-state assignment next time `handleCreateRelationship`
is touched) — neither blocks proceeding to R2.
