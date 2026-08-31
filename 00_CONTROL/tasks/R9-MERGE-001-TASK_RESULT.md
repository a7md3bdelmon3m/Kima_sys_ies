# TASK RESULT

Task ID: R9-MERGE-001
Status: FIXED — logic/regression-verified via Node, NOT real-browser-verified
(same standing limitation as M-18/M-19/H-05/H-06 — no Chromium binary, no
npm registry access, no general network egress in this sandbox, confirmed
directly before starting, not assumed from any prior session)

## Context

Two independent lines of work on R9 (topology graph view) existed at the
start of this task: an in-session draft (`R9-001`, simpler, non-interactive
Canvas rendering, its own `graph.js` fix already applied) and a separately
uploaded, more complete pack (`KIMA_EIS_V3_MERGED_2026-08-29.zip`,
containing a fully interactive pan/zoom/pinch/tap-to-navigate topology
renderer, plus its own fixes for M-19/M-20, and its own `R9-TASK_CONTRACT.md`
/`R9-TASK_RESULT.md`).

Rather than silently picking one or attempting an automatic merge, the two
were compared in detail (file sizes, category-naming fidelity to
`TOPOLOGY_RULES.md`, defect presence) and presented to the user, who chose
to adopt the uploaded pack as the project baseline and have this task fix
whatever defects the comparison had found in it.

## Comparison findings (informed the adoption decision)

| Aspect | In-session draft (R9-001) | Uploaded pack (R9) |
|---|---|---|
| Category naming | Used `dependency` instead of `logical` — a deviation from `TOPOLOGY_RULES.md`'s literal six category names | Matches `TOPOLOGY_RULES.md`'s six category names verbatim |
| `equipment_association` categorization | `physical` (risks implying a physical link `TOPOLOGY_RULES.md` warns against) | `logical` (more faithful to the rule's intent) |
| `graph.js#traverse` `relationshipTypes` defect | Fixed | Present (confirmed by direct reproduction — see below) |
| `window` resize-listener leak | Not applicable (design has no `window` listener) | Present (confirmed by direct simulation — see below) |
| `backward`/`reverse` form defect (M-19) | Logged only, not fixed (out of that task's declared scope) | Fixed |
| Pan/zoom/pinch/tap-to-navigate | None | Full implementation (Pointer Events, DPR-aware, hit-testing) |

Conclusion: the uploaded pack is materially more mature as a user-facing
feature and fixes an additional real defect (M-19) the draft had only
logged. It carried two real defects of its own, now fixed by this task —
see below — plus a third found during the same review (missing
service-worker cache entry).

## Defects found and fixed

### 1. `graph.js#traverse`'s `relationshipTypes` filter (same class as the
defect independently found and fixed in the R3-002/R9-001 line of work)

Confirmed present in the uploaded pack's `graph.js` (25 lines, unmodified
from pre-R9) by direct reproduction under Node:

```
rels = A-connection->B-piping->D-containment->E
traverse(rels, 'A', { relationshipTypes: ['containment'] })
=> nodes: ['A']   // E is unreachable despite a real containment edge existing
```

Root cause: the type filter was applied before deciding whether to
continue the BFS through an edge, so it silently blocked traversal past
the first non-matching edge rather than only filtering which edges/nodes
are reported in the result.

Fix: replaced the single `visited` Set with two — `walked` (gates BFS
continuation, applied regardless of type match) and `reported` (gates
inclusion in the returned `nodes`/`edges`, applied only to matching
edges). No caller-facing signature or default-behavior change.

Verification (executed, not just read): re-ran the full existing 6-case
direction-semantics regression plus 3 new cases reproducing and then
confirming the fix, against the actual edited file via a real ES-module
`import()` (not a retyped copy). All 9 cases PASS. Re-ran `impact.js`'s
`calculateImpact` (the only real production consumer of `traverse`)
against the same fixture: identical output before and after the fix,
confirming zero regression. Re-ran the pre-existing `decision.test.mjs`:
PASS.

### 2. `topology.js#createTopologyRenderer`'s `window` resize-listener leak

Confirmed by reading: `createTopologyRenderer` calls
`window.addEventListener('resize', resize)` unconditionally on every
invocation, with no corresponding `removeEventListener` anywhere. Since
`app.js` calls `createTopologyRenderer(topologyCanvas, ...)` fresh inside
`selectEntity` every time an entity with relationships is selected (each
selection rebuilds `panel.innerHTML`, destroying the previous `<canvas>`
and creating a new one), every entity selection left one more permanent
listener on `window`, each closing over an abandoned canvas/context pair
that `window` itself never releases.

Confirmed mechanically (no real DOM/`ResizeObserver` available in this
sandbox either) via a minimal `EventTarget`-shaped simulation: 5 simulated
entity selections produced 5 accumulated listeners, 0 ever removed —
matching the exact mechanism read in the source.

