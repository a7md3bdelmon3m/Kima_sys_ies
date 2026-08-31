# TASK CONTRACT

Task ID: R3-002-B
Phase: R3 — Explorer + Search / Release-gate closure
Title: Fresh integrated-tree browser verification (Gate D) — attempted; discovered and fixed a release-blocking IndexedDB schema-upgrade bug

## Objective
Run the fresh integrated-tree real-browser regression that R3-002, INT-001,
and INT-002 all reported as environment-blocked, to close Gate D
(CR-06) or report precisely why it still cannot close. Do NOT assume the
prior "environment blocked" finding still holds — verify it directly in
this execution environment first.

## Inputs
- 30_WEB_APP/* as delivered in this package (DB_VERSION=7, 10 stores)
- 00_CONTROL/tasks/R2-001-test-harness/cdp.js (existing dependency-free
  CDP driver, reused as-is per project precedent — not reinvented)
- 00_CONTROL/PROJECT_STATE.md "Next task" guidance

## Allowed files
- 30_WEB_APP/db.js (fix only, minimal diff)
- 00_CONTROL/PROJECT_STATE.md (state update)
- 00_CONTROL/tasks/R3-002-B-TASK_CONTRACT.md, R3-002-B-TASK_RESULT.md
- 00_CONTROL/DECISION_LOG.md (new ADR entry)
- 00_CONTROL/V3_GAP_REGISTER.txt (if a gap must be logged)

## Constraints
- No schema/version change unless the bug fix strictly requires it.
- No UI/feature change. This task verifies and, only if a defect blocks
  verification, fixes the minimum code required to make the existing,
  already-specified behavior actually run.
- Every claim classified per NO_HALLUCINATION_PROTOCOL
  (SOURCE-DERIVED / IMPLEMENTED / TEST-VERIFIED / MODEL-INFERENCE /
  UNRESOLVED).
- Run `node 00_CONTROL/consistency_check.mjs` before closing, per
  GOV-SYNC-004 required discipline.

## Acceptance tests
1. Headless Chromium (already present in this sandbox at
   ~/.cache/puppeteer/chrome) launched, connected via CDP.
2. index.html served over local HTTP (127.0.0.1) — hosted mode.
3. Page loads, IndexedDB opens without exception, KimaDB is defined on
   window, DB_VERSION/store count match db.js.
4. Minimum smoke coverage: create entity, list entities, searchEntities
   basic query — each checked via real Runtime.evaluate calls, not
   inferred.
5. Same checks repeated under file:// mode.
6. Full console/exception log captured as evidence for every navigation.

## Forbidden changes
- No changes to relationship/evidence/observation/decision domain logic.
- No changes to import.js, decision.js, graph.js, impact.js unless a
  defect in them is directly discovered and blocks the acceptance tests.
- No silent schema bump.

## Deliverables
- 00_CONTROL/tasks/R3-002-B-TASK_RESULT.md with full evidence.
- Fixed 30_WEB_APP/db.js if a defect is found (with before/after diff
  recorded in the task result and DECISION_LOG.md).
- 00_CONTROL/PROJECT_STATE.md updated to reflect real, tested state.

## Dependencies
- None blocking. Independent of R2-003/R9.
