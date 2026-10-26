# UI Review Checklist

Use this checklist before completing any user-visible screen. The first section is checked
before implementation starts.

## Before You Build

- [ ] The screen brief is answered in writing: user, goal, primary decision, minimum information, single primary action.
- [ ] The screen is framed as a user job, not as an entity page.
- [ ] Information is tiered P0-P3, and nothing from P3 sits in the initial view.
- [ ] Nothing is planned for the screen only because a query or DTO returns it.
- [ ] The action budget holds: one primary action, one or two visible secondary ones, the rest disclosed.

## Task And Hierarchy

- [ ] The primary user, task, and action are clear.
- [ ] Orientation lands in about two seconds at the minimum window size: context, title, state, leading information, primary action.
- [ ] Exactly one action carries primary weight, and its copy names the action and its object.
- [ ] The chosen surface matches the workflow: grid, dialog, side panel, detail, dashboard, full screen, or separate window.
- [ ] Sections that own records, actions, or permissions are `SideNav` submodules, not tabs.
- [ ] Everything the module offers works end to end: list, create, edit, detail, and removal.
- [ ] Screen title, module title, description, and summaries are not duplicated.
- [ ] Filters and actions are attached to the content they affect.
- [ ] No card, panel, KPI, helper text, or decorative element exists without a user benefit.
- [ ] The screen remains understandable without implementation knowledge.

## Data And Language

- [ ] Labels use business language.
- [ ] Raw IDs, enum tokens, provider names, file paths, session internals, and secret references are hidden.
- [ ] Relationships use `LookupBox` or selection dialogs, not free-text IDs.
- [ ] Generated values are not requested from the user.
- [ ] Numbers, money, dates, times, percentages, and empty values are formatted consistently for the user's culture.
- [ ] Copy addresses the user directly, with no third-person narration, internal process detail, or specification references.

## Foundations

- [ ] Spacing, radius, type, color, and duration values come from the documented resource dictionaries.
- [ ] Grouping is expressed with space and alignment before borders or cards.
- [ ] Accent color stays within its budget, and no state relies on color alone.
- [ ] Every interactive element covers default, hover, keyboard focus, pressed, disabled, and loading.
- [ ] Feedback appears within about 100 ms, and every animation explains a change.

## Component Selection

- [ ] Each control was chosen from the user's intent, not mapped from the data type.
- [ ] Choices that fit on screen are visible, large sets are searchable, and no dropdown holds only two options.
- [ ] Exact values are typed, approximate values may slide, and small quantities step.
- [ ] `ToggleSwitch` applies immediately, and checkboxes wait for submit.
- [ ] Action levels use consistent button styles, and destructive actions are visually distinct.
- [ ] Custom controls are at least as accessible as the built-in WPF control they replace.

## Grids

- [ ] `PagedGrid` and `PagingBar` are used.
- [ ] Paging happens in the query; no screen loads every row into memory.
- [ ] Row and column virtualization stay enabled.
- [ ] Default columns match what the primary decision needs.
- [ ] The row opens the record with double click and `Enter`, and visible row actions stay within budget.
- [ ] Search, filters, sorting, columns, and row actions follow the application contract.
- [ ] Ordering is deterministic and the paging bar matches every other module.
- [ ] Control order, alignment, and spacing match the other modules.
- [ ] States use `StatusBadge` with an icon or dot, never color alone.
- [ ] Loading, empty collection, no matches, partial data, and error states are covered.
- [ ] Row commands are valid for permission and record state through `CanExecute`.
- [ ] Icon actions have opaque, theme-aware tooltips and automation names.
- [ ] Long values and narrow windows remain usable through horizontal scrolling inside the grid.

## Forms And Dialogs

- [ ] The form ViewModel derives from `ObservableValidator`, and the use case re-validates every invariant.
- [ ] Fields use `FormField`: label, required state, helper text, and inline error are associated.
- [ ] Conditional fields hide, clear, and validate correctly.
- [ ] Sensitive values remain masked and are never returned in plaintext.
- [ ] Duplicate submission is prevented by the running command's `CanExecute`.
- [ ] Failed submission preserves user input and focuses the first invalid field.
- [ ] Dialogs have an owner, take focus, return it on close, cancel with `Esc`, and submit with `Enter` unless destructive.
- [ ] A complex or high-risk workflow uses a full screen when a dialog would be cramped.
- [ ] Create and edit use a dialog, or the change's design records why the full screen was needed.
- [ ] Irreversible deletion requires `TypedConfirmDialog`; reversible actions take one step with undo.
- [ ] `FileField` handles pick, replacement, rename, and removal with per-file state.

## Feedback And State

