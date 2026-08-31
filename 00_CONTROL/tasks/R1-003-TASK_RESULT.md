# TASK RESULT

Task ID: R1-003
Status: PASS

## Files created

- `00_CONTROL/tasks/R1-003-TASK_CONTRACT.md`
- `00_CONTROL/tasks/R1-003-TASK_RESULT.md` (this file)
- `00_CONTROL/tasks/R1-003-REVIEW.md`

## Files modified

- `30_WEB_APP/db.js` — `createRelationship` extended with two new checks,
  both inside its existing single `readwrite` transaction, before the
  three referential-integrity checks it already had:
  - rejects `source_entity_id === target_entity_id` (closes
    R002-REV-03), checked before opening the transaction (pure input
    validation, no read needed)
  - rejects an exact duplicate of `[source_entity_id, target_entity_id,
    relationship_type]` against any existing relationship, found via the
    existing `ix_relationship_source` index scoped to the caller's
    `source_entity_id` (closes R002-REV-04). A different
    `relationship_type` between the same two entities remains allowed —
    this is not a blanket multi-edge ban.
  No other function's signature or behavior changed; no schema/index/
  version change.
- `30_WEB_APP/app.js` — `handleCreateRelationship`, `handleCreateEvidence`,
  and `handleLinkEvidence` each now disable their form's submit button
  for the duration of the in-flight request (`try`/`finally`, re-enabled
  on both success and failure paths) to close R002-REV-05. No other
  handler logic changed.
- `30_WEB_APP/README.md` — documented R1-003's scope and test results.
- `00_CONTROL/PROJECT_STATE.md` — status update (see State changes).
- `00_CONTROL/DECISION_LOG.md` — not modified this task (no new
  architectural decision beyond what R1-002-REVIEW.md's Recommendation
  already framed; the operator's explicit choice to open R1-003 is
  recorded implicitly by this task's own existence and TASK_CONTRACT,
  consistent with how R1-001/R1-002 handled routine task decisions).

## Tests executed

Real functional tests against headless Chromium 141.0.7390.37 via
Playwright/CDP — the same methodology as R1-001/R1-002, not a new
untested approach.

**New R1-003 suite** (`run_r1003_tests.mjs`):
| ID | Check | Maps to acceptance test | Result |
|---|---|---|---|
| R003-1 | Page loads with zero console errors | — (baseline sanity) | PASS |
| R003-2 | Self-loop relationship rejected, error names the entity | 1 | PASS |
| R003-3 | Exact-duplicate relationship rejected, error names the existing relationship_id | 2 | PASS |
| R003-4 | Different relationship_type between the same two entities still allowed (negative case) | 2 | PASS |
| R003-5 | Rapid double-submit on relationship form → exactly one persisted relationship (measured by store count, not UI appearance) | 3 | PASS |
| R003-6 | Rapid double-submit on evidence form → exactly one persisted evidence record | 4 | PASS |
| R003-7 | All three R1-002 forms' submit buttons present for guard wiring (structural check) | — | PASS |

7/7 passed.

**Full regression** (`run_tests.mjs`, unmodified from R1-002): all 19
R1-001/R1-002 checks (A1–A6, B1–B8, F1–F4) re-ran against the R1-003-
modified files and passed — zero regression.

**Total this task: 26/26 real-browser assertions passed** (7 new + 19
regression).

Syntax validation (re-run before packaging):
```
$ node --input-type=module --check < app.js  → OK
$ node --input-type=module --check < db.js   → OK
```

## Test evidence

Raw console output, R1-003 suite:
```
PASS — R003-1: page loaded with no console errors  []
PASS — R003-2: self-loop relationship rejected (acceptance test 1)  {"threw":true,"message":"A relationship cannot reference the same entity (\"ent_mtb4k0sq_279554335734731928427384306542619087090\") as both source and target","entityId":"ent_mtb4k0sq_279554335734731928427384306542619087090"}
PASS — R003-3: exact duplicate relationship rejected (acceptance test 2)  {"rel1Id":"rel_mtb4k0t1_16651447924211782729873401143567842572","dupThrew":true,"dupMessage":"Duplicate relationship: [ent_mtb4k0sz_1272899311138064951823023474964081027624 → ent_mtb4k0t0_2301974979392850907020149285153843398139, piping] already exists (relationship_id \"rel_mtb4k0t1_16651447924211782729873401143567842572\")","diffTypeThrew":false,"diffTypeMessage":"","rel2Id":"rel_mtb4k0t3_3717727051928846855632319426569178436","totalRels":2}
PASS — R003-4: different relationship_type between same entities still allowed (acceptance test 2, negative case)  {same object as above, totalRels=2}
PASS — R003-5: double-submit on relationship form creates at most one relationship (acceptance test 3)  countBefore=2 countAfter=3 delta=1 btnDisabledAfterSettle=false
PASS — R003-6: double-submit on evidence form creates at most one evidence record (acceptance test 4)  before=0 after=1 delta=1
PASS — R003-7: all three R1-002 form submit buttons present for guard wiring  {"relBtnExists":true,"evBtnExists":true,"linkBtnExists":true}

ALL R1-003 TESTS PASSED
```

