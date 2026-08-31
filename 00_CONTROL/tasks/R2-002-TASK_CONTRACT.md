# TASK CONTRACT

Task ID: R2-002
Phase: R2
Title: Selective integration — vendor real `.xlsx` library locally; extend import.js with `.xlsx` support (no code carried over uncredited from external review sources)

## Objective

Two uploaded external packages (`30_WEB_APP_01.zip`, `30_WEB_APP_02.zip`)
were reviewed line-by-line as unsolicited reference material (not
authored under this project's task-contract discipline). Neither is
adopted wholesale. This task extracts exactly one safe, self-contained
asset — the vendored SheetJS `xlsx.full.min.js` library, byte-identical
in both uploads — and uses it to close the `.xlsz` gap explicitly left
open in `R2-001-TASK_RESULT.md`. All application logic in this task is
written fresh against this project's existing `import.js`/`db.js`
contracts; no `import-export.js` code from the reviewed upload is
carried over, for the reasons in Constraint 1.

## Inputs

- `00_CONTROL/MASTER_PROJECT_CONTRACT.md` (v3.0, 14 principles)
- `00_CONTROL/tasks/R2-001-TASK_CONTRACT.md` / `TASK_RESULT.md` (this
  task extends that pipeline, does not replace it)
- `05_IMPORT/IMPORT_PIPELINE.md`, `IMPORT_CONFLICT_POLICY.md`
- Existing `30_WEB_APP/import.js`, `db.js`, `app.js`
- External reference (read-only, not copied as code):
  `30_WEB_APP_01.zip`'s `lib/xlsx.full.min.js` — SHA-256
  `cc015130aa8521e7f088f88898eba949ccdcbfb38df0bd129b44b7273c3a6f41`,
  identical in `30_WEB_APP_02.zip`. SheetJS is a well-known, widely
  audited open-source library (Apache-2.0); vendoring the minified
  build itself (not the surrounding application code that used it) is
  the one asset from the review judged safe to adopt.

## Allowed files

Only inside `30_WEB_APP/`:
- `lib/xlsx.full.min.js` (new — vendored copy, byte-identical to the
  reviewed source, integrity-verified by hash before and after copy)
- `import.js` (extend — add an `.xlsx`/`.xls` parse branch alongside
  the existing CSV branch; all other pipeline stages already
  format-agnostic per R2-001)
- `app.js` (extend — file input `accept` attribute, minor UI label
  update)
- `index.html` (extend — load `lib/xlsx.full.min.js` via a local
  `<script>` tag; add `.xlsx,.xls` to the file input's `accept`)
- `README.md` (update feature list)

## Constraints

1. **No CDN dependency, under any circumstance.** The reviewed
   `30_WEB_APP_02` loaded the same library from
   `https://cdnjs.cloudflare.com/...` in `index.html` despite bundling
   an unused local copy, and its service worker explicitly bypassed
   caching for that URL — meaning the feature silently breaks offline
   and under `file://`, directly violating this project's "offline
   without a server" principle. This task loads
   `lib/xlsx.full.min.js` via a local relative `<script>` tag only.
   Verified by grep sweep (zero occurrences of `cdnjs`, `http://`, or
   `https://` anywhere in the delivered files) before TASK_RESULT is
   written.
2. **No dynamic code execution of any kind.** The reviewed
   `30_WEB_APP_02` executed user-data-derived strings via
   `new Function(...)` for a "QA rule engine" — a code-injection
   pattern this task does not replicate anywhere, including
   incidentally. `.xlsx` parsing only ever produces plain data
   (strings/numbers/booleans via `XLSX.utils.sheet_to_json`), never
   executable code paths.
3. **No silent error-swallowing.** The reviewed `import-export.js` used
   nested `try { ... } catch (err) { results.warnings.push(...) }` for
   sub-operations, which is fine, but `db.js` in that same package had
   21 occurrences of bare `catch (e) { /* تجاهل */ }` with zero
   reporting, several inside bulk-import loops. This task's `.xlsx`
   branch reuses R2-001's existing conflict-classification path
   (`BLOCKER`/`MAJOR`/`MINOR`/`INFO`, all surfaced in the preview) for
   every row-level failure — nothing is swallowed silently.
4. **No IndexedDB schema/version change.** The reviewed
   `30_WEB_APP_02` bumped `DB_VERSION` 1→2 and added three new object
   stores (`field_note`, `pid_drawing`, `qa_rule`) with no ACR on
   record — a direct governance violation this task does not repeat.
   This task adds zero stores/indexes; `.xlsx` rows map onto the exact
   same `entity`/`entity_identifier` write path R2-001 already built
   and tested for CSV.
5. **Reuse, don't fork, the pipeline.** `.xlsx` parsing produces the
   same intermediate row-array shape CSV parsing already produces
   (`rows[0]` = header, `rows[1..]` = data), so it feeds into the
   existing `validateStructure` → `recognizeHeaders` → `buildImportPlan`
   → `previewImport`/`commitImport` stages completely unchanged. Only
   the `parse` stage gains a second branch; nothing else in
   `import.js` is duplicated or forked per-format.
6. **Fingerprint stays raw-bytes-first**, per R2-001's existing
   constraint (`.xlsx` files are binary — `sha256HexBytes` over the
   file's `ArrayBuffer` already handles this with no change needed; the
   ordering guarantee — fingerprint before parse — must hold for
   `.xlsx` exactly as it does for CSV).
7. `xlsx.full.min.js` is loaded as a classic global-scope `<script>`
   (SheetJS's UMD build exposes a global `XLSX`), not an ES module
   import — `import.js` accesses it via `window.XLSX`, checked
   explicitly with a clear error if absent (mirrors R2-001's existing
   "fail loud, not silent" posture), not assumed present.
8. No build toolchain introduced — the library is a pre-built static
   file, loaded exactly as CSS/JS already are.

## Acceptance tests

Real functional tests against an actual Chromium browser (same
dependency-free raw-CDP harness pattern from R1-002/R2-001), covering:
- An `.xlsx` file (built via `XLSX.utils.json_to_sheet` for the test
  itself, since no binary `.xlsx` fixture exists yet in the pack)
  round-trips through fingerprint → parse → preview → commit
  identically to an equivalent CSV (same row count, same field
  mapping, same conflict detection).
- `.xlsx` fingerprinting is stable/differs correctly (T001-equivalent
  for the new format).
- `.xlsx` import is idempotent on re-import (T003-equivalent).
- Multi-sheet `.xlsx` files: only the first sheet is processed in this
  task (documented, not silently ignored) — or, if trivial to support
  once implemented, all sheets are processed with each sheet's rows
  tagged by sheet name in `sourceReference`. Decision recorded in
  TASK_RESULT based on actual implementation complexity encountered.
- Zero CDN network requests occur at any point (verified by checking
  the page never issues a request to any non-`localhost`/non-`file://`
  origin during the test run).
- Full regression: all R1-001/R1-002/R2-001 real-browser checks
  (48 total from R2-001-TASK_RESULT.md) still pass unmodified.

## Forbidden changes

- No file from either reviewed upload is copied verbatim except
  `lib/xlsx.full.min.js` itself (hash-verified).
- No `new Function`/`eval` anywhere.
- No CDN URL anywhere in any delivered file.
- No IndexedDB schema/version change.
- No edits to `21_ANDROID_STARTER/`, `15_BUILD/*`, role files, or other
  `00_CONTROL/` governance files during IMPLEMENT.

## Deliverables

- `30_WEB_APP/lib/xlsx.full.min.js` (vendored, hash-verified).
- Modified `30_WEB_APP/import.js`, `app.js`, `index.html`, `README.md`.
- `00_CONTROL/tasks/R2-002-TASK_CONTRACT.md` (this file).
- `00_CONTROL/tasks/R2-002-TASK_RESULT.md` with real test evidence,
  including an explicit "External code review disposition" section
  documenting exactly what was and wasn't adopted from the two uploads
  and why (per this project's anti-hallucination / evidence
  discipline — a review this thorough deserves a durable record, not
  just this conversation).
- `00_CONTROL/PROJECT_STATE.md` updated.
- `00_CONTROL/FILE_MANIFEST.md` regenerated.

## Dependencies

R1-001, R1-002, R2-001 (all PASS) — extends `import.js` without
modifying its CSV path or any `db.js` function signature already
tested.
