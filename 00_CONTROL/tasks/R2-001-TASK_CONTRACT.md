# TASK CONTRACT

Task ID: R2-001
Phase: R2
Title: CSV workbook import pipeline — fingerprint → parse → preview → commit, idempotent

## Objective

Implement `05_IMPORT/IMPORT_PIPELINE.md`'s full stage sequence
(fingerprint → parse → structural validation → header/schema recognition
→ normalization → candidate identity resolution → relationship extraction
→ evidence attachment → conflict report → preview → commit) inside
`30_WEB_APP`, entirely client-side, so a user can pick a local workbook
file on their phone and import entities (and, where the source expresses
them, identifiers/relationships) into the existing IndexedDB store, with
zero duplication on re-import of the same file.

## Inputs

- `00_CONTROL/MASTER_PROJECT_CONTRACT.md` (v3.0, 14 principles)
- `00_CONTROL/ACR/ACR-001-web-platform-shift.md`
- `00_CONTROL/PROJECT_STATE.md` (confirms R1-001 + R1-002 both PASS)
- `05_IMPORT/IMPORT_PIPELINE.md`, `05_IMPORT/IMPORT_CONFLICT_POLICY.md`
- `03_DOMAIN/CANONICAL_ENTITY_CONTRACT.md`, `RELATIONSHIP_CONTRACT.md`
- `04_DATA/INDEXEDDB_SCHEMA_R0.md` (`source` store fields, required
  `checksum_sha256`)
- `14_LLM_OS/roles/web_engineer.md`, `data_engineer.md`
- Existing code: `30_WEB_APP/db.js` (`registerSource`, `sha256Hex`,
  `createEntity`, `addEntityIdentifier`, `createRelationship` already
  exist from R1-001/R1-002 — this task wires a new pipeline on top of
  them, not a replacement)

## Allowed files

Only inside `30_WEB_APP/`:
- `import.js` (new — pipeline logic: fingerprint, CSV parse, structural
  validation, header recognition, normalization, candidate identity
  resolution, relationship extraction, conflict classification,
  preview/commit orchestration)
- `db.js` (extend only where a genuine gap exists — e.g. a bulk-commit
  transaction helper reusing existing per-record functions; no schema
  change)
- `app.js` (add an "Import" section: file picker, preview render,
  conflict report render, commit action)