Fix: replaced the `window.addEventListener('resize', ...)` call with a
`ResizeObserver` scoped to the canvas element itself
(`new ResizeObserver(() => resize()).observe(canvas)`), plus an explicit
`destroy()` method added to the object `createTopologyRenderer` returns
(disconnects the observer). `app.js` now stores the active renderer in
`state.activeTopologyRenderer` and calls `.destroy()` on it — both when a
new renderer is about to be created for a newly-selected entity, and when
an entity with no relationships is selected (so no topology section is
rendered at all) — before discarding the reference. This is a second,
deterministic disposal layer on top of the `ResizeObserver` fix itself,
which alone is already sufficient: nothing external references the
observer or its canvas once the canvas is removed from the DOM, so both
become garbage-collectable normally with no per-call-site cleanup
required.

`ResizeObserver` browser support confirmed via web search before adopting
it (not assumed): supported unflagged in Chrome 64+, Firefox 69+, Safari
13.1+/13.4+, Edge 79+ since 2020 — no polyfill or CDN dependency needed,
consistent with this project's zero-CDN build contract.

Verification: `node --check` clean on `graph.js`, `topology.js`, `app.js`,
`service-worker.js` after all edits. The fix's *mechanism* (DOM-node
removal releases the observer/canvas with no lingering `window`
reference) was verified by code inspection and by consulting
`ResizeObserver`'s documented lifecycle semantics, not by an actual
heap-snapshot/GC run — that would require a real browser, unavailable in
this sandbox.

### 3. `topology.js` missing from `service-worker.js`'s offline cache list

Found while checking the PWA offline-cache manifest for consistency with
`app.js`'s new imports: `SHELL_ASSETS` in `service-worker.js` listed
`graph.js`, `impact.js`, `decision.js`, etc., but not `topology.js`,
despite `app.js` now importing it. Under an installed/offline PWA
(this app's primary intended usage mode per its own README), the
service worker would never cache `topology.js`, so the entire Topology
feature would fail to load once the app is used offline.

Fix: added `'./topology.js'` to `SHELL_ASSETS` and bumped `CACHE_NAME`
from `kima-eis-shell-v7` to `kima-eis-shell-v8` (required for previously
installed clients to actually pick up the new asset list — a same-named
cache is not re-populated).

Verification: `node --check` clean; manual inspection confirms the file
list now matches every local ES-module import `app.js` makes.

## Files changed

- `30_WEB_APP/graph.js` — `traverse`'s `relationshipTypes` handling
  (defect 1)
- `30_WEB_APP/topology.js` — `ResizeObserver` + `destroy()` (defect 2)
- `30_WEB_APP/app.js` — `state.activeTopologyRenderer` tracking and
  disposal calls (defect 2, second layer); `state` initialization updated
  to declare the new field explicitly
- `30_WEB_APP/service-worker.js` — `SHELL_ASSETS` + `CACHE_NAME` (defect 3)

No changes were made to `index.html`, `styles.css`, `db.js`, `impact.js`,
or `decision.js` — none were implicated by any of the three defects.

## Not verified (unchanged from R9-TASK_RESULT.md's own disclosure, still
true after this task's fixes)

- No real Canvas drawing, no real pointer/touch/wheel events, no real
  tap-to-navigate — read and traced, not executed in a browser.
- No confirmation of real layout dimensions
  (`canvas.getBoundingClientRect()`) in an actual rendered page.
- No full regression pass of R1-001 through R3-002 in a real browser.
- The `ResizeObserver` fix's actual firing behavior on a real resize event,
  and the `destroy()` call's actual effect on heap retention, are both
  unverified beyond code-level reasoning — this specifically requires a
  real browser and was not possible here either.

## Recommendation

Unchanged in substance from R9-TASK_RESULT.md and R3-002-TASK_RESULT.md:
real-browser verification remains the single outstanding gate before any
of R3-002/R9/this task's fixes can move from PARTIAL/FIXED-pending-
regression to PASS. When a browser becomes available, the same session
should cover, in one pass: the context-bar filter (R3-002), the topology
view's full interaction surface (pan/zoom/pinch/tap, depth/direction
controls, the type-filter now that it's fixed), the `backward`->`reverse`
form fix (M-19), the `openDb()` upgrade-handler fix (M-20), and this
task's three fixes — plus a full R1-001 baseline regression, since none of
these have been chained together in one real-browser run yet.

## State changes

Per `SYSTEM_OF_RECORD_MATRIX.md`'s own precedence rule (matrix first, then
propagate): `SYSTEM_OF_RECORD_MATRIX.md` (H-05 note + new M-22 row),
`V3_GAP_REGISTER.txt` (new M-22 row, three defects), and
`PROJECT_STATE.md` (this task's entry, adoption-of-uploaded-pack decision
recorded) all updated in this same session, in that order.
