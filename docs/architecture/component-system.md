# Shared Control Contracts

## Purpose

The exact names may follow the codebase, but each application provides one shared
implementation for each recurring operational interaction. Do not create a new variant in
each module.

## Foundation

Prefer built-in WPF controls restyled through the project resource dictionaries and
semantic tokens. Use the one icon family, stored as `Geometry` resources, instead of
hand-drawn paths when a suitable icon exists. Treat third-party control suites and sample
code as source to review and adapt, not as an independent design system; adopting a suite
is an ADR decision, as `wpf-mvvm.md` states.

Prefer, in order: a style, a `ControlTemplate` or `DataTemplate`, an attached property or
behavior, a custom control with a themed template, and only then a `UserControl`.

## System Layers

Keep one direction of dependency, from generic to specific:

```text
built-in controls -> shared styles and controls -> domain views -> features -> screens
```

| Layer            | Knows about                                | Example                  |
| ---------------- | ------------------------------------------ | ------------------------ |
| Built-in control | Tokens and interaction mechanics           | `Button`, `DataGrid`     |
| Shared control   | Application-wide interaction contracts     | `PagedGrid`, `LookupBox` |
| Domain view      | One business concept                       | `ApprovalDecisionView`   |
| Feature          | One workflow                               | `ApprovalPanelView`      |
| Screen           | Shell placement, layout, and its ViewModel | `ApprovalsScreen`        |

- A layer may use the layers below it, never the ones above.
- Business rules never sink into a style or a shared control.
- A screen composing built-in controls directly is fine. A shared control referencing a domain type is not.
- Do not promote something used once into a domain view. Compose it in place until repetition proves the concept.

## Recommended Inventory

Shell and structure:

| Contract           | Responsibility                                                           |
| ------------------ | ------------------------------------------------------------------------ |
| `ShellWindow`      | Main window: `SideNav`, content region, `ToastHost`, and overlay layer   |
| `SideNav`          | Permission-aware navigation with no more than two levels                 |
| `PageHeader`       | Screen title, context, and the primary action                            |
| `EmptyState`       | Distinguish no data, no matches, missing prerequisite, and no permission |
| `ErrorState`       | Sanitized error with a safe retry command                                |
| `LoadingIndicator` | Pending indicator and skeleton rows that preserve geometry               |

Data and search:

| Contract      | Responsibility                                                                    |
| ------------- | --------------------------------------------------------------------------------- |
| `PagedGrid`   | Shared `DataGrid` style, columns, sorting, selection, states, and `PagingBar`     |
| `PagingBar`   | One footer for paging: position, page size, and previous and next commands        |
| `SearchBox`   | Debounced search text with clear command and keyboard shortcut                    |
| `FilterBar`   | Stable arrangement of search, filters, active filter labels, and grid commands    |
| `StatusBadge` | Shared state label using icon or dot, label, and color, consistent across modules |

Input:

| Contract           | Responsibility                                                                    |
| ------------------ | --------------------------------------------------------------------------------- |
| `FormField`        | Label, input, inline error template, and helper text with stable geometry         |
| `NumericStepper`   | Bounded whole-number input with increment and decrement commands                  |
| `MoneyBox`         | Culture-aware `decimal` entry and display with fixed scale                        |
| `DateField`        | Typed or calendar date entry returning `DateOnly`, with clear behavior            |
| `LookupBox`        | Searchable, paged selection for relationships and large option sets               |
| `SegmentedControl` | Two to five visible, mutually exclusive options with stable geometry              |
| `SelectionCards`   | Card choice among options with meaningful differences                             |
| `ToggleSwitch`     | `CheckBox` style for settings that apply immediately                              |
| `FileField`        | Pick, replace, and remove files through `IFilePickerService`, with per-file state |

Dialogs and feedback:

