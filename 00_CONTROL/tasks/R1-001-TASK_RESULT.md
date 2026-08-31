# TASK RESULT

Task ID: R1-001
Status: PASS

## Files created

- `30_WEB_APP/index.html`
- `30_WEB_APP/styles.css`
- `30_WEB_APP/app.js`
- `30_WEB_APP/db.js`
- `30_WEB_APP/manifest.webmanifest`
- `30_WEB_APP/service-worker.js`
- `30_WEB_APP/README.md`
- `00_CONTROL/ACR/ACR-001-web-platform-shift.md`
- `00_CONTROL/tasks/R1-001-TASK_CONTRACT.md`
- `00_CONTROL/tasks/R1-001-TASK_RESULT.md` (this file)
- `04_DATA/INDEXEDDB_SCHEMA_R0.md`

## Files modified

- `00_CONTROL/PROJECT_STATE.md` — platform section, completed list, next task, blocking issues
- `00_CONTROL/DECISION_LOG.md` — added ADR-007 (web platform shift)

## Tests executed

Real functional tests against an actual Chromium browser engine (headless
Chrome 131.0.6778.204, launched via Puppeteer/CDP) — not static analysis,
not assumption. Two suites, both against the literal `30_WEB_APP` files:

**Suite A — hosted mode (`http://localhost`, simulates PWA/served delivery)**
| ID | Check | Result |
|---|---|---|
| A1 | Page loads with zero console errors | PASS |
| A2 | Entity created via UI appears in list | PASS |
| A3 | Persisted entity record has all 8 `CANONICAL_ENTITY_CONTRACT.md` fields | PASS |
| A4 | Two `entity_identifier` rows persist for one entity | PASS |
| A5 | Duplicate `[namespace, identifier]` rejected via unique index, surfaced as non-crashing toast | PASS |
| A6 | Data persists after full page reload | PASS |
| A7 | Document renders with `dir="rtl"` | PASS |
| A8 | Service worker registers under http(s) | PASS |

**Suite B — raw `file://` mode (no server, direct phone file open)**
| ID | Check | Result |
|---|---|---|
| F1 | `file://` load with zero console errors | PASS |
| F2 | Entity created successfully under `file://` | PASS |
| F3 | Data persists across reload under `file://` | PASS |

All 11/11 checks passed. This empirically resolves the ACR-001 risk item
regarding `file://` IndexedDB restrictions, at least for Chromium (the
dominant Android browser engine); Firefox/Samsung Internet were not tested
in this pass and remain an open verification item (see Unresolved issues).

## Test evidence

Raw console output, both suites, this session:

```
=== TEST RESULTS ===
PASS — A1: page loaded with no console errors
PASS — A2: entity created and appears in list  [rows=1]
PASS — A3: entity record has all CANONICAL_ENTITY_CONTRACT fields  [["entity_id","entity_type","canonical_name","display_name","status","context","created_at","updated_at"]]
PASS — A4: two identifiers persisted for entity  [rows=2]
PASS — A5: duplicate identifier rejected, app did not crash  [rows=2 toast="...Duplicate identifier: [P&ID-TAG, PV-1042] already exists (ux_entity_identifier)"]
PASS — A6: data persists after reload  [rows=1]
PASS — A7: document direction is RTL  [dir=rtl]
PASS — A8: service worker registered under http(s)  [registrations=1]

ALL TESTS PASSED

=== FILE:// MODE TEST RESULTS ===
PASS — F1: file:// loads with no console errors
PASS — F2: entity created under file://  [rows=1]
PASS — F3: data persists after reload under file://  [rows=1]

ALL FILE:// TESTS PASSED
```

Manifest JSON validity and JS syntax also verified independently
(`python3 -m json.tool`, `node --check`) before functional testing.

## Decisions

- `entity.context` conflict (present in contract, absent from original SQL
  schema) resolved explicitly and documented in `INDEXEDDB_SCHEMA_R0.md`,
  not silently — added as an optional field on the `entity` store.
- Vanilla JS/ES modules chosen over any framework or bundler for R1, per
  the task contract's "no build toolchain at this stage" constraint.
- `21_ANDROID_STARTER` retained untouched, for reference only, per ACR-001.

## Warnings

- Referential integrity (foreign-key equivalents) is enforced entirely in
  `db.js` application code, not by the storage engine — this is weaker than
  the original SQLite/Room model and depends on disciplined code review as
  the schema grows (documented as a known risk in ACR-001).
- No backup/export mechanism exists yet — clearing browser site data for
  this origin destroys all local records. In scope for a future task under
  `13_BACKUP`.

## Unresolved issues

- `file://` behavior verified only on Chromium; not yet verified on
  Firefox for Android or Samsung Internet. Recommend verifying on the
  user's actual device/browser before relying on `file://` mode
  exclusively for daily use.
- Only `entity` and `entity_identifier` stores have UI/CRUD logic in R1.
  `relationship`, `evidence`, `standard_mapping`, `change_event`, and
  `source` stores exist in the IndexedDB schema (`db.js` `openDb`) but have
  no UI yet.
- No automated test suite is committed into the pack itself (tests were run
  from a temporary external harness); a follow-up task should decide
  whether/how to commit a lightweight in-repo test runner reachable from
  the phone itself.

## State changes

`PROJECT_STATE.md` updated: phase remains R1, "Next task" advanced past
R1-001 (bootstrap) toward the next vertical slice (relationship + evidence
CRUD). See file for exact wording.

## Recommended next task

R1-002 — Extend the web app with `relationship` and `evidence` object
stores' UI/CRUD, wiring `RELATIONSHIP_CONTRACT.md`'s `validation_state`
and `evidence_id` linkage so the traceability thesis (ledger panel) shows
real relationship/evidence data, not just identifiers.