- [ ] Every command has pending, success, and error feedback.
- [ ] Every command that changes data runs through `IFeedbackService` instead of ad hoc handling.
- [ ] No asynchronous path ends without a handled outcome.
- [ ] Routine success and error use the shared `ToastHost` notification.
- [ ] Persistent banners are reserved for persistent conditions.
- [ ] Notifications are not clipped and remain readable in every theme and at the minimum window size.
- [ ] Exceptions are sanitized; no exception message reaches the screen, and diagnostics stay in logs.
- [ ] Durable background work survives closing a dialog, closing the window, and restarting the application when required.
- [ ] Stale queries cannot overwrite newer scope or filter state.
- [ ] The UI thread never blocks on disk, database, or network work.

## Navigation And Windows

- [ ] Navigation has no more than two visible levels.
- [ ] Destinations and commands are permission-aware, and the use case still enforces each permission.
- [ ] Reordered navigation persists per user and offers a keyboard alternative.
- [ ] Only the selected destination has the solid active state.
- [ ] Chevrons appear only on expandable parents.
- [ ] Returning to a list restores filters, sort, page, scroll position, and selection.
- [ ] Separate windows are justified, owned, and never open off screen.
- [ ] `PageHeader` controls are stable, borderless where appropriate, and meet the target size.
- [ ] No overlap occurs at the minimum window size or at 100, 125, 150, and 200 percent display scale.
- [ ] Moving the window between monitors with different scales keeps text and icons sharp.
- [ ] Long text wraps or trims predictably, with the full value available.

## Keyboard And Input

- [ ] Every command works from the keyboard, with a logical tab order and visible focus.
- [ ] Access keys and shortcuts are unique, documented, and shown in tooltips or menus.
- [ ] Focus lands on the first useful control on open and returns to the invoking control on close.
- [ ] Touch targets are about 40 DIP or more on touch stations, and nothing depends on hover.
- [ ] Barcode scans land in the scan target, never trigger an unintended default button, and keep focus ready for the next scan.

## Performance

- [ ] Lists are paged in the query and virtualized; nothing unbounded feeds the screen.
- [ ] Images are sized for display and decoded at that size.
- [ ] Heavy, rarely used screens and their data load on demand, not at startup.
- [ ] No new N+1 query pattern or avoidable sequence of dependent queries.

## Theme And Accessibility

- [ ] Light, dark, and Windows high-contrast themes cover screens, dialogs, popups, tooltips, notifications, inputs, grids, and charts.
- [ ] The Windows show animations setting disables nonessential animation.
- [ ] Automation names, labels, and help text are set; Narrator reads the screen sensibly.
- [ ] Status is not communicated only through color.
- [ ] Contrast meets 4.5:1 for body text and 3:1 for large text and meaningful icons.

## Polish Pass

Once the feature works, review it as a finished product rather than a working prototype. A
feature is not complete when only its happy path looks acceptable.

- [ ] Spacing, alignment, and typography follow the scales, and related elements are grouped.
- [ ] Hover, pressed, focus, selected, and disabled states are present and distinct.
- [ ] Loading, empty, error, and success states look as deliberate as the default state.

## Screen Audit

Answer these by looking at the finished screen, not at the code.

- [ ] **Two seconds**: where am I, what is this, and what is the main action?
- [ ] **Visual anxiety**: after five seconds, do the eyes know where to start, or does everything compete for attention?
- [ ] **Elimination**: for each element, what breaks if it leaves the initial view? Remove or disclose whatever answers "nothing".
- [ ] **New user**: could someone who never saw the code or the database tell what this does, where they are, and what happens next?
- [ ] **Recurring user**: can a daily user filter, act, repeat, and keep context without unnecessary steps or the mouse?
- [ ] **Task cost**: decisions, clicks, keystrokes, context switches, and remembered values are counted, and each one is justified.
- [ ] **Consistency**: same state, same badge; same action, same control; same kind of detail, same pattern; same hierarchy, same type scale.
- [ ] **Restraint**: was any complexity added that does not improve the experience? If so, simplify.

A screen that makes the user feel they must understand everything before acting has failed,
even when every requirement is implemented.

## Verification Evidence

- [ ] Relevant unit and ViewModel tests pass.
- [ ] The critical flow is covered by an integration test against a real SQLite database.
- [ ] Screenshots at the minimum window size and at 100 and 200 percent scale were inspected when layout changed.
- [ ] The debug output shows no new binding errors or unhandled exceptions.
- [ ] `dotnet build` with analyzers and `dotnet test` pass.
- [ ] Create, edit, detail, and removal were exercised against real use cases and data, not mocks.
- [ ] The screen was smoke-tested under Wine during development and accepted on real Windows before release.
- [ ] Each requested criterion is reported as met or as a declared exception with its reason.
