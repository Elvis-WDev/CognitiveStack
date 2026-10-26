# WPF And MVVM

## Runtime

- UI framework: WPF with XAML, hosted in `{{ProjectName}}.Desktop`.
- Pattern: MVVM with CommunityToolkit.Mvvm.
- ViewModels: `ObservableObject` with `[ObservableProperty]`, commands with
  `[RelayCommand]` or `AsyncRelayCommand`, validation with `ObservableValidator`.
- Composition: ViewModels and services resolved from the Generic Host container.
- Styling: XAML resource dictionaries with semantic tokens and shared styles.
- Icons: one icon family as vector geometry resources.
- Data grids: the WPF `DataGrid` behind one shared grid style and paging contract.
- Feedback: one shared feedback service for every command that changes data.

## Product UI Principle

Build the working product as the first screen. Operational desktop applications should feel
quiet, predictable, and fast for repeated use. The interface exists to help a person scan,
compare, decide, and act; it should not advertise its own implementation.

Do not treat every dataset as a dashboard, every section as a card, or every action as a
permanent panel. Choose the smallest surface that supports the workflow.

Design from the user, never from the schema. A table, a column, a use case, or an existing
feature does not decide what belongs on screen; the user's job does. Answer the screen brief
in `cognitive-architecture.md` before implementing. A screen whose user, decision, required
information, and primary action are unknown is not ready to build.

## Required Pattern Documents

Open only the documents relevant to the task:

- User, decision, information tiers, and action budget: `cognitive-architecture.md`.
- Spacing, type, color, states, and motion tokens: `design-foundations.md`.
- Choosing the control that fits the user's intent: `component-selection.md`.
- Screen hierarchy and visual workload: `interface-design.md`.
- Shared control inventory and contracts: `component-system.md`.
- Grids, filters, columns, row actions, and paging: `data-tables.md`.
- Forms, selectors, dialogs, side panels, and full-page workflows: `forms-and-workflows.md`.
- Notifications, validation, loading, empty, error, and background-work states: `feedback-and-states.md`.
- Shell, navigation, windows, DPI, keyboard, touch, and themes: `navigation-and-windows.md`.
- Completion audit: `../quality/ui-checklist.md`.

## MVVM Rules

- A view binds to its ViewModel. The ViewModel calls use cases from `Application` and never
  touches SQL, files, `HttpClient`, or devices directly.
- ViewModels contain no WPF types: no `Window`, `MessageBox`, `Visibility`, `Brush`, or
  `Dispatcher`. They expose state and commands; converters and styles turn them into visuals.
- Code-behind is limited to purely visual behavior that XAML cannot express, such as moving
  focus, window chrome, or platform interop. Never business rules, SQL, or validation.
- ViewModels reach dialogs, navigation, files, and notifications through interfaces:
  `IDialogService`, `INavigationService`, `IFilePickerService`, and `IFeedbackService`.
- Commands own their availability. `CanExecute` reflects permission, validity, and whether
  the command is already running; the view does not duplicate that logic in triggers.
- One ViewModel per screen or dialog. Child ViewModels for reusable sections are composed by
  the parent, not resolved from inside views.

## Threading

- The UI thread renders and binds. Disk, database, network, XML or file generation, and heavy
  computation never run on it.
- Commands that do I/O are `AsyncRelayCommand`s that accept a `CancellationToken`, expose
  `IsRunning`, and disable themselves while running.
- Properties bound to the interface are updated on the UI thread. Await use cases from the
  ViewModel so continuations return to it; never touch WPF objects from a background thread.
- Never block the UI thread with `.Result`, `.Wait()`, `Thread.Sleep`, or synchronous I/O.

## State Ownership

- Persistent state: the database, read and written only through use cases.
- Screen state: the ViewModel, including the current page, filters, sort, and selection.
- Form state and validation: the form ViewModel, through `ObservableValidator`.
- Session state: the session service, read by ViewModels through `Application`.
- Transient visual state: the view, such as an expanded section or hover.
- Persisted work state: pending operations in SQLite when work must survive closing the
  window or restarting the application.

Do not mirror the same state across the view, several ViewModels, and the database without a
single declared owner.

## Component Policy

1. Reuse an existing shared style, template, or control.
2. Compose built-in WPF controls with the project styles.
3. Add a shared control when at least two workflows need the same contract.
4. Create a module-specific control only when its behavior is genuinely domain-specific.

Third-party control suites are a dependency decision: justify them in an ADR, map them to the
project tokens, verify keyboard and screen-reader behavior, and wrap them behind the shared
contract. Do not accumulate near-duplicate controls.

## Component Construction

- Keep each view and ViewModel focused on one job. A file that keeps growing is decomposed
  before more is added to it.
- Prefer styles and templates over custom controls, and custom controls over user controls
  that copy markup.
- Separate data from presentation: the ViewModel resolves loading, error, and empty; the view
  renders the state it is given.
- Use `DataTemplate` selection by ViewModel type for navigation and polymorphic content.
- Keep bindings typed: set `d:DataContext` for design-time checking and compiled binding
  errors visible in the output, and treat binding errors as defects.

## Scope Of A UI Change

An interface change is bounded by the interface. A redesign does not silently modify business
logic, permissions, use-case contracts, database schema, security rules, or domain behavior.
When a screen needs something the application layer does not provide, document the gap in the
OpenSpec change and implement it as its own change.

- Within that boundary, improve freely: interaction patterns, layout, hierarchy, feedback,
  control choice, state representation, keyboard flow, and discoverability.
- Never invent pricing, permissions, business rules, required fields, or workflows that change
  the product logic.
- Never ship a control that does nothing. An unsupported action is a documented requirement,
  not a button.
- Never replace missing data with a permanent mock. A fixture belongs in a test, not in a
  screen.
- Never drop working functionality because the new layout has no room for it.

## Technical Information Boundary

ViewModels may carry internal IDs in values and commands, but views present labels, names,
business references, or searchable selectors. Database keys, file paths, provider names,
secret references, framework names, exception messages, and raw metadata stay hidden unless
the user has a documented operational reason to inspect them.

Diagnostic and support tools may expose technical detail, but they are permission-gated and
visually distinct from normal workflows.

## Verification Baseline

Before completion, exercise the screen with real conditions, not the demo dataset that made
the screenshot look good:

- zero, one, a handful, and tens of thousands of records;
- realistic, empty, loading, partial, failure, and unauthorized states;
- missing optional values, long names, large numbers, other locales, and other time zones;
- the minimum supported window size and display scales of 100, 125, 150, and 200 percent;
- keyboard-only operation with visible focus, and touch when the product supports it;
- light, dark, and Windows high-contrast themes;
- repeated command clicks, slow operations, cancellation, and closing the window mid-operation;
- a Wine smoke run during development and a real Windows run before release;
- the shared UI checklist.
