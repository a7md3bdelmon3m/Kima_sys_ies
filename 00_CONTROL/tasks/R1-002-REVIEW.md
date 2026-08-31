# REVIEW

Task ID: R1-002
Reviewer role: Adversarial Reviewer (per `14_LLM_OS/roles/adversarial_reviewer.md`),
applying `14_LLM_OS/protocols/REVIEW_PROTOCOL.md`'s seven dimensions.
Scope: the R1-002 diff only (`30_WEB_APP/{db.js,app.js,index.html,styles.css}`
additions). R1-001's existing entity/identifier code is reviewed only where
R1-002 touches or depends on it.

Every finding below carries a severity (BLOCKER/MAJOR/MINOR/OBSERVATION)
and a claim classification (SOURCE-DERIVED/IMPLEMENTED/TEST-VERIFIED/
MODEL-INFERENCE/UNRESOLVED), per `NO_HALLUCINATION_PROTOCOL.md`.

## 1. Requirements

Checked against `00_CONTROL/tasks/R1-002-TASK_CONTRACT.md`'s acceptance
tests 1–7.

- Tests 1–6 map directly to Suite A/B checks B3/B4/B8/B7/A5 and pass —
  TEST-VERIFIED (see R1-002-TASK_RESULT.md's raw output).
- Test 7 (`node --check` + manifest validity) — TEST-VERIFIED, re-run
  before packaging.
- **Finding R002-REV-01 (MINOR):** the task contract's objective point 3
  says the ledger panel shows "evidence records linked to the selected
  entity." The actual implementation shows evidence linked via the
  entity's *outgoing* relationships only (`ix_relationship_source`).
  Evidence reachable only via a relationship where the selected entity
  is the *target* is invisible in that entity's panel. This is
  documented as a Warning in TASK_RESULT ("relationship UI only
  supports creating... from the currently selected entity") but the
  panel's *read* path has the same asymmetry and was not separately
  called out until this review. IMPLEMENTED as designed, not a contract
  violation (the contract didn't specify direction), but worth flagging
  as an interpretation choice a stricter reading could dispute.
  Classification: IMPLEMENTED (the asymmetry itself) / MODEL-INFERENCE
  (that this is likely to surprise a user browsing bidirectionally).

## 2. Architecture

- No IndexedDB schema/version change — TEST-VERIFIED (`DB_VERSION`
  remains `1`; `openDb`'s `onupgradeneeded` block is byte-identical to
  R1-001, confirmed by diff during this review).
- `db.js` remains the sole data-access layer; `app.js` contains no raw
  `indexedDB.*` calls — TEST-VERIFIED by grep (`grep -n "indexedDB\." app.js`
  returns zero matches outside `db.js`).
- No build toolchain introduced — TEST-VERIFIED (no `package.json`,
  no bundler config added to `30_WEB_APP/`).
- **Finding R002-REV-02 (OBSERVATION):** `RELATIONSHIP_TYPES` is defined
  once in `db.js` and duplicated as literal `<option>` values in
  `index.html`. This is not a contract violation (the contract doesn't
  forbid it) and both lists were verified identical by manual
  comparison — TEST-VERIFIED for current correctness — but it is a
  maintenance risk: a future edit to one list without the other would
  silently desync UI-permitted values from `db.js`-enforced values
  (the latter would still catch it, per Decision in TASK_RESULT, so no
  silent corruption — just a confusing UX where the dropdown offers a
  value the layer then rejects, or omits one it would accept).

## 3. Source / data integrity

This is where adversarial review found real, previously-undocumented
gaps — not just re-confirming what TASK_RESULT already claimed.

- **Finding R002-REV-03 (MAJOR): no protection against a self-referencing
  relationship.** `createRelationship` never checks
  `source_entity_id !== target_entity_id`. Confirmed live in this review
  session (not just by code read) via a direct `db.js` call against a
  real entity through the running app:
  ```
  { "selfLoopAllowed": true,
    "selfLoopRelationshipId": "rel_mtamljvj_5285141126184992891518152994832201918" }
  ```
  Classification: TEST-VERIFIED. No contract in
  `03_DOMAIN/RELATIONSHIP_CONTRACT.md` or `06_GRAPH/TOPOLOGY_RULES.md`
  explicitly forbids self-loops, so this is not a contract violation —
  but it is an unreviewed engineering-data integrity gap with real
  downstream risk once R9 (topology traversal) and R12 (impact
  analysis) are built on this data. Not fixed in this review (out of
  R1-002's already-closed scope per the task contract); flagged for an
  explicit follow-up decision, not silently patched.
- **Finding R002-REV-04 (MAJOR): no duplicate-relationship guard.**
  Unlike `entity_identifier`'s `[namespace, identifier]` unique index,
  `relationship` has no uniqueness constraint on
  `[source_entity_id, target_entity_id, relationship_type]`. Confirmed
  live: creating the identical relationship twice (same source, target,
  type) against two real entities succeeded both times, producing two
  distinct IDs for the same fact, and a subsequent count confirmed 3
  relationships total (1 self-loop from R002-REV-03 + 2 duplicates):
  ```
  { "duplicateAllowed": true,
    "rel1Id": "rel_mtamljvk_499186221159023780417171350861174704464",
    "rel2Id": "rel_mtamljvk_2085415011236141712940420013002193425109",
    "totalRelationshipsForEntity": 3 }
  ```
  Classification: TEST-VERIFIED. `INDEXEDDB_SCHEMA_R0.md`'s
  `relationship` store section does not define such an index, so this
  again is not a contract violation of the *written* schema — but it
  means duplicate submission (e.g. from Finding R002-REV-05 below) is
  not caught at the data layer.
- **Finding R002-REV-05 (MINOR): no double-submit guard on any R1-002
  form.** None of `handleCreateRelationship`, `handleCreateEvidence`, or
  `handleLinkEvidence` disable their submit button during the `await`,
  unlike the relationship form's *initial* enable/disable logic (which
  only gates on entity selection, not on in-flight submission). A rapid
  double-tap on mobile (the primary target device per
  `01_PRODUCT/MOBILE_UX_REQUIREMENTS.md`) can fire two submissions
  before the first `toast` renders. Combined with R002-REV-04 (no
  duplicate guard at the data layer), this can silently produce
  duplicate relationship/evidence records from a single user action.
  TEST-VERIFIED as a code-level gap (confirmed by reading the event
  handlers; not separately re-tested with simulated rapid double-taps
  in this review — that specific interaction timing is MODEL-INFERENCE
  regarding real-device likelihood, though the code path itself is
  TEST-VERIFIED to have no guard).
- Referential integrity for the cases the task contract *did* require
  (non-existent target entity, non-existent source, non-existent
  evidence) — TEST-VERIFIED, re-confirmed by re-reading B4/B8's assertion
  logic and their raw output in TASK_RESULT (rejection messages
  correctly reference the specific missing ID, not a generic error).
- Transaction atomicity (`createRelationship`/`createEvidence`/
  `linkEvidenceToRelationship` each run inside one `readwrite`
  transaction spanning all stores they touch) — TEST-VERIFIED by code
  read; matches `INDEXEDDB_SCHEMA_R0.md`'s referential-integrity policy
  requirement.

## 4. Tests

- The original 18 named checks / 19 assertions are TEST-VERIFIED — raw
  output already in `R1-002-TASK_RESULT.md`, re-inspected during this
  review for internal consistency (no check's assertion text and
  reported PASS/FAIL contradict each other).
- **Finding R002-REV-06 (MAJOR): none of R002-REV-03/04/05 above were
  covered by the original test suite.** The original suite tested that
  *invalid* references are rejected, but never tested self-loops or
  duplicate submission because the task contract's acceptance tests
  didn't ask for it — this is a test-coverage gap traceable to a
  requirements gap (Finding R002-REV-03/04 in section 1), not a case of
  tests being written and then not run. Classification: UNRESOLVED
  (whether this needs a dedicated task or can be deferred to R9's
  topology work is an open decision, not decided by this review).
- Regression coverage of R1-001 — TEST-VERIFIED (A1–A6 in Suite A/F all
  re-ran and passed against the modified files, not skipped).

## 5. Performance

- Not adversarially tested in this review beyond what TASK_RESULT
  already disclosed as untested (large relationship/evidence counts).
  `listEvidenceForEntity`'s implementation does one `get()` per
  relationship inside a loop rather than a single indexed bulk read —
  TEST-VERIFIED by code read to be correct, but its cost is O(n)
  round-trips for an entity with n outgoing relationships.
  Classification of impact: MODEL-INFERENCE (no measurement was taken;
  at R1-002's realistic personal-use data volumes this is very unlikely
  to be perceptible, but this is inference, not a measured claim).
  Severity: OBSERVATION, not MINOR — flagging the pattern for awareness
  ahead of R9/R14, not claiming a present problem.

## 6. Security

- No new external network calls introduced (evidence/relationship
  creation is 100% local IndexedDB) — TEST-VERIFIED by code read (no
  `fetch`/`XMLHttpRequest` added anywhere in the R1-002 diff).
- `escapeHtml` is applied consistently to all new user-supplied strings
  rendered into the ledger panel (relationship type label via a
  static lookup table — not user input — target display name,
  evidence locator, evidence type) — TEST-VERIFIED by code read; no new
  unescaped interpolation found in the diff.
- **Finding R002-REV-07 (OBSERVATION):** `source_reference` (free text)
  is accepted by `createRelationship` but is not currently rendered
  anywhere in the UI (only stored). No XSS exposure today because it's
  never interpolated into the DOM, but if a future task starts
  displaying it, it must go through `escapeHtml` like the other fields —
  noting this now so it isn't missed later. MODEL-INFERENCE (a
  forward-looking risk, not a present vulnerability).

## 7. Release impact

- No `manifest.webmanifest` or `service-worker.js` changes — TEST-VERIFIED,
  re-confirmed unchanged by this review's diff check; no re-versioning
  or cache-invalidation concern introduced.
- Findings R002-REV-03/04/05 (self-loops, duplicate relationships, no
  double-submit guard) do not block this task's own acceptance criteria
  (none of which required these protections) but should be weighed
  before any claim that R1's data layer is "release-ready" in a
  stronger sense than "meets its own task contracts." Recommend they be
  captured as explicit follow-up items rather than silently assumed
  fixed or silently assumed acceptable.

## Summary by severity

| Severity | Count | IDs |
|---|---|---|
| BLOCKER | 0 | — |
| MAJOR | 3 | R002-REV-03 (self-loop), R002-REV-04 (duplicate relationship), R002-REV-06 (test-coverage gap for both) |
| MINOR | 2 | R002-REV-01 (evidence panel directionality), R002-REV-05 (no double-submit guard) |
| OBSERVATION | 3 | R002-REV-02 (duplicated type list), performance note (§5), R002-REV-07 (unrendered field, forward risk) |

**No BLOCKER-severity finding exists.** All three MAJOR findings are
genuine data-integrity gaps but none contradicts a written contract
(`RELATIONSHIP_CONTRACT.md`, `INDEXEDDB_SCHEMA_R0.md`) — they are gaps
the contracts themselves are silent on, discovered by adversarial
testing that went beyond the task contract's own acceptance criteria.
Per `MASTER_PROJECT_CONTRACT.md`'s Definition of Done, this review's
existence and its unresolved findings are being reported explicitly
rather than treated as blocking R1-002's already-evidenced PASS status
retroactively — the task contract's acceptance tests were fully met;
this review found additional issues *outside* that contract's scope,
which is exactly what adversarial review is for.

## Recommendation

Do not silently fix R002-REV-03/04/05 as part of closing this review.
Options for the operator to decide explicitly:
(a) accept as known-open technical debt, tracked here and in
    TASK_RESULT's Unresolved issues, to be addressed opportunistically
    in R9 (topology) or a dedicated small task;
(b) open a small explicit follow-up task now (e.g. R1-003) scoped
    narrowly to: reject self-loop relationships, add a
    `[source_entity_id, target_entity_id, relationship_type]` uniqueness
    check (application-enforced, matching the `entity_identifier`
    pattern — no schema/index change required, since IndexedDB
    compound-key uniqueness could be added later as a real ACR-covered
    index if this data grows large), and disable submit buttons during
    in-flight requests on all four R1-002 forms.
This review does not choose between (a)/(b) — that is an explicit
scope/priority decision, not a technical one this review can make
unilaterally.
