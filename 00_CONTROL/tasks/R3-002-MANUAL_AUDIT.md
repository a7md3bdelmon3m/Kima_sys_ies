# MANUAL LOGIC AUDIT — R3-002

Auditor: Claude (session-independent review, no code changes made)
Date: 2026-08-29
Scope: Full manual trace of R3-002's acceptance criteria against the
current `30_WEB_APP` source, substituting for real-browser click-through
verification, which this session's sandbox was independently confirmed
to be unable to run (no Chromium binary present; `npm` registry access
returns 403; general network egress returns `host_not_allowed`). This is
a fresh, independent confirmation of the same environment class the
project's INT-001/INT-002/R3-002 sessions already hit — not assumed
from their prior reports.

## Method

Every function touched by R3-002 was extracted verbatim (via `view`/
`grep -n`, never retyped from memory) from the real, current `db.js`
and `app.js`, then traced by hand against each of R3-002-TASK_CONTRACT
.md's five acceptance criteria. Where hand-tracing left genuine doubt
(the HTML-attribute/JSON round-trip for breadcrumb prefixes), the exact
`escapeHtml` function was extracted verbatim and executed under plain
Node.js against adversarial inputs (ampersands, quotes, apostrophes,
angle brackets) with a manual HTML-entity decoder simulating what a
browser does when reading the attribute back via `.dataset` — not
assumed safe by inspection alone.

`searchEntities`/`parseContextHierarchy` were also re-executed verbatim
under Node against a harder synthetic entity set than R3-002's own
7-case check used, specifically adding: an entity with no `context`
field at all (`undefined`), an entity with `context: ''`, a
single-level context with no `/` at all, case-mismatched context
strings, and whitespace padding around `/` separators.

## Findings

1. **Acceptance criterion 1 (click narrows list to prefix)** — logic
   confirmed correct by trace and by executing the actual filter
   predicate: `contextLevels` prefix-matches
   `parseContextHierarchy(e.context).levels` exactly as designed, is a
   true no-op when empty/omitted (verified against both `[]` and
   `undefined`), and correctly returns zero matches for a filter deeper
   than any real entity's context. No divergence from the task result's
   claims found.

2. **Acceptance criterion 2 (explicit clear control)** — confirmed:
   `#explorer-context-filter-clear`'s click handler sets
   `contextLevels = []` and calls `renderExplorer()`. The filter
   indicator host is rebuilt fresh (`innerHTML` replaced) on every
   `renderExplorer` call, so there is no listener-accumulation risk
   across repeated renders (each render creates a brand-new button
   node with its own single listener; nothing is ever attached to a
   node that persists across renders).

3. **Acceptance criterion 3 (filter independent of selection)** —
   confirmed by exhaustive grep: `state.explorer.contextLevels` is
   mutated in exactly two places (the breadcrumb click handler and the
   clear button). `selectEntity` never touches it. `renderExplorer`
   calls `renderContextFilterIndicator()` unconditionally as its first
   statement, before either of its own early-return paths (empty
   entity set; zero search results) — so the active-filter indicator
   stays correctly visible even when the filtered list is empty, which
   is what "visible independently of results" requires.

4. **Acceptance criterion 4 (no regression for no-context/unstructured
   entities)** — confirmed, including edge cases not in R3-002's own
   7-case check: entities with `context` entirely absent, `null`, or
   `''` all resolve to zero parsed levels and correctly never match any
   active `contextLevels` filter, without throwing. A single-level
   context with no `/` (e.g. `"Plant A"`) also correctly produces zero
   levels (consistent with R3-001's own established convention), so it
   is correctly excluded from a `["Plant A"]` filter — this is
   expected/contractual behavior, not a bug, but worth stating
   explicitly since it may look surprising to a user who typed a
   single-segment context expecting it to be filterable.

5. **Acceptance criterion 5 (no schema change)** — confirmed by
   inspection: `contextLevels` is a `searchEntities` query-option field
   and an in-memory `state.explorer` field only; no IndexedDB
   store/index/version change appears anywhere in the diff.

6. **HTML-attribute/JSON round-trip for `data-context-prefix`** —
   specifically re-verified under adversarial inputs (values containing
   `&`, `"`, `'`, `<`, `>`) since this was the single most
   likely-looking failure point on inspection. Confirmed safe: the
   attribute is single-quote-delimited, `escapeHtml` escapes double
   quotes (harmless inside a single-quoted attribute) and all other
   HTML-special characters, and the browser's automatic entity-decoding
   on `.dataset` access correctly reconstructs the original JSON string
   for `JSON.parse` in every tested case. No defect found here — an
   initial suspicion was raised and then disproven by direct
   simulation, not left as an assumption in either direction.

## Observation (not a defect, flagged for awareness)

Context-level matching in `searchEntities` is case-sensitive and exact
per level (matching `parseContextHierarchy`'s existing no-normalization
contract), whereas the free-text `query` search is case-insensitive.
Real workbook data observed in ADR-013 already has inconsistent
capitalization; a breadcrumb generated from one entity's context (e.g.
`"Plant A"`) will not match another entity whose context was entered as
`"plant a"`. This is contractually consistent with existing R3-001
behavior (not a new regression introduced by R3-002) and is not a
required fix under this task's acceptance criteria, but should be kept
in mind if user reports describe the drill-down filter as "missing"
entities that visually appear to share the same context.

## What this audit does NOT establish

This is a static/manual logic audit, not execution in a real DOM. It
does not and cannot confirm:
- That the actual `click` event fires and propagates correctly on the
  real rendered `<button>` elements in a live browser.
- Any layout/CSS rendering correctness (visual appearance, RTL
  behavior, focus order) beyond confirming referenced classes/variables
  resolve to real definitions.
- Behavior under `file://` origin specifically (already separately
  flagged as untouched/unverified in the original R3-002 result, for
  unrelated ES-module reasons per ADR-009).
- Absence of any interaction with browser extensions, real user timing
  (double-clicks, rapid re-clicks during the 150ms search debounce
  window), or actual regression of R1-001 through R3-001's real-browser
  checks — those require actual execution, which remains unavailable in
  every sandbox this project has had access to so far, this one
  included.

## Conclusion

No functional defect was found in R3-002's implementation across
manual tracing of all five acceptance criteria plus one adversarial
round-trip simulation. This raises confidence beyond the original
logic-only pass, but it is still not equivalent to real-browser
verification and must not be recorded as such. R3-002 remains PARTIAL
— its status should read "logic-verified twice, independently, still
pending real-browser evidence" rather than being promoted to PASS on
the strength of this audit alone.

## Recommendation

Unchanged from the original R3-002-TASK_RESULT.md: real-browser
click-through and full regression are still required to close R3-002
as PASS. Until a browser becomes available in some session, R9
(Topology) may proceed in parallel, since this audit found no
functional coupling between R3-002's explorer-filter change and R9's
planned graph-traversal/Canvas work.
