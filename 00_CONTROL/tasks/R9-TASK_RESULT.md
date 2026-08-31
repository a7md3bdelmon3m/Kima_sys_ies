# TASK RESULT

Task ID: R9
Status: PARTIAL — implemented and logic-verified, NOT browser-verified
(same standard/limitation as R3-002 — see that task's precedent)

## Pre-existing bug found and fixed (blocking, unrelated to R9's own scope)

While reading the relationship-creation form to understand what data R9
would have available to draw, found: the form's direction `<select>`
offered `value="backward"`, but `db.js`'s `RELATIONSHIP_RULES`/
`assertRelationshipSemantics` and `graph.js`'s `buildGraph`/`traverse`
both only recognize `'forward'`/`'reverse'`/`'bidirectional'`. Confirmed
by extracting `assertRelationshipSemantics` verbatim and running it
under Node: submitting `direction: 'backward'` throws `Invalid
direction backward for <type>` unconditionally, for every relationship
type. This means **any relationship created via the "backward" option
in the running app was failing outright** — a live, user-facing defect
already shipped in prior deliveries (R1-002/R1-003), not introduced by
this task.

Fixed by changing the form option's value (and label) from `backward`
to `reverse` in `app.js` — no other file needed a change, since
`db.js`/`graph.js` already agreed with each other on `'reverse'`; only
the form was out of step. Re-verified with the same extracted-function
Node test: `direction: 'reverse'` now passes `assertRelationshipSemantics`
for every relationship type whose `allowedDirections` includes it.

This is recorded here rather than filed as a separate ADR because it
was found in the course of this task and is a one-line, unambiguous
fix — but it is flagged prominently since it is a correctness bug in
already-delivered functionality, not a topology design decision.

## R9 implementation

- New `30_WEB_APP/topology.js`: `createTopologyRenderer(canvas,
  onNodeClick)` — a Canvas-based graph renderer. Traversal is entirely
  delegated to an injected `traverseFn` (in practice, `graph.js#traverse`)
  — `topology.js` contains no graph-walking logic of its own, per this
  task's constraint.
- `RELATIONSHIP_TYPE_CATEGORY`: an explicit map, one entry per
  `db.js`'s 12 `RELATIONSHIP_TYPES` values, to one of six categories
  (`physical`, `process_flow`, `containment`, `instrumentation`,
  `logical`, `reference`) per `TOPOLOGY_RULES.md`. Only `piping` and
  `connection` map to `physical`; every other type renders with a
  visually distinct dash pattern/color so it cannot be mistaken for a
  real physical link.
- Deterministic radial layout (`layoutRadial`): start entity at center,
  each traversal depth on its own ring, evenly spaced — no physics
  simulation, no randomness, so the same graph always renders
  identically (matches this project's "UI never invents/varies data"
  posture and keeps the layout itself testable).
- Pan via pointer drag, zoom via wheel + two-finger pinch, tap-to-select
  a node (short tap, not a drag, to avoid misfiring during a pan).
- Wired into the ledger panel (`app.js`) as a new "الرسم البياني
  للعلاقات (Graph)" section, between Relationships and Evidence, with
  depth (1/2/3/5) and direction (both/out/in) controls. Only rendered
  when the entity has at least one relationship (criterion 5).
- `db.js` gained `listAllRelationships()` (list every relationship
  across all entities) — needed because `traverse` must walk beyond an
  entity's direct edges, and no existing function returned the full
  relationship set; `listRelationshipsForEntity` only returns one
  entity's direct edges. Added following the exact pattern of the
  existing `listEvidence`/`listSources` functions; exported from
  `KimaDB`'s public API alongside them.
- CSS added for `.topology-canvas` (touch-action: none — required
  since pan/pinch are handled manually and must not fight native
  scroll/zoom), `.topology-controls`, `.topology-legend`, using only
  CSS custom properties already defined in `styles.css` — no new
  design tokens invented.

## Verified by execution (logic only — see Not verified)

No real browser was available this task, same constraint as R3-002.
Verification here covers the pure-logic parts, extracted verbatim from
the actual modified files and run under Node (not reimplemented):

- `traverse` (graph.js, unmodified) run against a realistic 6-edge,
  multi-hop K-103-style relationship set: depth-2 traversal correctly
  reached a node 2 hops away; depth-1 traversal correctly excluded it.
  Both confirmed by inspecting the actual node/depth list returned.
- `layoutRadial` (topology.js) run against that same traversal result:
  produced one position per node (count matched), all positions
  distinct, start node correctly centered.
- `RELATIONSHIP_TYPE_CATEGORY` checked against `db.js`'s actual 12
  `RELATIONSHIP_TYPES` values: all 12 covered (no silent default used),
  and confirmed exactly `['connection','piping']` map to `'physical'`
  — matching `TOPOLOGY_RULES.md`'s requirement precisely.
- `assertRelationshipSemantics` (db.js, unmodified) re-run to confirm
  the `backward`→`reverse` form fix actually resolves the bug: before
  the fix, `direction:'backward'` threw for every type; the fixed form
  now only ever submits `'forward'`/`'reverse'`/`'bidirectional'`, all
  of which pass for at least one relationship type.
- `node --check` passed on every JS file in `30_WEB_APP/` (`app.js`,
  `db.js`, `topology.js`, `graph.js`, `impact.js`, `decision.js`,
  `import.js`, `service-worker.js`); `manifest.webmanifest` still valid
  JSON; CSS brace-balance checked (119 open, 119 close).
- Element-ID cross-references (`#topology-canvas`, `#topology-depth`,
  `#topology-direction`) grep-checked: each appears exactly twice in
  `app.js` — once in the HTML-generating template, once in the
  post-render query — consistent, no typo'd selector.

## Not verified

- **No real Canvas drawing, no real pointer/touch events, no real
  click-to-navigate.** The `pointerdown`/`pointermove`/`pointerup`
  handlers, the actual `ctx.arc`/`ctx.stroke` drawing calls, and
  hit-testing against real screen coordinates were written and traced
  by reading, not executed in a browser. This is the same gap R3-002
  carries, now also true for a materially more complex, more
  interaction-heavy feature — the risk of an unverified defect is
  correspondingly higher here than in R3-002.
- No confirmation that the ledger panel's actual `getBoundingClientRect()`
  returns non-zero dimensions in a real layout (assumed true based on
  reading the CSS/DOM structure — no `hidden`/`display:none` found on
  any ancestor — but not run).
- No regression pass against R1-001 through R3-002's previously
  verified/logic-verified behavior. The `backward`→`reverse` fix in
  particular touches a form that R1-002/R1-003 already covered in
  their own (real, browser-run) test suites; this task's fix was not
  re-run through those suites.
- `db.js`'s new `listAllRelationships` was checked only by reading, for
  consistency with `listEvidence`/`listSources`'s pattern — not
  executed against a real IndexedDB instance.

## Recommendation

Before this can move from PARTIAL to PASS: run a real browser session
(network permitting, to install Playwright/Chromium) covering — the
`backward`→`reverse` fix via the relationship-creation form itself; the
graph rendering and its depth/direction controls; pan, zoom (wheel and
pinch), and tap-to-navigate on the canvas; and a full regression pass
of R1-001 through R3-002.

## State changes

`PROJECT_STATE.md`, `DECISION_LOG.md`, and `V3_GAP_REGISTER.txt` to be
updated in this same task to reflect R9 as PARTIAL, and to record the
`backward`/`reverse` fix and its own gap for pending browser
verification, following R3-002's precedent (M-17).
