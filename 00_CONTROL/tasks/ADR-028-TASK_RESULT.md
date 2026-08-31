# TASK RESULT

Task ID: ADR-028
Status: PASS (governance reconciliation only; no application code touched)
Date: 2026-08-30
Trigger: two independently-developed packages, both forked from the same
point immediately after ADR-025, were uploaded back in the same overall
conversation:
- `KIMA_EIS_V3_ADR025_R2003_HARNESS_2026-08-30.zip` was previously
  adopted and then received a governance-reconciliation pass (this
  session's own earlier ADR-026: M-13/`SYSTEM_OF_RECORD_MATRIX.md`
  corrections).
- `KIMA_EIS_V3_ADR027_T013_HARNESS_2026-08-30.zip` was uploaded next,
  containing its own differently-numbered ADR-026 (T012 backup/restore
  harness) and ADR-027 (T013 offline harness) — real work this
  session's tree lacked, built by a separate fork of the same starting
  point.

Per the user's explicit instruction ("ابدأ الدمج" / "start the
merge"), this task merges both forks' real, non-overlapping
contributions into one tree.

## What was analyzed before any file was touched

Both trees were diffed file-by-file against each other and against
this session's own prior work before any merge decision:

1. **`30_WEB_APP/` confirmed byte-identical** between the two trees
   (`diff -rq`, zero differences) — neither fork touched application
   code; both are pure test-harness/governance work.
2. **Every difference outside `30_WEB_APP/`** was confirmed additive
   on one side and simply absent (not contradicted) on the other:
   - The `ADR027` tree added `T012-test-harness/` (17/17 checks,
     backup/restore round-trip) and `T013-test-harness/` (11/11
     checks, offline operation) — genuinely new real-Chromium
     evidence this session's own tree lacked.
   - This session's own tree (via its own earlier ADR-026) had
     corrected `V3_GAP_REGISTER.txt`'s M-13 row and brought
     `SYSTEM_OF_RECORD_MATRIX.md` up to date through ADR-025 — work
     the `ADR027` tree lacked (its own `SYSTEM_OF_RECORD_MATRIX.md`
     was still at its original 2026-08-28/ADR-011 state, never
     updated even through its own ADR-026/027).
3. **Independently re-verified, not merely re-read**: this task
   re-ran the `T013-test-harness`'s own documented "real methodology
   problem" — that `Network.emulateNetworkConditions({offline:true})`
   does not actually block `fetch()` to a `localhost`-hosted target in
   this Chromium build — via a standalone probe script before trusting
   the harness's claim. Confirmed independently: `fetch()` to the
   local server returned `status:200, ok:true` while
   `navigator.onLine` correctly read `false`. This is a real, load-
   bearing finding, not an assumption inherited from the other
   session's ADR text.

## Merge decision

Adopted the **`ADR027` tree as the merge base** — it has the more
complete real-browser evidence (T012/T013 harnesses this session's own
tree lacked). This session's own governance-correction work
(M-13/`SYSTEM_OF_RECORD_MATRIX.md`) was re-applied on top — not copied
verbatim from the earlier tree, since that earlier tree's
`SYSTEM_OF_RECORD_MATRIX.md` did not yet know about H-09/M-10/ADR-026/
027 (copying it verbatim would have regressed those rows back to
OPEN).

## Files modified

- `00_CONTROL/DECISION_LOG.md` — added ADR-028 (this merge's own
  record — distinct from either tree's differently-numbered ADR-026,
  named explicitly as such to avoid the same kind of ambiguity ADR-019
  and this session's own earlier ADR-026 both had to resolve for prior
  numbering collisions).
- `00_CONTROL/V3_GAP_REGISTER.txt` — M-13 corrected OPEN → FIXED,
  citing the tool's presence/use since ADR-017 and naming ADR-028 as
  the correction record.
