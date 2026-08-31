# TASK CONTRACT

Task ID: R9
Source: `17_ROADMAP/ROADMAP.md` ("R9 — Topology: Graph traversal and
visual topology"), `01_PRODUCT/MOBILE_UX_REQUIREMENTS.md` (entity screen
tab "Graph", action "Topology"), `06_GRAPH/TOPOLOGY_RULES.md`.

## Objective

A Canvas-rendered, interactive (zoom/pan/tap) graph view of an entity's
relationships, reachable from the entity ledger panel, built on this
project's own `graph.js#traverse` — not a new traversal implementation.

## Constraints

- `graph.js`'s `buildGraph`/`traverse` are the only traversal logic;
  this task must not re-implement graph walking elsewhere.
- Per `TOPOLOGY_RULES.md`: "A graph edge must never imply a physical
  connection unless its relationship type explicitly says so." Every
  one of `db.js`'s `RELATIONSHIP_TYPES` must be explicitly categorized
  (logical / physical / process flow / containment / instrumentation /
  reference) and rendered so only the types that are genuinely
  physical (`piping`, `connection`) are visually indistinguishable from
  "a real connection."
- No external code — the R2-002 review flagged a rejected package's
  BFS/Canvas topology renderer as "architecturally sound, worth
  reimplementing fresh... not copied." This task writes fresh code
  against this project's actual contracts.
- Must not alter the `entity`/`relationship` schema.

## Acceptance criteria

1. From an entity with at least one relationship, a graph view renders
   showing that entity and its connections out to a configurable depth.
2. Edges are visually distinguishable by category per TOPOLOGY_RULES.md
   — physical vs. non-physical must not look the same.
3. Pan (drag), zoom (wheel + pinch), and tap-to-select-node all work.
4. Selecting a node in the graph navigates to that entity.
5. No behavior change to entities with zero relationships (graph
   section does not render, no error).
