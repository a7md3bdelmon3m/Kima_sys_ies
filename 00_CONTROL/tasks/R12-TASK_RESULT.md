# TASK RESULT

Task ID: R12
Status: PARTIAL — implemented and logic-verified, NOT browser-verified
(same standard as R3-002/R9)

## Gap found between contract and existing code

`impact.js#calculateImpact` already existed (written before this
task, never wired to any UI or tested) but only implemented steps 1-2
of `IMPACT_ANALYSIS.md`'s 5-step contract — traversal and a bare
distance frontier (`{entity_id, depth, caused_by}`). Steps 3
(classify into identity/functional/topology/document/standard/QA), 4
(attach evidence), and 5 (produce a report, not just a node list) were
entirely absent. This task rewrote `impact.js` to close that gap
rather than wiring the incomplete version to the UI as-is.

## Design: relationship_type -> impact category mapping

`IMPACT_CATEGORY` (impact.js) maps all 12 of `db.js`'s
`RELATIONSHIP_TYPES` to one of the six categories `IMPACT_ANALYSIS.md`
defines, with documented rationale per entry (see the comment block in
`impact.js` itself for the full reasoning):

| relationship_type | category | why |
|---|---|---|
| containment, equipment_association | identity | affects what the contained/associated entity's place/identity is understood to be |
| dependency, function, process_flow, piping, instrumentation, connection | functional | describes actual plant operation |
| document_reference, drawing_reference | document | explicitly about documents |
| standard_mapping | standard | the DEXPI/CFIHOS mapping relationship per 08_STANDARDS/DEXPI_CFIHOS_POLICY.md |
| reference | QA | least semantic weight of the 12 types — flagged for review rather than asserted into a stronger category, per "impact is not a guess engine" |

`topology` is a real, defined category with **no relationship_type
mapped to it by default** — reasoned in-code as: topology-specific
impact is a graph-shape question (traversal depth/path itself), not a
per-edge attribute, so populating it would mean inventing a
classification the data doesn't actually assert. This is a design
judgment call, not something the terse `IMPACT_ANALYSIS.md` source
spells out explicitly — documented here and in-code so it can be
revisited if that reading turns out wrong.

This is a genuinely different classification from
`topology.js`'s `RELATIONSHIP_TYPE_CATEGORY` (which asks "is this edge
physical, per TOPOLOGY_RULES.md?"). Kept as two independent, separately
-documented mappings, per `RELATIONSHIP_CONTRACT.md`'s explicit
"topology and impact analysis must consume these same relationships"
— same 12 types, two purposes, not one derived from the other.

## Implementation

- `impact.js` rewritten: `calculateImpact` now returns `{changed,
  generated_at, impacted, categorySummary}`. Each `impacted` entry adds
  `relationship_type`, `category`, `evidence_id`, `validation_state` —
  all read directly off the connecting relationship closest to the
  changed entity (the edge that produced that entity's minimum
  traversal depth), never invented or aggregated across multiple paths.
  `categorySummary` is a plain count per category for the report
  summary line.
- Wired into `app.js`'s entity ledger panel as a new "تحليل الأثر
  (Impact)" section (placed after Graph/R9, before Evidence), with a
  depth selector and a "حساب الأثر" (Calculate Impact) button —
  deliberately button-triggered, not auto-run on every render, since
  impact analysis answers a specific "what if I change this?" question
  rather than being a passive display.
- New `impactReportHtml` (app.js) renders the report: entity label,
  depth, relationship type, category (Arabic label), validation-state
  badge (reusing the existing `validationBadgeClass` — no new badge
  logic), and either the evidence_id or an explicit "بدون دليل مرتبط"
  (no evidence linked) note — never silently omitted, so an entity with
  no evidence backing its impact path is visibly flagged as such, not
  indistinguishable from one that does.
- No new CSS — reuses `.chip`, `.chip--outline`, `.badge*`,
  `.identifier-list`, `.topology-legend` (all already defined,
  grep-confirmed).

## Verified by execution (logic only — see Not verified)

- `IMPACT_CATEGORY` checked against `db.js`'s real 12
  `RELATIONSHIP_TYPES`: all 12 covered, no silent gaps; confirmed only
  `identity`, `functional`, `document`, `standard`, `QA` are ever
  produced (topology correctly never populated, matching the
  documented design decision) and no invalid category value is ever
  used.
- `calculateImpact` (impact.js) extracted verbatim and run under Node
  (via `vm`, isolated context) against a realistic 8-edge K-103-style
  relationship set with mixed evidence/validation states: confirmed
  correct depth resolution (a 2-hop entity correctly excluded at
  depth-1 traversal, correctly included and depth-tagged at depth-2),
  correct category assignment for one relationship of each of the 6
  distinct categories actually produced, correct evidence surfacing
  (an entity connected via an edge with `evidence_id:'evd_001'`
  correctly shows that id; an entity connected via an edge with
  `evidence_id: null` correctly shows `null` — not invented, not
  silently dropped), and `categorySummary`'s counts confirmed to sum
  exactly to `impacted.length`.
- `impactReportHtml` (app.js) extracted verbatim and run under Node
  against realistic report data: confirmed entity labels, category
  labels (Arabic), and evidence notes all render correctly; confirmed
  the empty-report case renders the correct "no impact" message; and
  confirmed a malicious `canonical_name` (`<script>alert(1)</script>`,
  simulating what a real imported entity name could contain) is
  correctly HTML-escaped, not rendered raw.
- `node --check` passed on all 8 JS files in `30_WEB_APP/`; CSS
  brace-balanced; manifest valid JSON.
- `consistency_check.mjs` (adopted in the prior ADR-017 merge task,
  actually run again here, not assumed still-valid) correctly FAILED
  immediately after this task's edits (stale inventory hash/size for
  `app.js`/`impact.js`), then passed cleanly after
  `regen_inventory.mjs` was re-run — confirms the governance tooling
  is being used continuously, not just once at merge time.
- Element-ID consistency (`#impact-depth`, `#impact-run`,
  `#impact-report`) grep-confirmed: each appears exactly twice in
  `app.js`.

## Not verified

- No real browser was available this task (same constraint as
  R3-002/R9/ADR-017's merge). The button click handler, the actual DOM
  update (`el('#impact-report').innerHTML = ...`), and the interaction
  between this new section and the Graph/R9 section's shared
  `ensureAllRelationships()` fetch were traced by reading, not executed.
- No regression run against R1-001 through the ADR-017 merge's prior
  (browser- or logic-verified) behavior.
- The `IMPACT_CATEGORY` mapping's design judgment (especially
  `reference` -> `QA` and `topology` being intentionally unpopulated)
  reflects this task's best-effort reading of a genuinely terse source
  contract (`IMPACT_ANALYSIS.md` gives category names with zero
  elaboration) — not confirmed against any external authority, since
  none exists in this pack. Flagged as a judgment call, not a fact,
  so it can be revisited if the project's owner reads it differently.

## Recommendation

Before PASS: real-browser click-through of the Impact button across
several entities with varying relationship shapes (including one with
zero relationships, to confirm the section correctly doesn't render),
plus the same combined R1-001 through R12 regression pass already
recommended after the ADR-017 merge — this adds one more item to that
same pending pass rather than requiring a separate one.

## State changes

`PROJECT_STATE.md`, `DECISION_LOG.md`, `V3_GAP_REGISTER.txt`, and
`SYSTEM_OF_RECORD_MATRIX.md` updated in this same task, following
R3-002/R9's precedent.