- `index.html` (add the Import section's mount point)
- `styles.css` (extend, reusing existing patterns — chip/badge/table
  styling as needed for the conflict report and preview)
- `README.md` (update feature list)

## Constraints

1. **File format decision (explicit, not silent):** no Excel-parsing
   library is available locally in this environment and none can be
   fetched (no network egress; introducing an unvetted npm dependency
   without a build toolchain is out of scope per the task-contract
   pattern established since R1-001). Per the R2 phase prompt's own
   explicit fallback ("أو تحليل CSV يدوي إن كان المصدر بهذه الصيغة"),
   this task implements **manual CSV parsing** as the R2-001 source
   format. `.xlsx` support becomes a follow-up task (R2-002) once a
   vetted, locally-vendored parsing library is available — this is
   reported as a scoping decision in TASK_RESULT, not hidden.
2. Pipeline stage order is fixed and may not be reordered or collapsed:
   fingerprint → parse → structural validation → header/schema
   recognition → normalization → candidate identity resolution →
   relationship extraction → evidence attachment → conflict report →
   preview → commit.
3. **Fingerprint** = SHA-256 over the raw file bytes (via
   `crypto.subtle.digest`, same primitive already used in
   `db.js#sha256Hex`, generalized to accept an `ArrayBuffer` too, not
   only text) — computed before any parsing, so two byte-identical files
   always fingerprint identically regardless of parse outcome.
4. **Preview must not mutate the canonical database.** All pipeline
   stages up to and including conflict report run against an in-memory
   representation only.
5. **Commit** persists only rows with no BLOCKER-severity conflict, and
   does so inside IndexedDB transactions following the same
   referential-integrity discipline as R1-002 (parent existence checked
   within the same transaction before a child write commits).
6. **Idempotent re-import:** importing the same file (same fingerprint,
   same rows) a second time must not create duplicate entities. This
   task's identity-resolution stage keys off `entity_identifier`'s
   existing unique compound index (`[namespace, identifier]`, already
   enforced natively since R1-001) — a row whose identifier already
   exists resolves to the existing `entity_id` (update path) rather than
   creating a new entity (insert path). A `source` row is still recorded
   per import attempt (for provenance/audit of "when was this file last
   imported"), but re-importing an unchanged file must not duplicate any
   `entity`/`entity_identifier`/`relationship` record.
7. Row position (CSV line number) is never treated as identity, under
   any circumstance, including as a fallback.
8. Original source cell values are preserved verbatim in the evidence/
   audit trail for each committed record — no silent normalization that
   discards the original string.
9. Ambiguous matches (e.g., an identifier that looks similar to but does
   not exactly equal an existing one) are never auto-merged — only exact
   `[namespace, identifier]` matches resolve to an existing entity;
   anything else is a new entity candidate, full stop, per
   `CANONICAL_ENTITY_CONTRACT.md`'s Matching rule (no fuzzy merge).
10. Every conflict is classified BLOCKER/MAJOR/MINOR/INFO per
    `IMPORT_CONFLICT_POLICY.md`; no BLOCKER-classified row may be
    committed under any UI action.
11. No IndexedDB schema/version change. No build toolchain introduced.
12. Existing R1-001/R1-002 functionality (manual entity/identifier/
    relationship/evidence/source CRUD via the existing forms) must keep
    working unmodified — this task adds a new, separate Import section;
    it does not replace or gate the existing manual forms.

## Acceptance tests

Real functional tests against an actual Chromium browser (same
dependency-free raw-CDP harness pattern established in R1-002), covering
`16_TESTING/TEST_MATRIX.md`'s T001–T003 plus supporting checks:

- **T001 (source fingerprint):** importing a CSV file computes and
  stores a real SHA-256 fingerprint of the file's raw bytes; the same
  file's fingerprint is stable across repeated computation; a
  byte-different file produces a different fingerprint.
- **T002 (deterministic import):** importing a well-formed CSV with N
  data rows and no conflicts commits exactly N entities (or N
  entity+identifier pairs, as applicable) with all required
  `CANONICAL_ENTITY_CONTRACT.md` fields populated and source values
  preserved.
- **T003 (idempotent re-import):** importing the exact same file twice
  results in the same entity count as importing it once — zero
  duplicate entities, zero duplicate identifiers, verified by checking
  the `entity`/`entity_identifier` store counts before/after the second
  import.
- Conflict classification: a CSV containing a row whose identifier
  duplicates an in-file identifier (BLOCKER, per
  `IMPORT_CONFLICT_POLICY.md`'s first example) is reported as a conflict
  and is not committed, while non-conflicting rows in the same file
  still commit successfully.
- Preview mode performs zero IndexedDB writes (verified by store counts
  unchanged between "before preview" and "after preview, before
  commit").
- Row-position independence: reordering the rows of an otherwise
  identical CSV and re-importing does not change which entities are
  created/matched (proves row position isn't used as identity).

## Forbidden changes

- No edits to `21_ANDROID_STARTER/`, `15_BUILD/*`, role files, or any
  `00_CONTROL/` governance file during IMPLEMENT (only during STATE
  UPDATE, and only `PROJECT_STATE.md` + this task's own records).
- No redefinition of `CANONICAL_ENTITY_CONTRACT.md`,
  `RELATIONSHIP_CONTRACT.md`, or `IMPORT_CONFLICT_POLICY.md` semantics.
- No silent IndexedDB schema/version bump.
- No fuzzy/similarity-based identity matching introduced anywhere.

## Deliverables

- New `30_WEB_APP/import.js`.
- Modified `30_WEB_APP/db.js` (only if a genuine gap requires it),
  `app.js`, `index.html`, `styles.css`, `README.md`.
- `00_CONTROL/tasks/R2-001-TASK_CONTRACT.md` (this file).
- `00_CONTROL/tasks/R2-001-TASK_RESULT.md` with real test evidence.
- Test harness additions under `00_CONTROL/tasks/R2-001-test-harness/`
  (reusing `00_CONTROL/tasks/R1-002-test-harness/cdp.js` as-is).
- `00_CONTROL/PROJECT_STATE.md` updated (Next task advanced).
- `00_CONTROL/FILE_MANIFEST.md` regenerated.

## Dependencies

R1-001 (PASS) and R1-002 (PASS) — this task builds on `db.js`'s existing
entity/identifier/relationship/evidence/source functions and must not
regress any of them.
