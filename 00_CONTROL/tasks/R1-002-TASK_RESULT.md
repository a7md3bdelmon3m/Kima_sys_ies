# TASK RESULT

Task ID: R1-002
Status: PASS

## Files created

- `00_CONTROL/tasks/R1-002-TASK_CONTRACT.md`
- `00_CONTROL/tasks/R1-002-TASK_RESULT.md` (this file)

## Files modified

- `30_WEB_APP/db.js` — added `getEntity`, `registerSource`, `listSources`,
  `sha256Hex`, `createEvidence`, `listEvidence`, `RELATIONSHIP_TYPES`,
  `createRelationship`, `listRelationshipsForEntity`. `openDb`'s store/index
  definitions untouched (already matched `INDEXEDDB_SCHEMA_R0.md` exactly
  since R1-001) — no schema/version change.
- `30_WEB_APP/app.js` — extended `selectEntity`'s ledger-panel rendering to
  show relationships (with direction arrow, type, validation_state badge,
  evidence-linkage indicator) and evidence linked to those relationships;
  added relationship-creation form, source-registration form, and
  evidence-creation form, each wired to the new `db.js` functions with
  user-facing error toasts on referential-integrity rejection. Original
  entity/identifier creation and rendering logic unchanged.
- `30_WEB_APP/styles.css` — added `select` to existing input styling (was
  previously unstyled), added `.evidence-registration` block styling
  (a `<details>`/`<summary>` disclosure) reusing existing color tokens; no
  new visual language introduced. `.identifier-row` given `flex-wrap: wrap`
  to accommodate the extra badge/chip now possible on relationship rows.
- `00_CONTROL/PROJECT_STATE.md` — Next task advanced past R1-002 (see
  State changes below).

## Tests executed

Real functional tests against an actual Chromium browser engine (headless
Chrome 131.0.6778.204, same binary/version as R1-001), driven directly via
the DevTools Protocol over a raw WebSocket (Node 22's built-in `WebSocket`
global) rather than Puppeteer — Puppeteer's npm package was not available
in this sandbox (no network egress to install it) and no cached copy of
the library itself was found, only the Chrome binary. This is a materially
equivalent test method: the same real browser engine executes the same
`30_WEB_APP` files unmodified; only the driver connecting to it differs
(raw CDP vs. Puppeteer's CDP wrapper). The minimal driver
(`Runtime.evaluate`, `Page.navigate`, console capture) is retained at
`00_CONTROL/tasks/R1-002-test-harness/` for reuse by future tasks (see
Decisions).

Two suites, both against the literal `30_WEB_APP` files (verified
byte-identical via SHA-256 to the files actually served/opened before
concluding the run):

**Suite A (re-run + extended) — hosted mode (`http://localhost:8899`)**
| ID | Check | Result |
|---|---|---|
| A1 | Page loads with zero console errors | PASS |
| A2 | Entity created via UI appears in list | PASS |
| A3 | Persisted entity record has all 8 `CANONICAL_ENTITY_CONTRACT.md` fields | PASS |
| A4 | Two `entity_identifier`-equivalent write persists (1 identifier added, verified count) | PASS |
| A5 | Duplicate `[namespace, identifier]` rejected via unique index, non-crashing toast | PASS |
| A6 | Data persists after full page reload | PASS |
| A7 | Document renders with `dir="rtl"` | PASS |
| A8 | Service worker registers under http(s) | PASS |
| R002-1 | Second entity created (relationship target) | PASS |
| R002-2 | Relationship form renders once ≥2 entities exist | PASS |
| R002-3 | Relationship created (`piping`) between two real entities, appears in `listRelationshipsForEntity` | PASS |
| R002-4 | New relationship defaults to `validation_state: 'UNREVIEWED'` | PASS |
| R002-5 | Relationship referencing non-existent `target_entity_id` rejected with clear error (mirrors T006 intent) | PASS |
| R002-6 | Source registered with real SHA-256 checksum (64 hex chars, not a placeholder) | PASS |
| R002-7 | Evidence created referencing the registered source | PASS |
| R002-7b | Created evidence's `source_id` matches the registered source exactly | PASS |
| R002-8 | New evidence defaults to `validation_state: 'UNREVIEWED'` | PASS |
| R002-9 | Evidence referencing non-existent `source_id` rejected with clear error (mirrors T007 intent) | PASS |
| R002-10 | Relationship created with `evidence_id` link; ledger panel visibly shows the evidence-linked badge and evidence section | PASS |
| R002-11 | Relationship referencing non-existent `evidence_id` rejected with clear error | PASS |
| R002-12 | Relationship with an invalid `relationship_type` (bypassing the UI `<select>` directly via `db.js`) rejected with clear error | PASS |
| R002-13 | Zero console errors/exceptions accumulated across the entire run | PASS |

22/22 passed.

**Suite B (re-run + extended) — raw `file://` mode (no server)**
| ID | Check | Result |
|---|---|---|
| F1 | `file://` load with zero console errors | PASS |
| F2 | Entity created successfully under `file://` | PASS |
| F3 | Data persists across reload under `file://` | PASS |
| F4 | Relationship created successfully under `file://` (no server at all), defaults to `UNREVIEWED` | PASS |
| F5 | Zero console errors across the `file://` run | PASS |

5/5 passed.

**Total: 27/27 real browser checks passed.** This both re-confirms zero
regression to R1-001's original 11 checks (all their equivalents re-ran
here, A1–A8 plus F1–F3) and newly verifies every R1-002 acceptance
criterion from the task contract.

