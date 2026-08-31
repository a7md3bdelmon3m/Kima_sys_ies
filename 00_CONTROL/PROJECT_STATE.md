# PROJECT STATE

Project: KIMA EIS Personal Web (PWA)
Pack: v3.0 — INT-001 INTEGRATED FINAL3
State: CORE_CRUD_GRAPH_IMPACT_REAL_DEVICE_CONFIRMED_2026-08-30
— INTEGRATION RECONSTRUCTED; RELEASE GATE PARTIALLY NARROWED BY A REAL
ANDROID CHROME SESSION (2026-08-30, ADR-020/021/022), FULL GATE D
MATRIX STILL PENDING (context browser + exact/prefix/normalized search,
filter, sort, pagination all live in 30_WEB_APP; persistent Plant/
Area/Unit/System context bar clickable as a navigational filter
(R3-002); a Canvas-based relationship graph view (R9) wired into the
entity ledger panel with pan/zoom/pinch/tap-to-navigate; a
pre-existing direction-value bug in the relationship-creation form
(backward vs. reverse) was found and fixed in the course of R9; a
second, more severe pre-existing bug — openDb()'s onupgradeneeded
handler calling .objectStore() on the IDBDatabase handle instead of
the upgrade transaction, which broke every fresh IndexedDB
initialization — was found and REAL-BROWSER-VERIFIED-FIXED in a
parallel session (R3-002-B) and merged into this tree via ADR-017;
consistency_check.mjs/regen_inventory.mjs tooling adopted from that
session and used to find and correct real control-document drift
(stale V6 backup-format/cache-version claims) in this tree; on
2026-08-30 the user ran the merged tree on a real Android Chrome
device for the first time in project history — ADR-020 fixed a
sub-16px-font auto-zoom-on-load defect, ADR-021 confirmed entity
creation (R1-001), relationship creation with the corrected direction
options (R1-002/R1-003, M-19), and Topology graph Canvas rendering
(R9, M-18) all working live, and ADR-022 confirmed Impact Analysis
(R12, M-23) working live — see DECISION_LOG.md for the full record)

## Current phase
Core CRUD + relationship + Topology graph + Impact analysis
(R1-001, R1-002/R1-003, R9, R12) — REAL-DEVICE CONFIRMED on Android
Chrome, 2026-08-30 (ADR-021/ADR-022; see M-18/M-19/M-23 in
V3_GAP_REGISTER.txt, now FIXED). This is the first real-browser
confirmation of any of this project's UI in its history, but it
covers a single entity/relationship shape only (Test pump/Test
Line, one containment edge) — not a full regression. R3 — Explorer +
Search remains search/filter/sort/pagination PASS (logic-verified);
R3-002 (context filter) still PARTIAL, no real-device evidence yet
— see M-17 (OPEN) and 00_CONTROL/tasks/R3-002-TASK_RESULT.md.

## Current objective
With the core CRUD/graph/impact chain now real-device-confirmed
(ADR-021/ADR-022), the next objective is to extend real-device
coverage to the features that still have none: R3-002 (context bar
filter, M-17), R2-001/R2-002/R2-003 (CSV/Excel import), T012
(backup/restore), and T013 (offline mode) — see Next task for the
recommended order. R3-001's explorer (KimaDB.searchEntities
centralizing match/filter/sort/paginate in db.js) remains
logic-verified only; full disposition in
00_CONTROL/tasks/R3-001-TASK_RESULT.md and
00_CONTROL/tasks/R2-002-TASK_RESULT.md's external code review.

## Completed
- Product direction

