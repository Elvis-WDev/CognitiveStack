# Data Grid Pattern

## Purpose

Use one shared grid contract for operational collections. Users should not have to relearn
search, filters, columns, paging, or actions in each module.

A grid is the right surface when the user must compare, sort, filter, search, or analyze
several records. Data arriving in rows is not a reason to render a grid: one record's
attributes belong in a detail surface, and a handful of unrelated values belong in the
layout that fits them.

## Standard Anatomy

1. Compact title and description only when the `PageHeader` does not already supply them.
2. `SearchBox`.
3. Two or three high-value filters in the `FilterBar`; the rest go in an advanced filter
   dialog.
4. `Columnas` visibility control when the grid has optional columns.
5. Refresh, export, import, or create commands when applicable.
6. Sortable headers, rows, and one actions column.
7. `PagingBar` with result range, page size, page state, and first, previous, next, and
   last commands.

Keep controls in the same order, and use the same `PagingBar`, across the application.

Alignment must also hold from one module to the next: the same control heights, the same
cell padding, numbers aligned the same way, and the actions column at the same edge. A
user moving between modules should not notice the grid change.

## Shared Control Contract

`PagedGrid` is the shared `DataGrid` style plus `PagingBar`, bound to a list ViewModel. It
supports:

- typed column definitions bound to row DTOs, never to entities;
- debounced search through `SearchBox`;
- explicit option, date, and status filters;
- sorting through an allowlisted sort key that the query applies;
- column visibility;
- paging done in the query, with the same `PagingBar` for every grid;
- skeleton rows that preserve column geometry while loading;
- an informative `EmptyState`;
- permission-aware and state-aware row commands;
- selection only when a real bulk workflow exists;
- row and column virtualization with container recycling;
- horizontal scrolling inside the grid without shrinking text into illegibility.

Do not copy grid markup per module. Add configuration, a column template, or a shared style
instead.

## Paging And Queries

- Page in the query: `LIMIT` with a deterministic `ORDER BY`, and keyset paging on large
  tables. See `database.md`.
- Never load every row into an `ObservableCollection` and page or filter it in memory.
- Replace the page collection in one assignment instead of adding rows one at a time.
- Return the total count, or an explicit "more available" flag when counting is expensive.
- Cancel the previous query when filters, sort, or page change, and ignore late results.
- Run queries off the UI thread through an `AsyncRelayCommand`.

## Filters And Navigation State

- Filters live with the grid they affect.
- Use business labels and recognizable values.
- The list ViewModel owns filters, sort, page, scroll position, and selection;
  `INavigationService` restores them when the user returns from a record. See
  `cognitive-architecture.md`.
- Persist saved views or a user's default filters per user when operators repeat them daily.
- Reset to the first page when a filter changes.
- Debounce free-text search; do not debounce bounded options.
- Provide `Limpiar filtros` only when at least one filter is active.
- Show active advanced filters as removable labels when they are otherwise hidden.
- Validate structured filters such as dates, references, or ranges before querying.
- Use a `SegmentedControl` or toggle chips for short sets of values users switch often,
  instead of a multi-select dropdown.
- At narrow window widths, move secondary filters into a side panel with an apply command,
  and show the count of active filters on its trigger.

Avoid a separate oversized filter card above the grid, and never a row of ten dropdowns.

## Search

- Show `SearchBox` with a search icon, and a clear button once it holds a value.
- Debounce search, and show its loading state inside the grid, not over the window.
- `Ctrl+F` focuses the search box; `Esc` inside it clears the value.
- When nothing matches, say so and offer the next step: clear the search, remove a filter,
  or create the record.
- Accept a scanned barcode in the search box and open the record directly on an exact
  match. See `navigation-and-windows.md`.
- Highlight the matched text when it helps the user confirm the result.
- Offer recent searches only where users repeat the same lookups.
- When the application has many destinations and records, add a command palette to jump
  directly to a record or an action.

## Columns

Show the columns the primary decision needs, usually five to seven. Every extra column
costs scanning speed on every row the user reads. Keep the rest in the column menu or in
the detail surface, and let a saved preference remember what an individual user chose.