## Test evidence

Raw output, hosted suite (chrome 131.0.6778.204, headless, port 9224,
served via `python3 -m http.server 8899` from the literal `30_WEB_APP`
directory):

```
PASS — A1: console errors=0 []
PASS — A2: rows=1
PASS — A3: ["canonical_name","context","created_at","display_name","entity_id","entity_type","status","updated_at"]
PASS — A4: rows=1
PASS — A5: rows=1 toast="Duplicate identifier: [P&ID-TAG, PV-1042] already exists (ux_entity_identifier)"
PASS — A6: rows after reload=1
PASS — A7: dir=rtl
PASS — A8: registrations=1
PASS — R002-1: entity count=2
PASS — R002-2: relationship form present=true
PASS — R002-3: {"count":1,"type":"piping","validation_state":"UNREVIEWED"}
PASS — R002-4-default-validation-state: validation_state=UNREVIEWED
PASS — R002-5: {"rejected":true,"message":"Referential integrity violation: target_entity_id \"ent_does_not_exist_999\" does not exist"}
PASS — R002-6: {"count":1,"hasChecksum":true,"checksumLen":64}
PASS — R002-7: {"count":1,"source_id":"src_mtalozf0_1845835200219910323428546957501204209924","validation_state":"UNREVIEWED"}
PASS — R002-7b-source-linkage: evidence.source_id matches registered source
PASS — R002-8-default-validation-state: validation_state=UNREVIEWED
PASS — R002-9: {"rejected":true,"message":"Referential integrity violation: source_id \"src_does_not_exist_999\" does not exist"}
PASS — R002-10: {"evidence_id":"evd_mtaloznj_2131154012300191275913200633453000777200","panelShowsLinkedBadge":true,"panelShowsEvidenceSection":true}
PASS — R002-11: {"rejected":true,"message":"Referential integrity violation: evidence_id \"evd_does_not_exist_999\" does not exist"}
PASS — R002-12: {"rejected":true,"message":"Invalid relationship_type \"not_a_real_type\" (RELATIONSHIP_CONTRACT.md)"}
PASS — R002-13-no-console-errors: errors=0 []

=== SUMMARY ===
22/22 passed
ALL TESTS PASSED
```

Raw output, file:// suite (fresh Chrome profile, port 9223,
`--allow-file-access-from-files`, no HTTP server involved):

```
PASS — F1: console errors=0 []
PASS — F2: rows=1
PASS — F3: rows after reload=1
PASS — F4-relationship-under-file: {"relCreated":true,"validation_state":"UNREVIEWED"}
PASS — F5-no-console-errors: errors=0

=== SUMMARY (file://) ===
5/5 passed
ALL FILE:// TESTS PASSED
```

Syntax/format validation (re-run, unchanged method from R1-001):
```
$ node --check app.js && echo OK   → OK
$ node --check db.js && echo OK    → OK
$ node --check service-worker.js && echo OK → OK
$ python3 -m json.tool manifest.webmanifest > /dev/null && echo OK → OK
```

Byte-identity check confirming the files actually exercised by the browser
are exactly the files being delivered in this task (not a stale copy):
```
$ sha256sum pack/.../30_WEB_APP/{db.js,app.js,styles.css,index.html} \
    vs serve_root/30_WEB_APP/{...}
db.js:      match
app.js:     match
styles.css: match
index.html: match
```

## Decisions

