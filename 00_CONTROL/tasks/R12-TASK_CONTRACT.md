# TASK CONTRACT

Task ID: R12
Source: `17_ROADMAP/ROADMAP.md` ("R12 — Impact: Deterministic impact
analysis"), `10_CHANGE_IMPACT/IMPACT_ANALYSIS.md`,
`01_PRODUCT/MOBILE_UX_REQUIREMENTS.md` (entity screen action "Impact").

## Objective

Wire `impact.js#calculateImpact` (already written, never called from
any UI) into the entity ledger panel, and bring its output up to
`IMPACT_ANALYSIS.md`'s full 5-step contract — the existing function
only implemented steps 1-2 (traversal), not step 3 (classification),
step 4 (evidence), or step 5 (a structured report).

## Constraints

- "Impact is not a guess engine" (IMPACT_ANALYSIS.md) — every field in
  the report must be either read directly from stored data or a fixed,
  documented mapping; no severity/likelihood/narrative invented.
- Traversal must remain delegated to `graph.js#traverse` — no
  duplicate graph-walking logic (same constraint R9/topology.js held).
- The impact-category classification (identity/functional/topology/
  document/standard/QA) is a *different* classification from
  `topology.js`'s `RELATIONSHIP_TYPE_CATEGORY` (physical/logical/
  etc.) — per `03_DOMAIN/RELATIONSHIP_CONTRACT.md`'s "topology and
  impact analysis must consume these same relationships" (the same 12
  types, two independent classification schemes for two purposes).
  Must not conflate or derive one from the other.
- "Attach evidence" (step 4) means surfacing evidence already recorded
  on the connecting relationship (`RELATIONSHIP_CONTRACT.md`'s
  `evidence_id` field) — never generating new evidence.

## Acceptance criteria

1. Every one of `db.js`'s 12 `RELATIONSHIP_TYPES` is explicitly mapped
   to one of the 6 contract-defined categories (or 'uncategorized' if
   deliberately unmapped) — no silent default.
2. From an entity with relationships, running impact analysis produces
   a report listing every impacted entity with: depth, the relationship
   type/category connecting it, its evidence_id (or none, truthfully),
   and its validation_state.
3. No behavior change to entities with zero relationships (Impact
   section does not render, no error).
4. No change to the `entity`/`relationship`/`evidence` schema.
