# TASK RESULT

Task ID: ADR-033
Status: PASS (real code fix + Node-logic verification; real-DOM harness written but not executed)
Date: 2026-08-30

## Trigger

Continuation of `V3_RELEASE_GATE.txt` gap work, specifically the real
`#backup-import-input` file-picker+`confirm()` UI path named as an
example by the user.

## Independent finding

While reviewing `handleBackupImport` to plan the real-DOM harness,
found it hardcoded `{replace:true}` — `importBackup`'s tested,
working `replace:false` merge mode (verified since T012/ADR-026) was
never exposed in the UI. Unnamed gap, found by code review not by
request.

## What was built

- `#backup-import-mode` select (replace/merge, default replace) in
  `index.html`.
- `handleBackupImport` reads it, computes `isReplace`, passes to
  `importBackup`, shows mode-appropriate `confirm()` text.
- `R2-004-backup-import-ui-harness/test_r2004_hosted.js` — real-DOM
  harness (File+DataTransfer+change event) following T012's exact
  convention. **Written, not executed** — no Chromium/network
  reachable.
- `R2-004-backup-import-ui-harness/node_logic_test.mjs` — 6/6 PASS,
  source-string verification of the actual fix.

## Verification

- `node --check` clean on `app.js`.
- `R2004_LOGIC_TEST=PASS (6/6)`.
- Full regression: `decision.test.mjs`, H-04's and
  H-07/H-08/H-10/H-11's own tests all re-run, all still pass (db.js
  untouched this task).
- `which chromium chromium-browser google-chrome` — empty, confirming
  the real-browser gap fresh for this task.

## Gap-register status

New row M-25: PARTIALLY CLOSED. `V3_RELEASE_GATE.txt`'s corresponding
line updated to reflect the fix + unexecuted harness.

## Files modified

- `30_WEB_APP/index.html`, `30_WEB_APP/app.js`
- `00_CONTROL/tasks/R2-004-backup-import-ui-harness/` (new: cdp.js,
  test_r2004_hosted.js, node_logic_test.mjs, README.md)
- `00_CONTROL/V3_GAP_REGISTER.txt`, `SYSTEM_OF_RECORD_MATRIX.md`,
  `RELEASE_VERIFICATION/V3_RELEASE_GATE.txt`, `DECISION_LOG.md`
  (ADR-033), `PACKAGE_CONTENTS.md` (count),
  `RELEASE_VERIFICATION/V3_FILE_INVENTORY.txt` / `V3_FILE_SHA256.txt`
  (regenerated)

## What was deliberately NOT done

- Harness not claimed as run or as evidence until it actually executes
  against a real browser.
- No simulated/faked DOM behavior in Node to manufacture false
  confidence.
- `file://` mode, SW install/update-cycle testing, combined regression
  pass — untouched, still separately open.

## Result

Real, independently-useful UX fix (merge mode now reachable from the
UI) plus a ready-to-run test harness. The named real-browser evidence
gap in `V3_RELEASE_GATE.txt` remains open and is reported as such.