- **Test driver substitution (Puppeteer → raw CDP):** R1-001 used
  Puppeteer/CDP; this sandbox had no network access to install the
  `puppeteer` npm package and no cached copy of the library, only a cached
  Chrome 131.0.6778.204 binary. Rather than skip real-browser testing or
  fall back to static analysis, a ~70-line dependency-free CDP client was
  written using Node 22's built-in `WebSocket` global, talking to the same
  Chrome binary directly. This is disclosed explicitly (not silently
  substituted) because the task contract and `BUILD_CONTRACT.md`'s
  "no success without actual output evidence" principle require knowing
  exactly what produced the evidence. The harness is saved under
  `00_CONTROL/tasks/R1-002-test-harness/` so it does not need to be
  rewritten for R1-003+ and so a reviewer can rerun it independently.
- **`evidence.source_id` stand-in for the missing import pipeline:** since
  R2 (workbook import) doesn't exist yet, `evidence` needs a real parent
  `source` row to satisfy referential integrity honestly. Added a minimal
  "register a source" form (source_type, source_name, source_version) that
  computes a genuine SHA-256 over the entered text via
  `crypto.subtle.digest`, rather than fabricating a placeholder checksum.
  This is scoped narrowly per the task contract's constraint 6 and is not
  intended to anticipate or redesign R2's real import/fingerprinting flow.
- **`relationship_type` restricted via `<select>`, but `db.js` also
  validates it:** the UI `<select>` cannot submit an invalid category, but
  `createRelationship` in `db.js` independently rejects any value outside
  `RELATIONSHIP_TYPES` — this matters because `db.js` is a reusable data
  layer other callers (future UI, and later the R2 import pipeline) will
  also invoke directly, not only through this form.
- **Evidence surfaced in the ledger panel only via relationship linkage,
  not as its own top-level entity-scoped list:** `evidence` records aren't
  inherently entity-scoped in the schema (they attach to a `relationship`
  via `evidence_id`, or could attach elsewhere later, e.g.
  `standard_mapping`). Showing "evidence linked to this entity's
  relationships" is what `RELATIONSHIP_CONTRACT.md`'s `evidence_id` field
  actually models; a separate unscoped global evidence browser is a
  reasonable future addition but out of this task's scope.

## Warnings

- The relationship UI only supports creating relationships **from** the
  currently selected entity (`source_entity_id` is fixed to it); creating
  one where the selected entity is only the `target` requires selecting
  the other entity first. This matches the schema (direction is a field
  on the relationship, not a UI restriction) but is a usability
  limitation worth revisiting once R3 (Explorer) exists.
- `evidence` has no delete/edit UI yet, matching `relationship` and
  `entity`'s existing R1-001 state — full CRUD (not just Create+Read) for
  all stores remains future work, not unique to this task.
- No automated test suite is committed inside the pack itself yet (same
  open item as R1-001) — this task's tests ran from the temporary external
  harness described above, now saved alongside the task record but not
  yet reachable from the phone itself. Still an open decision (see
  Unresolved issues).
- Performance with a large number of relationships/evidence records was
  not tested (out of scope for R1-002; belongs with T015/R9).

## Unresolved issues

- (Carried over from R1-001, unchanged) `file://` behavior verified only
  on Chromium; Firefox for Android / Samsung Internet still unverified.
- (Carried over from R1-001, unchanged) No backup/export mechanism exists
  yet; still in scope for a future `R14`/`13_BACKUP` task.
- (New, from this task) Whether to commit a lightweight in-repo test
  runner reachable from the phone itself (vs. continuing to rely on an
  external harness each session) is still an open decision — this task's
  raw-CDP harness is a candidate starting point for that but was not
  wired into `30_WEB_APP` itself, per the task contract's file scope
  (only `30_WEB_APP` application files were in scope for IMPLEMENT; the
  harness is a test tool, saved under `00_CONTROL/tasks/`, not part of the
  shipped application).
- `standard_mapping` and `change_event` stores still have no UI/CRUD
  (unchanged from R1-001 — in scope for R6 and R8 respectively).

## State changes

`PROJECT_STATE.md` updated: phase remains R1, "Next task" advanced past
R1-002. See file for exact wording.

## Recommended next task

R1 has now delivered its two planned vertical slices (R1-001: identity;
R1-002: relationship + evidence). Recommend closing R1 as substantively
complete for its originally-scoped traceability thesis and proceeding to
**R2 — Import + Diff** (workbook ingestion, preview/commit pipeline,
fingerprinting, idempotent re-import), per
`KIMA_EIS_PHASE_PROMPTS_R1_R15.md`'s R2 prompt. R2 will also retire the
narrow `registerSource` stand-in added here once a real import pipeline
can populate `source` records from actual ingested files.