- `00_CONTROL/SYSTEM_OF_RECORD_MATRIX.md` — brought current through
  ADR-027 (nine ADRs beyond its last real update at ADR-011/013):
  - `CR-06`: BLOCKED → PARTIALLY VERIFIED, itemizing R3-002/R2-003/R9/
    R12's ADR-020-025 evidence AND this tree's own H-09/M-10 ADR-026/
    027 evidence together.
  - `CR-07`: cites this merge's own M-13/matrix-staleness finding as a
    further real instance of the contradiction class this row tracks.
  - `H-05`/`H-06`: real-device-confirmation citations (ADR-021/022)
    added alongside the existing Node evidence.
  - Two new rows added: `H-09` (backup/restore, FIXED via ADR-026's
    17/17) and `M-10` (offline, FIXED via ADR-027's 11/11) — neither
    existed as named rows in this file before, in either tree.
  - `M-16` (R2-003), the R3-002 row, the R9 row, `M-19`, `M-20`,
    `M-21`: each upgraded with real-browser/real-device citations,
    matching this session's own earlier ADR-026 pass, adapted for
    this (newer, more complete) base tree.
  - `M-23` (R12) added, citing ADR-022's real-device confirmation.
  - Header/closing notes corrected to state the file's own staleness
    plainly and to stop claiming CR-06 cannot be upgraded "based on
    anything in this file" — no longer accurate once CR-06 itself is
    PARTIALLY VERIFIED with citations to real ADR-020–027 evidence.
- `00_CONTROL/RELEASE_VERIFICATION/V3_RELEASE_GATE.txt` — `BROWSER E2E`
  and `FULL T001-T016` corrected from blanket "BLOCKED/NOT VERIFIED"
  to itemized partial status including H-09/M-10's closure, matching
  `SYSTEM_OF_RECORD_MATRIX.md`'s CR-06 row; final decision line
  updated to name the specific remaining gaps (file:// mode, the real
  file-picker+confirm() UI path, SW install/update cycling, combined
  regression, performance, P&ID) rather than a generic "blocked gates."
- `00_CONTROL/V3_CURRENT_STATE.txt` — added the missing reference to
  T013/ADR-027 in the real-Chromium-verified summary paragraph (this
  tree's own file had it for ADR-026/T012 but not yet ADR-027/T013 —
  a small drift found and corrected in this same merge pass rather
  than left for a future task).
- `00_CONTROL/PACKAGE_CONTENTS.md` — file count corrected to include
  this task's own new file.
- `00_CONTROL/RELEASE_VERIFICATION/V3_FILE_INVENTORY.txt` /
  `V3_FILE_SHA256.txt` — regenerated via `regen_inventory.mjs` after
  all edits stabilized.

## What was deliberately NOT done

- No row was marked beyond what its cited evidence supports. H-09/
  M-10/M-16/the R3-002 row/the R9 row all still explicitly name their
  own remaining gaps (file:// mode, the real confirm()-dialog UI path,
  install/update cycling, the M-22 leak-scenario specifically) rather
  than being rounded up to "fully verified."
- CR-06 was not marked CLOSED — corrected from BLOCKED to PARTIALLY
  VERIFIED, a real change reflecting real new evidence, but the
  release gate's final decision (do not mark RELEASE VERIFIED yet) is
  unchanged, since real gaps genuinely remain.
- No application code in `30_WEB_APP/` was opened for editing — this
  task is governance/documentation reconciliation only, consistent
  with both source trees' own scope.
- This session's own earlier ADR-026 governance work was not copied
  verbatim into this tree — it was re-derived against this tree's
  current, more advanced state (H-09/M-10 already FIXED here) so as
  not to regress anything the `ADR027` tree had already achieved.

## Tests executed

1. **Full `30_WEB_APP/` byte-diff** between both source trees — zero
   differences, confirming neither fork touched application code.
2. **Full `00_CONTROL/` diff** between both trees — confirmed every
   difference was additive-and-non-overlapping before merging anything.
3. **Independent re-verification of `T013-test-harness`'s own claimed
   methodology finding** (the `emulateNetworkConditions`/localhost
   false-negative risk) via a standalone probe run in this task,
   rather than trusting the other tree's ADR text alone — confirmed:
   `fetch()` to `localhost` returns `200` even under CDP's
   offline-emulation flag, while `navigator.onLine` correctly flips.
   This independently validates why `T013-test-harness` was right to
   use a real SIGKILL of the HTTP server instead of relying on
   emulation alone.
4. `node 00_CONTROL/consistency_check.mjs` — run before this task's
   edits (baseline) and after every edit stabilized (final check
   below) — both clean, confirming this documentation-only pass
   introduced no drift.
5. `node 00_CONTROL/regen_inventory.mjs` — regenerated the canonical
   194-file inventory/hash pair after all edits.
6. Final `node 00_CONTROL/consistency_check.mjs` → clean `PASS`, exit
   0, only the 2 expected self-referential warnings.
7. `node --check` — clean on all 8 `30_WEB_APP/*.js` files (confirming
   this task, despite touching governance files extensively, left
   application code untouched and syntactically intact).
8. `node 30_WEB_APP/decision.test.mjs` → `DECISION_ENGINE_TEST=PASS`.

## Result

Both forks' real contributions are merged: this tree now has T012
(backup/restore, 17/17) and T013 (offline, 11/11) real-Chromium
evidence AND a governance record (`V3_GAP_REGISTER.txt`,
`SYSTEM_OF_RECORD_MATRIX.md`, `V3_RELEASE_GATE.txt`) that accurately
reflects all of it, cross-checked and corrected rather than assumed
consistent. `consistency_check.mjs` passes cleanly. Remaining real
gaps before any RELEASE VERIFIED label — named explicitly in
`V3_RELEASE_GATE.txt` and `SYSTEM_OF_RECORD_MATRIX.md`'s CR-06 row —
are: `file://` mode for R2-003 and T012, the real backup-import
file-picker+`confirm()` UI path, Service Worker install/update-cycle
testing, a single combined all-modules regression pass, performance
testing, and P&ID acceptance. None of these are newly discovered by
this task; all were already correctly named as open in the trees' own
harness READMEs and ADR text — this task's contribution is making sure
the governance-summary documents actually say so too, consistently
with each other and with `DECISION_LOG.md`.
