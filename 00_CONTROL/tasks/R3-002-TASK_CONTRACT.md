# TASK CONTRACT

Task ID: R3-002
Source: R3-001-TASK_RESULT.md's Recommended next task (Warnings section)

## Objective

Promote the persistent context bar into a navigational filter: clicking
a Plant/Area/Unit/System breadcrumb segment narrows the entity explorer
to entities sharing that context prefix, with a visible way to clear the
filter.

## Constraints (carried over from R3-001-TASK_CONTRACT.md)

- Do not re-implement filtering logic outside `KimaDB.searchEntities` —
  any new filter dimension must be a new option on the existing
  function, applied through the existing `renderExplorer` call site.
- Do not invent a stricter context-hierarchy convention than what
  `parseContextHierarchy` already enforces (still free-text by
  contract; still no validation on level count).
- The context bar's existing behavior (showing the selected entity's
  breadcrumb, hidden when nothing is selected or context is
  unstructured) must be preserved, not replaced.

## Acceptance criteria

1. Clicking a breadcrumb segment sets the explorer filter to that
   segment's full prefix and the entity list updates to only entities
   matching that prefix.
2. An explicit clear control removes the filter and restores the full
   (or otherwise-filtered) list.
3. The active context filter is visible independently of which entity
   is currently selected.
4. No regression to existing search/filter/sort/pagination behavior for
   entities with no context set or with a context that doesn't contain
   "/".
5. No change to `entity`/`entity_identifier` schema or any other store.