- Architecture direction
- LLM operating model
- Benchmark integration
- Starter Android project (reference only — see ADR-007)
- Initial domain/data contracts
- QA gate model
- ACR-001: platform shift to HTML/CSS/JS + IndexedDB (approved 2026-08-25)
- File integrity verification of full v2.0 pack (68/68 SHA256 match)
- Governance sync (2026-08-25): MASTER_PROJECT_CONTRACT.md,
  REFERENCE_ARCHITECTURE.md, 14_LLM_OS/roles/ (new web_engineer.md,
  data_engineer.md updated, android_engineer.md marked historical),
  15_BUILD/* (BUILD_CONTRACT.md, BUILD_COMMANDS.md, new
  WEB_RUNTIME_BASELINE_2026-08-25.md; old Gradle-track files archived
  under 15_BUILD/ANDROID_TRACK_HISTORICAL/), and README.md all brought
  into alignment with ACR-001/ADR-007 — closing the gap where control-log
  files (this file, DECISION_LOG.md, INDEXEDDB_SCHEMA_R0.md) reflected the
  web platform shift but core contract/architecture/role/build files still
  described the superseded native-Android track.
- Governance sync (2026-08-26, GOV-SYNC-002 / ADR-009): closed a second,
  narrower documentation gap that GOV-SYNC-001 (ADR-008) did not reach —
  `17_ROADMAP/ROADMAP.md`'s R1 entry still named "Room/SQLite
  implementation" and `16_TESTING/TEST_MATRIX.md`'s T016 still read
  "release APK smoke test". Both corrected to name the IndexedDB/web
  bundle track in force since ACR-001/ADR-007. Wording-only; no
  architecture, schema, role, or test semantics changed. See
  00_CONTROL/tasks/GOV-SYNC-002-TASK_RESULT.md.
- R1-002: relationship + evidence CRUD extended into the ledger panel,
  with application-enforced referential integrity for entity/source/
  evidence references and a minimal source-registration stand-in ahead
  of the real R2 import pipeline. 27/27 real browser checks passed
  (22 hosted + 5 file://), including full regression of all R1-001
  checks. See 00_CONTROL/tasks/R1-002-TASK_RESULT.md.
- R2-001: CSV workbook import pipeline (fingerprint → parse → structural
  validation → header recognition → normalization → candidate identity
  resolution → conflict report → preview → commit), idempotent re-import
  keyed off the existing entity_identifier unique index. 48/48 real
  browser checks passed (17+4 new, 22+5 regression), zero regression to
  R1-001/R1-002. A real duplicate-identifier-handling bug was found and
  fixed during testing (see task result's Decisions). .xlsx not yet
  supported — CSV only, an explicit scoping decision (no local Excel
  parser available). See 00_CONTROL/tasks/R2-001-TASK_RESULT.md.
- R2-002: .xlsx/.xls import support, via a locally-vendored SheetJS
  library (30_WEB_APP/lib/xlsx.full.min.js, loaded by a local <script>
  tag, zero CDN dependency — verified by live network-request
  monitoring, not just code reading). Extracted selectively from two
  externally-uploaded reference packages after full code review: only
  the library file was adopted (hash-verified); the surrounding
  application code from one package was explicitly rejected for a CDN
  dependency that breaks offline use, new Function()-based dynamic code
  execution, an undocumented IndexedDB schema/version bump, and 21
  silent-error-swallowing occurrences — full disposition recorded in
  00_CONTROL/tasks/R2-002-TASK_RESULT.md. 67/67 real browser checks
  passed (14+5 new, 22+5+17 regression), zero regression anywhere.
- R3-001: Explorer + Search — KimaDB.searchEntities (centralized exact/
  prefix/normalized match, type/status filter, sort, pagination) and
  KimaDB.parseContextHierarchy (best-effort Plant/Area/Unit/System
  parse of the existing free-text `context` field — no schema change;
  an unstructured context correctly shows no invented breadcrumb).
  app.js's flat unfiltered entity list replaced with a real search UI;
  a persistent context bar added to the header. Two real bugs found and
  fixed during testing: a test-seed omission, and a genuine app bug
  where any context string with zero "/" characters was incorrectly
  treated as a one-level hierarchy. 90/90 real browser checks passed
  (21+6 new, 22+5+17+14+5 regression), zero regression anywhere. Full
  record in 00_CONTROL/tasks/R3-001-TASK_RESULT.md.

## Platform (post-ACR-001)
Implementation target: HTML/CSS/JavaScript, IndexedDB persistence, delivered as
(a) standalone local file (`file://`, no server) and (b) installable PWA
(Service Worker + Web App Manifest). Native Android/Kotlin/Room track is
superseded and retained for reference only.

## INT-001 verification status
Fresh integrated-tree browser regression was attempted on 2026-08-28. The managed Chromium execution environment blocked local HTTP and file origins, preventing module-level browser execution. Structural/static checks passed; the release gate remains PENDING. See `00_CONTROL/INT-001-INTEGRATED-VERIFICATION.md`.

## Integration status
INT-001 has reconstructed the R2/R4/R5 authoritative baseline. R5 is the executable baseline and five unique historical review/contract artifacts from R2-001-CLOSED were restored. No schema or feature change was introduced. Fresh integrated-tree browser regression is still required before a release-verified label. See `00_CONTROL/INTEGRATION_MANIFEST.md`, `00_CONTROL/INTEGRATION_REVIEW.md`, and `00_CONTROL/tasks/INT-001-TASK_RESULT.md`.

## ADR-013 (2026-08-28)
ADR-012's fix was verified only against a synthetic case. The real source
workbook (`K103_DEXPI_CFIHOS_Engineering_Master_v2_2.xlsx`, 41 sheets) was
obtained and tested directly against the real `recognizeHeaders` logic.
Its real vocabulary (`Tag_ID`, `Functional Type`/`Source/Functional Type`)
was not covered by ADR-012's `KNOWN_HEADERS` — fixed by extending it.
`02_Asset_Register`, `07_Valves`, `08_Equipment` now auto-map correctly.
`06_Instruments` is deliberately left to manual mapping: its only
candidate type column (`CFIHOS Class`) is the literal string `no_match` in
25.6% of its 831 rows, so auto-mapping it would silently corrupt a quarter
of imported records. Inspecting the rest of the real workbook also
surfaced a real import-scope boundary, logged as gap M-16: the entity-only
import pipeline has no path for relationship-shaped sheets
(`09_Piping_Connections`, `10_Relationships`, 179 rows combined) — that
requires a new pipeline stage, not a header-list extension. See
`DECISION_LOG.md`'s ADR-013 entry and `V3_GAP_REGISTER.txt` M-16.

## Next task
**Superseded by the 2026-08-30 real-device session (ADR-020/021/022)
— see below for the current recommendation. Kept here for history:**
~~Highest priority — was CRITICAL until fixed, now HIGH pending
independent re-verification (M-20)~~: this concern is narrowed, not
closed, by ADR-021/ADR-022 — the real-device session exercised
IndexedDB writes (entity/relationship creation) on a fresh profile
without error, which is evidence against M-20 having reintroduced a
first-launch failure, but no dedicated fresh-profile init test was
run in isolation, and no regression pass covered R2-003's
relationship-import or R9-MERGE-001's three fixes (M-22) specifically.

**Current recommendation (2026-08-30, post-ADR-022)**: the core
CRUD + relationship + Topology graph + Impact analysis slice
(R1-001, R1-002/R1-003, R9, R12) is now real-device-confirmed for a
single entity/relationship shape. The highest-value next step is to
extend that same real-device session's method — walking a real
feature manually on the user's own Android Chrome device and
recording the outcome as an ADR — to the features that still have
zero real-device evidence, in this order:
1. **R3-002 (context bar as filter, M-17)** — smallest remaining gap
   in the already-confirmed CRUD/graph/impact area; natural next
   click-through in the same UI already open on the device. STILL
   OPEN as of ADR-023 (2026-08-30) — R3-001's harness confirmed the
   context-hierarchy breadcrumb *renders* correctly (real Chromium,
   not yet real device), but not the click-to-filter interaction
   R3-002/M-17 is specifically about.
2. ~~**R2-001/R2-002/R2-003 (CSV/Excel import)**~~ — PARTIALLY
   SUPERSEDED by ADR-023 (2026-08-30, same session, this Claude
   instance rather than the user's device): R2-001 (CSV, 21/21 across
   hosted+file://) and R2-002's underlying xlsx pipeline (19/19
   across hosted+file://) are now real-Chromium-verified — see
   V3_CURRENT_STATE.txt's "VERIFIED VIA HEADLESS CHROMIUM" section.
   This is real browser execution but NOT real-device/touch/viewport
   evidence, so a real-device pass on an actual file picked on-device
   still has standalone value and is not fully redundant — but it is
   now lower-urgency than before, since the underlying import logic
   itself is broad-and-deep browser-tested. R2-003's
   relationship-specific commit path remains logic-verified only
   (M-16) and is the more valuable remaining target if real-device
   time is limited.
3. ~~**T012 (backup/restore) and T013 (offline mode)**~~ — SUPERSEDED
   (2026-08-30, this Claude session, not the user's device): both now
   have dedicated headless-Chromium test harnesses, built and run
   following ADR-023's "headless Chromium first" pattern. T012:
   17/17 checks passed (ADR-026). T013: 11/11 checks passed (ADR-027)
   — this run also found and worked around a real methodology gap
   (CDP's `Network.emulateNetworkConditions` does not block `fetch()`
   to `localhost` in this Chromium build; the harness instead
   SIGKILLs the actual local server process for a genuine network
   cutoff — see `T013-test-harness/README.md`). Neither is real-device
   evidence — both remain this session's own headless-Chromium
   sandbox only, same lighter evidence tier as ADR-023/024/025.
   Remaining gaps named explicitly rather than assumed closed: for
   T012, `file://` mode and the real file-picker+`confirm()` UI path;
   for T013, install/update-cycle testing and `file://` mode (a
   different claim in kind, since Service Workers do not register
   under `file://` at all).
Each real-device pass should be recorded as its own ADR in
DECISION_LOG.md, following the same discipline ADR-020/021/022
established: screenshot-backed, V3_GAP_REGISTER.txt/
V3_CURRENT_STATE.txt updated in the same task, consistency_check.mjs +
regen_inventory.mjs re-run afterward. Headless-Chromium passes should
follow ADR-023's discipline instead: each `*-test-harness/` directory
has its own `cdp.js` copy by design (not interchangeable — a stray
mismatched copy caused two false failures in ADR-023 itself, self-
corrected within that session), and V3_GAP_REGISTER.txt rows closed
this way should be worded distinctly from real-device rows, per
ADR-023's own example.

Beyond real-device coverage, the remaining CRITICAL/HIGH gaps
(CR-07 systemic state-reconciliation, H-01 through H-12) continue to
accumulate as logic-verified-only implementations; none is newly
urgent as of this session, but CR-07's underlying tooling
(consistency_check.mjs) should be pointed at PROJECT_STATE.md itself
in a future task, since this file is not currently covered by that
check and this session's drift (a stale "Next task" section) was
caught only by manual review, not automation.

M-16 (Excel-to-relationship import) is now FIXED (logic/regression-
verified) via R2-003 — see 00_CONTROL/tasks/R2-003-TASK_RESULT.md. A
real, deliberately-constructed risk beyond the originally-logged gap was
found and designed for during this task: `entity_type`+`canonical_name`
has no uniqueness constraint anywhere in this schema, so a relationship
endpoint resolving to 2+ entities is a real, reachable case — handled as
an explicit BLOCKER, never a silent first-match pick. Verified end-to-
end (16/16 assertions) against a real `.xlsx` built with this project's
own bundled SheetJS library, through a byte-for-byte identical copy of
the real `import.js`.

## Completed tasks log
- ADR-020/ADR-021/ADR-022 (2026-08-30) — first real-device (Android
  Chrome) session in project history. ADR-020 root-caused and fixed a
  sub-16px-font auto-zoom-on-load defect (styles.css). ADR-021
  confirmed entity creation, relationship creation (with the
  corrected forward/reverse/bidirectional direction options), and
  Topology graph Canvas rendering all working live — closing M-18 and
  M-19 as real-device-confirmed. ADR-022 confirmed Impact Analysis
  working live, correctly excluding the source entity and reporting
  only the true target (Test Line) per impact.js's own logic — closing
  M-23. No code changes in ADR-021/ADR-022 (documentation of the
  user's own live testing); ADR-020 changed styles.css only. See
  DECISION_LOG.md for full detail.
- R2-003 — Relationship-import pipeline, parallel to the existing
  entity-import pipeline (per ADR-013's own recommendation, not folded
  into KNOWN_HEADERS/buildImportPlan). New: `db.js#findEntitiesByCanonicalName`
  (exact-match entity lookup by canonical_name, used since relationship-
  sheet endpoints like K103's From_Tag/To_Tag resolve via canonical_name,
  not entity_identifier's namespace/identifier pair — confirmed during
  pre-implementation review that no existing lookup covered this case);
  `db.js#commitImportRelationshipRow` (creates evidence, then calls the
  existing `createRelationship` — no parallel write path);
  `import.js`'s `RelationshipImportPipeline`
  (`RELATIONSHIP_KNOWN_HEADERS`, `recognizeRelationshipHeaders`,
  `buildRelationshipImportPlan`, `previewRelationshipImport`,
  `commitRelationshipImport`); `app.js`/`index.html`'s import-mode
  switch UI. Ambiguous-endpoint-match, nonexistent-endpoint, invalid-
  relationship_type, invalid-direction-for-type, and exact-duplicate
  cases all verified end-to-end (16/16 Node assertions) against a real
  `.xlsx` fixture built with this project's own bundled SheetJS, through
  a diff-confirmed byte-identical copy of the real `import.js`. Closes
  M-16. Status: PARTIAL, logic/regression-verified via Node, NOT real-
  IndexedDB/browser-verified — folded into the combined verification
  pass named above.
- R9-MERGE-001 — Pre-adoption review of the uploaded, more complete R9
  pack (interactive pan/zoom/pinch/tap-to-navigate topology renderer)
  against an independently-developed, simpler in-session R9 draft. User
  chose to adopt the uploaded pack as project baseline after a detailed
  comparison (category-naming fidelity to TOPOLOGY_RULES.md, presence of
  known defects). Found and fixed three real defects in the adopted
  pack before finalizing the adoption: (1) `graph.js#traverse`'s
  `relationshipTypes` filter blocked BFS traversal through non-matching
  edges instead of only filtering reported results (same defect class
  independently found in the parallel R9-001 draft's own review) — fixed
  by splitting the single `visited` Set into `walked`
  (BFS-continuation gate) and `reported` (result-inclusion gate); (2)
  `topology.js`'s `createTopologyRenderer` leaked one `window`
  resize-listener per entity selection (confirmed via direct
  simulation: 5 selections -> 5 accumulated listeners, 0 released) —
  fixed via a canvas-scoped `ResizeObserver` plus an explicit
  `destroy()` called from `app.js` on every reselection; (3)
  `topology.js` was missing from `service-worker.js`'s offline-cache
  `SHELL_ASSETS` list, which would have silently broken the Topology
  feature under installed/offline PWA use — fixed, cache version bumped
  v7->v8. All three re-verified via Node (real ES-module import of the
  actual edited files, not rewritten copies): zero regression on the
  existing H-05 6-case suite, `impact.js#calculateImpact`, and
  `decision.test.mjs`. See 00_CONTROL/tasks/R9-MERGE-001-TASK_RESULT.md
  and SYSTEM_OF_RECORD_MATRIX.md's M-22 row. Status: FIXED,
  logic/regression-verified via Node, NOT real-browser-verified — folded
  into the same combined verification pass named above.
- R1-001 — Web app bootstrap: entity + entity_identifier CRUD over
  IndexedDB, RTL shell, PWA manifest/service worker, file:// + hosted
  modes both verified via real headless-browser tests (11/11 passed).
  See 00_CONTROL/tasks/R1-001-TASK_RESULT.md.
- GOV-SYNC-002 — Corrected two residual native-Android-track references
  that GOV-SYNC-001 missed: ROADMAP.md R1 entry ("Room/SQLite" →
  IndexedDB) and TEST_MATRIX.md T016 label ("release APK smoke test" →
  release package smoke test). Documentation-only; does not change or
  block Next task. See 00_CONTROL/tasks/GOV-SYNC-002-TASK_RESULT.md.
- R1-002 — Relationship + evidence CRUD wired into the ledger panel
  (createRelationship, createEvidence, registerSource added to db.js;
  app.js UI extended). App-enforced referential integrity verified by
  real rejection tests (non-existent entity/source/evidence references
  all rejected with clear errors, not silent corruption). 27/27 real
  Chromium checks passed (22 hosted + 5 file://), zero regression to
  R1-001. Test driver: dependency-free raw-CDP harness (Puppeteer was
  unavailable in this sandbox — no network, no cached package), saved at
  00_CONTROL/tasks/R1-002-test-harness/ for reuse. See
  00_CONTROL/tasks/R1-002-TASK_RESULT.md.
- R2-001 — CSV import pipeline (import.js: fingerprint via SHA-256 over
  raw bytes, manual RFC-4180 CSV parser, structural validation, header
  recognition with EN/AR aliases, normalization, exact-match identity
  resolution via the existing unique index, BLOCKER/MAJOR/MINOR conflict
  classification, preview with zero DB mutation, commit with per-row
  insert/update). db.js gained sha256HexBytes, findEntityIdentifier,
  updateEntity. 48/48 real Chromium checks passed (17 new hosted + 4 new
  file:// + 22+5 regression of R1-001/R1-002). Testing caught and fixed
  a real bug: duplicate-in-file identifiers were only flagging the
  second+ occurrence as BLOCKER, silently committing the first — fixed
  to flag all rows sharing a duplicated key equally, since picking a
  "winner" among duplicates without review is an unreviewed identity
  decision the matching rule doesn't permit. CSV-only for now (no local
  Excel parser available — explicit scoping decision, not silent gap).
  See 00_CONTROL/tasks/R2-001-TASK_RESULT.md.
- R2-002 — .xlsx/.xls import support via a locally-vendored SheetJS
  build extracted from two externally-uploaded reference packages after
  full line-by-line code review. Only the library file was adopted
  (hash-verified identical in both sources); the surrounding
  application code was explicitly rejected — one package's
  import/export code loaded the same library from
  cdnjs.cloudflare.com despite bundling an unused local copy (breaks
  offline use), executed IndexedDB-stored strings via new Function()
  (code-injection-shaped pattern), bumped DB_VERSION 1→2 with 3 new
  undocumented object stores (no ACR), and had 21 silent
  error-swallowing occurrences. None of that was adopted. import.js
  gained parseXlsx/detectFormat; every pipeline stage after `parse`
  stayed completely unchanged (format-agnostic by design since R2-001).
  67/67 real Chromium checks passed (14 new hosted + 5 new file:// +
  22+5+17 full regression of R1-001/R1-002/R2-001), zero regression,
  zero CDN requests verified live via network monitoring (not just code
  reading). Full external-review disposition record in
  00_CONTROL/tasks/R2-002-TASK_RESULT.md.
- R3-001 — Explorer + Search: KimaDB.searchEntities (case-insensitive
  substring match covering exact/prefix search, type/status filter as
  AND, sort by updated_at/canonical_name/entity_type in both
  directions, pagination) and KimaDB.parseContextHierarchy (best-effort
  Plant/Area/Unit/System breadcrumb from the existing free-text
  `context` field, only when it actually contains "/" — no schema
  change, no invented hierarchy for unstructured context). app.js's
  flat unfiltered list became a real explorer; a persistent context bar
  was added, shown only when the selected entity's context is
  parseable. Testing found and fixed two real bugs (one in the test
  seed data, one in parseContextHierarchy itself — see
  R3-001-TASK_RESULT.md's Decisions for the full account). 90/90 real
  Chromium checks passed (21 new hosted + 6 new file:// + 22+5+17+14+5
  full regression of every prior task), zero regression anywhere. See
  00_CONTROL/tasks/R3-001-TASK_RESULT.md.

## Blocking issues
No application defect is currently classified as a blocker. The **INT-001 release gate itself is blocked by the execution environment** until a real-browser regression can run against the integrated tree. Known open item: `entity.context` field present in
`CANONICAL_ENTITY_CONTRACT.md` but absent from `SQLITE_SCHEMA_R0.sql` —
to be resolved explicitly in `04_DATA/INDEXEDDB_SCHEMA_R0.md`, not silently.

## Required discipline
Every implementation task must update this file and produce a Task Result.

## INT-002 Verification Update
- Task: INT-002 — Fresh Integrated Runtime & Browser Verification
- Static/structural verification: PASS
- Pipeline parsing verification: PASS
- Fresh Chromium runtime: ENVIRONMENT BLOCKED by managed local-origin policy
- Release Verified: NO
- No application/schema changes were made during INT-002.

## GOV-SYNC-005 (adopted from a parallel session's tree, 2026-08-29)
- `00_CONTROL/GATE_D_RUNBOOK.md` and
  `16_TESTING/GATE_D_TEST_MATRIX_T001-T016.md` added: step-by-step
  local-browser runbook plus a fillable, provenance-traced T001-T016
  matrix, prepared for the Gate D run that remains blocked in every
  sandbox this project has run in so far. No row claims PASS — nothing
  was executed in a real browser by this or the originating task.
- Updated as part of this task's own merge (ADR-019) to add T009/T010
  coverage for the R9-MERGE-001/R12 fixes now present in this tree, and
  a new T017 for R2-003's relationship import — none of which existed
  when the matrix was first written.

## ADR-019 merge update (2026-08-29) — R12 merged onto R2-003/R9-MERGE-001
- Adopted `KIMA_EIS_V3_R12_IMPACT_2026-08-29.zip`'s `impact.js` rewrite
  (full 5-step `IMPACT_ANALYSIS.md` contract) and its `app.js`/
  `index.html` UI wiring, merged onto this session's own R2-003/
  R9-MERGE-001 tree (uploaded back as
  `KIMA_EIS_V3_R2-003_RELATIONSHIP_IMPORT_2026-08-29.zip`) — the two
  packages were confirmed, by full per-file diff, to be non-overlapping
  additions from the same ADR-017 base, not competing edits.
- `app.js` hand-merged (not replaced) to preserve R9-MERGE-001's
  `activeTopologyRenderer`/`destroy()` disposal logic and R2-003's
  relationship-import UI, neither of which existed in the R12 package's
  own copy of this file. `db.js`, `graph.js`, `topology.js`,
  `service-worker.js`, `import.js`, `index.html`, `styles.css`: no
  changes needed — the R2-003 base already had everything R12 needed
  from them.
- Full rationale, per-file merge decisions, and verification (including
  a new integration test proving `calculateImpact` works correctly on
  top of `traverse`'s R9-MERGE-001 fix specifically): `DECISION_LOG.md`
  ADR-019.
- `V3_GAP_REGISTER.txt`: R12's browser-verification gap renumbered
  `M-22`→`M-23` (a real conflict — `M-22` was already assigned here to
  R9-MERGE-001's three-defect row, a different finding entirely).
  `M-16` retained as this tree's `FIXED` version (R2-003's own), not
  the R12 package's older `OPEN` copy.
- Verified: `node --check` clean on all 8 `30_WEB_APP/*.js` files;
  `decision.test.mjs` PASS; new integration test confirms
  `calculateImpact`+`traverse` work correctly together post-merge.
- **Not verified — same standing limitation as everything else in this
  project**: no real browser available. R12/R9-MERGE-001/R2-003 each
  keep their own PARTIAL/FIXED-pending-regression status individually.
- Next actual step, unchanged in substance from every prior task's own
  recommendation: a real-browser session covering, in one pass, R3-002
  (context bar), R9 (topology pan/zoom/tap + the type-filter/listener
  fixes), R2-003 (relationship import), R12 (impact button), and a
  full R1-001 baseline regression — see `GATE_D_RUNBOOK.md` and
  `16_TESTING/GATE_D_TEST_MATRIX_T001-T016.md` for the prepared
  execution material. `SYSTEM_OF_RECORD_MATRIX.md` and the canonical
  inventory/hash files still need a `consistency_check.mjs`/
  `regen_inventory.mjs` pass reflecting this merge — deferred to the
  same follow-up task that runs them, not split across two tasks.

## ADR-034 update (2026-08-31) — H-03 mutation operations
- Added `updateRelationship`, `deleteRelationship`, `updateObservation`,
  `updateStandardMapping`, `deleteEntity` to `30_WEB_APP/db.js` —
  closes H-03's `db.js`-layer scope (change ledger now has the missing
  update/delete operations to be non-uniform *across*, per H-03's
  corrected ADR-030 description).
- Both new delete operations implement `04_DATA/INDEXEDDB_SCHEMA_R0.md`'s
  referential-integrity rule literally for the first time in this
  codebase: block with a named conflict rather than cascade.
  `deleteEntity` checks all 8 dependent store types (entity_identifier,
  relationship×2 indexes, observation, standard_mapping, decision,
  drawing_locator, field_observation, evidence-as-subject);
  `deleteRelationship` checks evidence-as-subject. `change_event` rows
  deliberately excluded from `deleteEntity`'s blockers — an audit trail
  outliving its subject is correct, not an orphan.
- Full rationale, decision detail, and per-function field-mutability
  rules: `DECISION_LOG.md` ADR-034;
  `00_CONTROL/tasks/H-03-mutation-ops-implementation/TASK_RESULT.md`.
- `V3_GAP_REGISTER.txt`: H-03 OPEN → PARTIALLY CLOSED (Node-logic-
  verified only, real-browser confirmation still open, same standing
  limitation as every `db.js`-layer task in this project's history).
  New row `M-26` (LOW, P2) logs `deleteObservation`/
  `deleteStandardMapping` as an explicit, smaller-blast-radius
  follow-up, not silently dropped.
- Verified: `node --check` clean on all 8 `30_WEB_APP/*.js` files;
  dynamic ES-module import of `db.js` confirms all 5 new names present
  at runtime; brace/paren balance confirmed (297/297, 1121/1121); new
  `node_logic_test.mjs` 14/14 PASS; zero regression across all 4
  pre-existing Node-logic suites (`decision.test.mjs`, H-04,
  H-07/H-08/H-10/H-11, R2-004).
- **Not verified — same standing limitation as everything else in this
  project**: no real browser or `fake-indexeddb` reachable this
  session (re-confirmed fresh: no Chromium binary found, `npm install
  fake-indexeddb --no-save` fails 403). No hand-rolled IndexedDB shim
  was built to fake a deeper tier — real transaction/index/abort
  semantics are exactly what a shim gets subtly wrong, which would be
  worse than the honestly-scoped Node-logic tier used here.
- No UI wiring added — `db.js` layer only, matching H-03's own literal
  scope. `deleteEntity` should not be exposed in the UI without its own
  cascade-or-cancel confirmation flow, which does not exist yet.
- `consistency_check.mjs` (Gate C) re-run after this change: initially
  FAIL on expected file hash/size/count drift from editing `db.js` and
  adding the new test file; `regen_inventory.mjs` re-run;
  `PACKAGE_CONTENTS.md`'s stale file count corrected (208→209); Gate C
  re-confirmed **PASS** (2 expected self-referential warnings only).
- Next actual step, unchanged in substance from every prior task's own
  recommendation: a real-browser session covering R3-002, R9, R2-003,
  R12, T012's file:// gap, T013's install/update-cycle gap, H-04/
  H-07/H-08/H-10/H-11's real-DOM confirmation, R2-004's real-DOM
  merge-mode click path, and now H-03's five new functions, in one
  pass — see `GATE_D_RUNBOOK.md` and
  `16_TESTING/GATE_D_TEST_MATRIX_T001-T016.md`.

