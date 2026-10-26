# WPF Instructions

Apply to XAML, `Desktop`, and `Presentation` files.

- Build actual product screens, not placeholders.
- Answer the screen brief before implementing: user, goal, primary decision, minimum information, single primary action.
- Design from the user's job, never from the database schema, and never show a value only because a query returns it.
- Keep one primary action per screen and take visual values from the documented spacing, type, color, and motion resources.
- Prefer existing shared controls and styles.
- Choose each control from the user's intent, not from the data type, and never swap an accessible built-in control for a less accessible custom one.
- Keep ViewModels free of WPF types, SQL, files, and network calls; views bind, ViewModels call use cases.
- Keep the UI thread free: I/O runs in `AsyncRelayCommand`s with cancellation.
- Keep forms validated with `ObservableValidator` and accessible through labels, access keys, and automation names.
- Never load every row into memory; page in the query and keep grid virtualization on.
- Read `docs/architecture/wpf-mvvm.md` and the interface-pattern document relevant to the screen before editing.
- Reuse the shared grid, paging bar, form field, dialog, notification, date, tooltip, and navigation controls.
- Keep internal identifiers, exception messages, and implementation details out of user-facing labels and inputs.
- Prefer grid-first module screens and focused workflows over decorative cards, duplicated summaries, and persistent success banners.
- Use `SideNav` submodules instead of tabs to separate the screens of a module.
- Route every command that changes data through `IFeedbackService` so none ends without pending, success, or error feedback.
- Require typed confirmation for irreversible actions and a single step with undo for reversible ones.
- Write copy that addresses the user directly, without third-person narration or internal process detail.
- Verify keyboard-only use, the minimum window size, 100 to 200 percent scale, and light, dark, and high-contrast themes before completion.
- Update UI documentation when user-visible workflows change.