Raw console output, full regression re-run (`run_tests.mjs`):
```
=== SUITE A: hosted mode (http) — R1-001 regression + R1-002 ===
PASS — A1 through A6, B1 through B8 (all 14 checks) — see R1-002-TASK_RESULT.md for the assertion text; identical output reproduced against R1-003's modified files.
=== SUITE F: file:// mode ===
PASS — F1 through F4 (all 4 checks)

ALL TESTS PASSED
```

## Decisions

- **The self-loop check runs before opening the IndexedDB transaction**
  (pure argument validation, like the existing `relationship_type`
  membership check), while the duplicate check runs inside the
  transaction (it requires a read). This mirrors the existing code's
  own pattern of validating what can be validated statically first.
- **The duplicate check is scoped to `[source_entity_id,
  target_entity_id, relationship_type]` exactly, not to `[source_entity_id,
  target_entity_id]` alone** — per the task contract and
  `R1-002-REVIEW.md`'s explicit framing ("a different relationship_type
  between the same two entities is still allowed"), since a pump can
  legitimately have both a `piping` and a `dependency` relationship to
  the same line, and collapsing those would be a real product regression,
  not a fix.
- **No new IndexedDB index was added for the duplicate check** — the
  existing `ix_relationship_source` index (already present since
  R1-001's schema) is sufficient, scoped by `source_entity_id` then
  filtered in application code for `target_entity_id` +
  `relationship_type`. This avoids a schema/version bump and keeps this
  task's `Allowed files` constraint (no ACR needed) intact.

## Warnings

- (Carried over from R1-002, unchanged by this task) `file://` ES-module
  CORS behavior on real Android browsers remains unverified beyond
  desktop Chromium.
- **New, from this task's review:** the cross-tab race-condition probe
  in `R1-003-REVIEW.md` confirmed no duplicate relationship is
  persisted under concurrent identical calls from two browsing contexts
  sharing the same IndexedDB database, but the specific test's timing
  meant the *reason* the second call failed was a referential-integrity
  rejection (stale read of the entity list) rather than a direct
  exercise of the duplicate-check path under true concurrency. The
  headline result (no duplicate lands) is TEST-VERIFIED; the precise
  mechanism is MODEL-INFERENCE (attributed to IndexedDB's standard
  `readwrite` transaction serialization, not directly proven by this
  specific test's error message). See `R1-003-REVIEW.md` §3 for the
  full raw evidence and a suggestion for a more precisely isolated
  follow-up test if this matters later.

## Unresolved issues

- (Carried over from R1-002, unchanged) `standard_mapping`/`change_event`
  UI, backup/export mechanism, in-repo test runner reachable from the
  phone — all still open, R1-003 did not touch these.
- (Carried over from R1-002-REVIEW, explicitly NOT addressed by this
  task's narrow scope) R002-REV-01 (evidence ledger panel only shows
  evidence via outgoing relationships, not incoming) and R002-REV-02
  (RELATIONSHIP_TYPES duplicated between `db.js` and `index.html`'s
  `<option>` list) and R002-REV-07 (unrendered `source_reference` field
  would need `escapeHtml` if ever displayed) remain open — out of scope
  by this task's own TASK_CONTRACT, not forgotten.
- (New, from this task's own review) whether to formalize the cross-tab
  concurrency probe into the committed test suite, and whether to run a
  more precisely isolated version of it, is left as an explicit open
  item — not silently assumed necessary or unnecessary.

## State changes

`PROJECT_STATE.md` updated: R1 phase now explicitly notes all three
R1-00x tasks (identity, relationship+evidence, integrity-gap closure) as
PASS with adversarial review completed for R1-002 and R1-003 alike; no
open BLOCKER; "Next task" remains R2 (unchanged from after R1-002 — this
task did not introduce a new phase, it closed debt within R1).

## Recommended next task

R1 is now complete to a higher bar than R1-002 alone achieved: both its
vertical slices are PASS, and every MAJOR/MINOR finding from R1-002's
adversarial review is either closed (R1-003) or explicitly deferred with
a documented reason (R002-REV-01/02/07). Recommend proceeding to
**R2 — Import + Diff** per `05_IMPORT/IMPORT_PIPELINE.md` and
`05_IMPORT/IMPORT_CONFLICT_POLICY.md`, as already recommended in
`R1-002-TASK_RESULT.md`.
