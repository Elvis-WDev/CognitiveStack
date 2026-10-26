---
name: implement-wpf-screen
description: Build or refactor operational WPF screens such as CRUD modules, grids, forms, dialogs, dashboards, point-of-sale screens, navigation, and windows. Use when a XAML or ViewModel task must follow the repository's MVVM rules and shared controls, reduce visual fatigue, hide technical details, and ship with complete interaction, keyboard, scaling, and theme states.
---

# Implement WPF Screen

## Load Context

Read:

- `AGENTS.md` and the nearest scoped instructions.
- The OpenSpec change being implemented: its specs, design, and tasks.
- `docs/architecture/wpf-mvvm.md`.
- `docs/architecture/cognitive-architecture.md`.
- `docs/architecture/design-foundations.md`.
- `docs/architecture/component-selection.md`.
- Pattern docs required by the task:
  - `docs/architecture/component-system.md`.
  - `docs/architecture/interface-design.md`.
  - `docs/architecture/data-tables.md`.
  - `docs/architecture/forms-and-workflows.md`.
  - `docs/architecture/feedback-and-states.md`.
  - `docs/architecture/navigation-and-windows.md`.
- `docs/quality/ui-checklist.md`.

## Workflow

1. Inspect the existing screen, shared controls and styles, tokens, use-case contracts, permissions, and neighboring modules.
2. Answer the screen brief in writing: user, goal, primary decision, minimum information, single primary action. Do not implement without it.
3. Tier the information P0-P3, choose the smallest suitable surface, and report diagnosis, solution architecture, technical implementation, and verification before coding.
4. Choose each control from the user's intent with the decision framework in `docs/architecture/component-selection.md`, then check the choice against both primitive and novelty bias.
5. Remove duplicated headings, summaries, cards, helper text, technical fields, and persistent feedback that do not help the task.
6. Reuse shared controls and styles before adding new ones. Extend the shared contract when behavior belongs across modules.
7. Put state and commands in the ViewModel, with no WPF types; keep code-behind to purely visual behavior.
8. Implement every relevant state: loading, empty, filtered-empty, partial, pending, success, validation, permission, conflict, not found, and unexpected failure, plus the interactive states of each control.
9. Keep internal identifiers in values only. Render business labels and generated references.
10. Make commands and destinations permission-aware in the ViewModel, and rely on the use case as the authority.
11. Exercise keyboard-only operation, access keys, shortcuts, focus return, touch and barcode input when supported, real data volumes, long content, the minimum window size, 100 to 200 percent scale, light, dark, and high-contrast themes, animations off, repeated commands, slow operations, cancellation, and closing the window mid-operation.
12. Run project verification, complete the UI checklist including the polish pass and the screen audit, and report every requested criterion as met or as a declared exception with its reason.
13. Update architecture or feature docs when the workflow or shared control contract changes.

## Component Decisions

- Use `PagedGrid` for operational collections, paged in the query and virtualized.
- Use a form ViewModel derived from `ObservableValidator`, with `FormField` for every field.
- Use `SegmentedControl`, `SelectionCards`, or toggle chips when the options fit on screen, and `LookupBox` for large sets.
- Use `ToggleSwitch` for settings that apply immediately and checkboxes for values submitted with a form.
- Use sliders only for approximate values in a known range, `NumericStepper` for small quantities, `MoneyBox` for amounts, and typed inputs for exact values.
- Use built-in WPF controls with the project styles and the project icon family.
- Use a `ToastHost` notification through `IFeedbackService` for routine action results.
- Use inline field errors for correctable input.
- Use a dialog through `IDialogService` for short create and edit.
- Use a full screen in the shell for long, high-risk, multi-section workflows.
- Use `LookupBox` or selection dialogs with `PagedGrid` for relationships.
- Use persisted pending operations for work that must survive closing a dialog, closing the window, or restarting the application.
- Use `SideNav` submodules instead of tabs when a section owns its own records, actions, or permission.
- Use `StatusBadge`, with icon or dot beside the color, for state columns.
- Use `TypedConfirmDialog` for irreversible actions, and a single step with undo from the notification for reversible ones.
- Use `FileField` with `IFilePickerService` for picking, replacement, rename, and removal.

## Guardrails

- Do not implement a screen whose brief is unanswered.
- Do not map data types to primitive inputs without evaluating the user's intent.
- Do not replace a built-in control with a custom one that is less accessible.
- Do not derive a screen from the schema, and do not display a value only because a query returns it.
- Do not ship more than one primary action, or a wall of simultaneous secondary actions.
- Do not introduce spacing, color, radius, or duration values outside the documented resource dictionaries.
- Do not ship a control without use-case support, or a screen that runs on mocked data.
- Do not expose raw IDs, framework names, provider internals, file paths, secrets, exception messages, or technical metadata without a documented operational need.
- Do not run SQL, file, network, or device work from a view or ViewModel, or on the UI thread.
- Do not load every row into an `ObservableCollection` to page or filter it in memory.
- Do not create a third navigation level.
- Do not split a module into tabbed screens.
- Do not ship a command that changes data without pending, success, and error feedback.
- Do not narrate the system in the third person or cite internal specifications in visible copy.
- Do not invent module-specific paging, notification, tooltip, date input, dialog, or grid behavior.
- Do not nest cards or use decorative panels as screen structure.
- Do not add KPI cards that repeat grid information or do not change a decision.
- Do not leave routine success or loading banners on the screen.
- Do not adopt a third-party control suite without an ADR, token mapping, and keyboard and screen-reader checks.
- Do not mark complete with overlap, clipped controls, unreadable tooltips, missing states, binding errors, or theme regressions.
