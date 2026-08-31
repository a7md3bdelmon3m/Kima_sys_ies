# TASK CONTRACT

Task ID: R9-001
Source: 17_ROADMAP/ROADMAP.md ("R9 — Topology: Graph traversal and visual
topology"), 06_GRAPH/GRAPH_ENGINE_CONTRACT.md, 06_GRAPH/TOPOLOGY_RULES.md,
01_PRODUCT/MOBILE_UX_REQUIREMENTS.md (Entity screen Actions: "Topology"),
PROJECT_STATE.md's Next task entry (2026-08-29 update).

## Pre-implementation findings (must be read before reviewing the diff)

1. **PROJECT_STATE.md's "Completed tasks log" is stale relative to
   both DECISION_LOG.md and SYSTEM_OF_RECORD_MATRIX.md.** `graph.js`
   (buildGraph/traverse), `impact.js` (calculateImpact — this is
   actually R12's function, not R9's), and `decision.js` (Decision-
   Support Engine, with its own passing `decision.test.mjs`) already
   exist and are wired into `app.js`. **Correction to this contract's
   first draft**: these ARE already fully documented — in
   `DECISION_LOG.md`'s ADR-011 and, more authoritatively, as rows
   H-05/H-06/the decision-engine row in
   `00_CONTROL/SYSTEM_OF_RECORD_MATRIX.md` (M-12), which that file's own
   text says is the designated tiebreaker for exactly this kind of
   question. My own initial claim that "no task contract/result exists
   for whatever work produced them" and that this was undocumented
   anywhere was wrong — I had not yet read SYSTEM_OF_RECORD_MATRIX.md
   when I wrote it, and I am correcting it here rather than letting an
   inaccurate finding stand, per this project's own stated discipline
   around CR-07-class errors. The one real, narrower gap that remains:
   `PROJECT_STATE.md`'s own completed-tasks log — a different document
   from the matrix — still jumps from R3-001 straight to nothing, and
   should reference the matrix for this pre-existing work so a reader
   of PROJECT_STATE.md alone isn't misled the way I was. That narrower,
   corrected fix is what State changes below actually does — not a
   claim that M-12 itself is unmet, which would itself now be a false
   CR-07-class statement.
2. **`graph.js`'s `traverse` already implements part of R9's scope**
   (direct neighbors, bounded traversal) per `GRAPH_ENGINE_CONTRACT.md`.
   R9-001 therefore does NOT reimplement traversal from scratch — it
   fixes a defect found in it (below), adds the visualization layer,
   and wires it into the UI, none of which exist yet.
3. **Defect found in `graph.js#traverse` during pre-implementation
   verification** (reproduced under plain Node against a 5-relationship
   synthetic fixture covering all three direction semantics): the
   `relationshipTypes` option filters candidate edges at every BFS hop,
   so it silently blocks traversal *through* a non-matching edge rather
   than only filtering which edges are reported in the result. Example:
   with relationships A-connection->B-piping->D-containment->E and
   `relationshipTypes:['containment']`, `traverse(rels,'A',...)` returns
   only the start node — even though a containment edge (D->E) is
   genuinely reachable from A. This option is currently never invoked
   with a non-null value anywhere in the app (`impact.js` only passes
   it through as a default `null`), so it has zero live behavioral
   impact today — but R9 is the first feature whose purpose (letting a
   user filter a topology view to specific relationship types, per
   `TOPOLOGY_RULES.md`'s required distinction between relationship
   kinds) would actually trigger it. Left unfixed, R9 would ship a
   type-filter control that silently returns empty results for any
   multi-hop path with mixed relationship types — a defect that would
   look like "no relationships of that type exist" when they do.
   **This contract fixes it as part of R9's scope**, not deferred.
   Note: `SYSTEM_OF_RECORD_MATRIX.md` currently lists H-05 (graph
   traversal) as CLOSED based on "all 6 direction cases PASS" — that
   prior test suite did not include a `relationshipTypes` case, which
   is why this defect went undetected until R9-001 needed to actually
   exercise that option. Per the matrix's own stated rule ("any change
   to a row's STATE must be made here first"), this contract updates
   that row's verification note, not just PROJECT_STATE.md — see State
   changes.
4. **Sequencing check**: R9 in the roadmap follows R5 (Evidence+
   Provenance), R6 (DEXPI/CFIHOS), R7 (QA), R8 (History+Change) — none
   of which are recorded as complete. Verified this does NOT block
   R9-001: `GRAPH_ENGINE_CONTRACT.md`'s node/edge model is the existing
   canonical entity/relationship data (from R1-002), which R9 depends
   on and which already exists. R9-001's scope here does not require
   evidence, DEXPI/CFIHOS mapping, QA findings, or change-event data.
   If a future R9 increment adds those overlays to the topology view,
   that dependency will apply then, not now.

## Objective

Add a per-entity Topology view: an on-demand, bounded graph
visualization centered on the currently selected entity, reachable via
an action on the entity (per MOBILE_UX_REQUIREMENTS.md's Entity screen
Actions list, which names "Topology" alongside "Impact" as an
entity-scoped action, not a top-level nav item).

## Constraints

- Reuse `graph.js#traverse` as the sole traversal source (per
  `GRAPH_ENGINE_CONTRACT.md`'s rule: "Graph views are projections of
  canonical relationships, never independent copies"). No parallel
  traversal implementation in the rendering layer.
- Fix the `relationshipTypes` defect in `graph.js` itself (shared code),
  not by working around it in the new UI layer.
- Rendering: HTML5 Canvas per PROJECT_STATE.md's own phrasing ("graph
  traversal + Canvas rendering"). No new runtime dependency — this
  project's build contract (`15_BUILD/BUILD_CONTRACT.md`) targets
  zero-CDN, offline-first delivery; a charting/graph-drawing library
  would repeat the exact CDN-dependency problem R2-002 explicitly
  rejected from an external package. Canvas is native to the platform.
- Per `TOPOLOGY_RULES.md`: the view must visually distinguish
  relationship categories (at minimum: physical/piping connection vs.
  logical/reference/containment/etc.) — an edge must never visually
  imply a physical connection it doesn't have.
- Per `DESIGN_SYSTEM.md`/`MOBILE_UX_REQUIREMENTS.md`: RTL-safe, one-
  handed-operable where practical, no invented data (an entity with no
  relationships shows an explicit empty state, not a blank canvas that
  looks broken).
- Must not regress any existing verified behavior (R1-001 through
  R3-001's real-browser-verified checks; R3-002's logic-verified
  behavior). New code only adds a new panel/action and one new module;
  it must not modify `db.js#searchEntities`, `app.js`'s explorer state,
  or any IndexedDB schema.

## Acceptance criteria

1. Selecting an entity with at least one relationship (in either
   direction, any depth up to a bounded default) shows a "عرض
   الطوبولوجيا" (View Topology) action; activating it renders a Canvas
   graph centered on that entity using `traverse()`'s existing output,
   unmodified in shape.
2. An entity with zero relationships shows an explicit, non-blank empty
   state when Topology is opened, per "UI never invents missing data."
3. Nodes are labeled with the entity's display name (or canonical name
   if display name absent) and are visually distinguished from the
   center/selected node.
4. Edges are visually distinguished by relationship category
   (physical/piping/connection vs. logical/containment/reference/
   dependency/etc.), per `TOPOLOGY_RULES.md`.
5. A depth control (or fixed sensible default, documented) bounds how
   far the traversal extends, to avoid an unreadable/unusable graph on
   a small mobile screen for a densely connected entity.
6. The `relationshipTypes` defect in `graph.js#traverse` is fixed so
   that filtering by relationship type only affects which edges are
   reported/rendered, not which nodes are reachable through
   non-matching intermediate edges — verified by a Node-executable
   regression case reproducing the exact failing scenario found in
   pre-implementation verification, now passing.
7. No regression: the existing zero-argument/default-options behavior
   of `traverse()` and `calculateImpact()` (impact.js's only consumer)
   is unchanged for every call site that doesn't pass
   `relationshipTypes` — verified by re-running the existing
   `decision.test.mjs`-style Node harness pattern against `graph.js`
   and `impact.js` before and after the fix.
8. `PROJECT_STATE.md` and `DECISION_LOG.md` are updated to close the
   specific stale-documentation gap this contract's finding #1
   identified (graph.js/impact.js/decision.js existence), without
   claiming to resolve M-12 as a whole.

## Verification approach (given this session's confirmed sandbox limits)

Same class of constraint as R3-002 and ADR-011: no Chromium binary, no
npm registry access, no general network egress (independently
re-confirmed this session before starting R3-002's audit). Verification
here will therefore be, in order of strength actually available:
1. Node-executable regression tests against the exact, verbatim
   `graph.js`/`impact.js` function bodies (as done for the defect
   discovery above) — covers the traversal/filtering logic fully.
2. `node --check` syntax validation of every modified/new JS file.
3. Manual DOM/Canvas trace by reading (the same method used for
   R3-002's audit) for the parts that cannot be executed outside a
   browser (actual Canvas drawing calls, click/tap handling on the
   entity action button, RTL layout).
4. Explicit, non-silent flagging of what remains unverified (Canvas
   rendering correctness, touch interaction, real-browser regression)
   exactly as R3-002-TASK_RESULT.md and R3-002-MANUAL_AUDIT.md did —
   this task will NOT be reported as browser-verified PASS.

## State changes

Per `SYSTEM_OF_RECORD_MATRIX.md`'s own precedence rule, updates happen
in this order:
1. `SYSTEM_OF_RECORD_MATRIX.md` — H-05's row updated to note the
   `relationshipTypes` defect found and fixed by R9-001 (the prior
   "6 direction cases PASS" evidence didn't cover this option); new row
   added for R9-001's topology-view feature itself.
2. `V3_GAP_REGISTER.txt` — new row for this defect class (a type-filter
   silently narrowing traversal below what it should) plus its closure,
   so the register and the matrix don't re-diverge the way CR-07
   describes.
3. `PROJECT_STATE.md` — corrects the completed-tasks-log gap (referring
   readers to the matrix instead of silently omitting graph/impact/
   decision work) and adds this task's own entry. Status recorded as
   PARTIAL/logic-verified, consistent with every task since R3-002 in
   this sandbox class — not upgraded to browser-verified PASS.