| Contract             | Responsibility                                                             |
| -------------------- | -------------------------------------------------------------------------- |
| `IDialogService`     | Modal dialogs and confirmations opened from ViewModels                     |
| `ConfirmDialog`      | Risk-proportional confirmation naming the record and the consequence       |
| `TypedConfirmDialog` | Irreversible actions confirmed by typing the record name                   |
| `IFeedbackService`   | Runs every data-changing command: pending, success notice, sanitized error |
| `ToastHost`          | Shared success, error, warning, and information notices                    |
| `INavigationService` | ViewModel-first navigation inside the shell                                |

Names in these tables are the kit defaults. Reuse an equivalent existing control rather
than renaming a mature codebase only to match this document. Which control fits which task
is decided in `component-selection.md`; this inventory only guarantees there is one
implementation of each.

## Contract Shape

Document each shared control with the same five parts, in its XAML file header or next to
its style:

| Part          | Content                                                                      |
| ------------- | ---------------------------------------------------------------------------- |
| Purpose       | The one interaction it owns, and what it deliberately does not do            |
| Bindings      | Dependency properties and commands, with types and default binding mode      |
| States        | Default, hover, keyboard focus, pressed, disabled, running, error, empty     |
| Accessibility | `AutomationProperties.Name`, `LabeledBy`, and the automation peer it exposes |
| Keyboard      | Tab order, arrow keys, Enter, Escape, access keys, and shortcuts             |

Expose values as dependency properties that bind two-way where the user edits them, and
actions as `ICommand` properties. A shared control never raises events that a ViewModel must
handle in code-behind.

## Contract Rules

Shared controls own interaction consistency, not business policy.

- `PagedGrid` owns grid mechanics; the module supplies typed columns, filter options, the page of rows from its ViewModel, and valid row commands.
- `FormField` owns label, error, and helper placement; the form ViewModel owns fields and business validation through `ObservableValidator`.
- `LookupBox` returns the selected item and its key; the search itself is a query the ViewModel supplies. It does not know a specific entity type.
- `IFeedbackService` owns pending state, notification variants, and error sanitization; callers provide human-readable messages and the use case to run.
- `ConfirmDialog` and `TypedConfirmDialog` own the confirmation ladder; the caller supplies the record label and the consequence.
- `StatusBadge` owns the visual encoding of a state; the module maps its domain values onto it.
- Permission-aware commands disable through `CanExecute`, and an `IsAllowed`-style property hides what the user cannot use. This is presentation only; the use case authorizes again.
- `NumericStepper`, `MoneyBox`, and `DateField` own parsing and display and return canonical values, never formatted strings.

Do not put use-case calls, domain-specific permissions, or record lifecycle rules inside a
generic style or shared control.

## Extension Threshold

Add an option to a shared control when:

- at least two modules need the behavior;
- the behavior has the same interaction and accessibility contract;
- the option does not make the base control understand domain entities.

Create a module-specific composition when the workflow has unique business sequencing,
calculations, or lifecycle rules. Compose shared controls inside it.

## External Control Intake

Before adopting a control from a third-party suite or sample code:

1. Inspect the source or package, its license, and the dependencies it adds.
2. Remove demo data, branding, global implicit styles, and unrelated variants.
3. Replace hard-coded colors, radius, effects, fonts, and spacing with project tokens.
4. Replace bundled icons with the project icon family where appropriate.
5. Align properties, commands, and states with the shared contract.
6. Verify keyboard, focus visuals, automation names, the animation setting, light, dark,
   high contrast, display scaling, and touch when supported.
7. Consolidate or remove any existing duplicate control.

Do not keep two grid systems, two notification systems, or several paging patterns after
adopting a replacement.

## Stable Geometry

Shared controls define stable dimensions so icons, running text, validation messages,
hover, or dynamic labels do not shift neighboring content. Reserve the space for the inline
error in `FormField`, use consistent icon sizes, and keep standalone touch targets, such
as navigation items, dialog close, and stepper buttons, at least 44 DIP on touch-capable
products.

## Ownership And Documentation

Document meaningful contract changes in `wpf-mvvm.md` or an ADR. When replacing a shared
control, migrate its consumers in one planned sequence and remove the obsolete variant.