- Put the primary identity first.
- Combine closely related secondary information in one cell when it improves scanning.
- Right-align numbers and money, and format them for the user's culture.
- Keep status concise and semantic.
- Keep the actions column last, stable, and narrow.
- Hide raw IDs, database timestamps, file paths, and technical metadata by default.
- Do not allow column visibility to hide the only meaningful record identity or action access.

Column names, empty values, and units must be understandable without source-code knowledge.

## Visual Encoding

Let the user recognize a value without reading every cell.

- Use `StatusBadge` for every state column in the application.
- Encode meaning with an icon, a dot, or text in addition to color. Color alone is not a
  state, and Windows high-contrast themes remove most of it.
- Keep the same value in the same treatment everywhere; a state must not change color
  between modules.
- Add a badge, icon, or inline control only when it speeds a decision. Plain text is correct
  for the rest.
- Keep badge and control dimensions stable so rows do not change height.
- Use an inline control in a cell only for a reversible change with immediate feedback.
- Do not color whole rows for routine states; reserve that for a condition that needs
  attention now.

## Row Actions

- Use icons from the project icon family with an `AutomationProperties.Name` and an opaque,
  theme-aware tooltip.
- Edit, inspect, export, retry, activate, deactivate, archive, or delete only when valid
  for the row state and current permission: `CanExecute` reflects both.
- Disable with an explanation, shown through `ToolTipService.ShowOnDisabled`, when the
  action is visible but temporarily unavailable.
- Confirm destructive or irreversible actions. See `forms-and-workflows.md`.
- Open the record from the row itself, with double click and `Enter`, instead of spending an
  icon on it or keeping a permanent side panel beside the grid.
- Budget the row: at most three visible icon actions, each frequent and valid for most rows.
  Everything else goes in a `...` menu that also opens as the row context menu and with
  `Shift+F10`. Two or three unambiguous icons are faster than a menu; seven are a wall.

## Paging Bar

Every grid uses the same footer language and geometry:

- `Mostrando A-B de N`;
- `Filas` page-size selector;
- first, previous, current page indicator or page numbers, next, and last;
- disabled boundary commands;
- stable dimensions so controls do not shift.

The query returns deterministic ordering. Selected records for a real multi-page workflow
are kept by key in the ViewModel and persist across page changes until the workflow is
completed or explicitly cleared.

## Empty, Loading, And Error States

- Loading: render skeleton rows inside the existing grid frame.
- Empty collection: explain what can be created and offer the valid primary action.
- No filter results: state that filters returned no matches and offer to clear them.
- Error: keep the grid frame, show a concise `ErrorState` with a retry command.
- Partial data: render available columns and disclose which secondary data failed.

Do not replace the entire screen with a `LoadingIndicator` for routine grid refreshes.

## Window Size And Scaling

- Design for the minimum supported window size first. See `navigation-and-windows.md`.
- Never compress a wide grid into columns too narrow to read; scroll horizontally inside
  the grid, with the primary identity column frozen when it helps.
- Keep primary identity and actions reachable at every supported size.
- Let toolbar controls wrap into logical rows.
- Keep column headers visible while rows scroll, without covering page controls.
- Size rows in DIP so they scale with the display; on touch stations, rows and row commands
  meet the touch target size.
- Test long unbroken values at 100, 150, and 200 percent display scale.

## Grid Review

Reject the implementation when:

- the paging bar differs from neighboring modules;
- controls, alignment, or spacing drift from neighboring modules;
- filters sit in a detached decorative section;
- row actions lack tooltips, accessible names, or feedback;
- the grid exposes internal identifiers;
- the default columns exceed what the primary decision needs;
- a row carries more than three visible actions;
- a state is communicated only by color;
- changing pages loses intentional selection;
- all rows are loaded into memory, or virtualization is disabled;
- loading causes layout jumps;
- content overlaps or becomes unreachable at the minimum window size or 200 percent scale;
- summary cards repeat what the grid already communicates.
