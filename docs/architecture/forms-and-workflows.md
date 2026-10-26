# Forms And Workflow Surfaces

## Select The Right Container

| Situation                                                | Container                     |
| -------------------------------------------------------- | ----------------------------- |
| Short create or edit form with independent fields        | Dialog                        |
| Quick confirmation or one decision                       | `ConfirmDialog`               |
| Contextual detail that benefits from list continuity     | Dialog or side panel          |
| Long form with several sections, calculations, or review | Full screen in the shell      |
| High-risk operation with draft, validate, submit stages  | Full screen in the shell      |
| Simple reversible value change                           | Inline control                |

Dialogs open through `IDialogService`; screens open through `INavigationService`. Do not
force every action into a dialog. Do not embed every create or edit form permanently inside
a list module.

For records that live in a grid, the dialog is the default container for create and edit.
Move to a full screen only when the dialog would genuinely fail the task, such as when:

- the record contains child collections the user manages while editing it;
- the screen is not a row of any grid, such as a profile or an application-wide setting;
- the workflow needs draft, validation, and submission as separate stages.

An exception is a decision to declare in the design of the OpenSpec change, not a preference
to exercise silently.

## Form Construction

Ask for what the task needs, not for what the model stores. A property existing on the
entity is not a reason to render a field: derive it, default it, or leave it to the screen
that owns it. Every question the form asks is work the user has to do, and every optional
field is a decision they have to make about whether to answer it.

A form is not a vertical stack of generic inputs. Before building one, ask:

- Can a field use a control that fits the intent better? See `component-selection.md`.
- Can related fields be grouped, and unnecessary ones removed?
- Can a default or an inferred value save the user effort?
- Can advanced options stay hidden until they are needed?
- Can a selection be shown visually, or its result previewed?

Build it this way:

- One form ViewModel derived from `ObservableValidator`, with data annotations or custom
  validation attributes, exposing errors through `INotifyDataErrorInfo`.
- Build every field from `FormField`: a persistent label, the input, the inline error
  template, and helper text. Placeholders are examples, not labels: `Nombre del proyecto`
  above `Sitio web de marketing`, never `Escribe algo...` alone.
- Write validation messages that are specific and actionable: `La contrasena debe tener al
  menos 8 caracteres`, never `Dato invalido`.
- Group fields by the user's mental model, not database tables.
- Keep required and optional behavior explicit.
- Validate format while typing only when it helps; show routine errors on lost focus or
  after the first submit attempt to avoid visual noise.
- Set a logical tab order, access keys on labels, and `IsDefault` on the primary button so
  `Enter` submits and `IsCancel` on `Cancelar` so `Esc` closes.
- Preserve entered values after a failed result.
- The submit command is an `AsyncRelayCommand` whose `CanExecute` is false while it runs.
- Focus the first invalid field, and add an error summary only when the form is long.

Positive per-field messages should be reserved for sensitive or error-prone input. A
screen full of `Dato valido` text creates noise without improving confidence.

## Defaults

Use a default when it is safe and predictable, because every prefilled correct value is a
decision the user does not have to make: the most common option, the current date, the
recommended configuration, or a remembered preference.

Never default to an option that is destructive, expensive, irreversible, or that shares
data outside the company or workspace. Those choices are made deliberately by the user.

## Multi-Step Processes

Use a stepper or wizard when the process has clearly separate stages, such as `Cuenta`,
`Empresa`, `Facturacion`, and `Confirmacion`, and splitting it reduces what the user must
hold in mind at once.

- Show the current step, the total, and the name of each stage.
- Let the user go back without losing what they entered; one ViewModel owns the whole
  process.
- Validate each step before moving on, and review everything before the final submission.
- Save a draft when abandoning the process would lose real work, so closing the window does
  not discard it.
- Do not split a short form into steps; a three-field form in three screens is friction, not
  structure.

## Rules The Use Case Enforces

The use case stays authoritative and re-validates every domain invariant, but the screen
should not let a user assemble an input the use case is known to reject.

- Mirror known constraints as validation, disabled options, or filtered choices.
- Read the constraint from the application layer when it is configurable; do not keep a
  second copy in the ViewModel.
- Explain the blocked state where the user is blocked, not after submission.
- Still handle the failed result when it arrives: map validation error codes to their
  fields and show the rest at the operation. A mirrored rule is a convenience, never a
  guarantee.
- Never present a ViewModel check as authorization.

## Business-Facing Relationships

Never ask users to type a foreign key, GUID, account ID, file path, or generated code. Use:

- `LookupBox` for moderate and large option sets, searching in the query as the user types;
- a selection dialog with `PagedGrid` for filter-heavy lists;
- dependent lookups for real hierarchies;
- read-only labels for generated references;
- automatic numbering by the use case for internal identifiers.

The ViewModel may hold an ID, but the control displays the recognizable business label.

Selections across paged selection dialogs persist until confirmed or cancelled.

Related records stay reachable. A reference shown in a grid or a detail view leads to the
record it names, and a record that cannot be removed because others depend on it says which
relationship blocks it.

## Conditional Fields

- Show fields only when they apply to the selected type or mode.
- Clear hidden values if they are no longer valid, and drop their validation errors.
- Disable dependent controls until their prerequisite is selected.
- Explain the dependency once, near the disabled control.
- Derive defaults from authoritative configuration rather than hardcoding options in the
  ViewModel.

## Sensitive Inputs And Files

- Passwords and secret values use a `PasswordBox` with a show or hide button; never bind
  the plain value into a long-lived property or a log.
- Edit forms preserve the existing secret when the field is empty.
- Use cases return configuration presence, never plaintext or ciphertext.
- `FileField` shows file name, type, size, validation state, and replacement intent, and
  opens the picker through `IFilePickerService`.
- Validate sensitive files in the use case; an extension filter in the picker is only early
  feedback.
- Never display secret references or certificate locations in grids.

### Managed Files

When a record owns attached files, the interface covers the whole lifecycle, not only the
first attachment:

- accept one or several files wherever the record allows more than one, with visible progress;
- show name, type, size, and validation result per file;
- replace one file without disturbing the rest of the record;
- rename or relabel a file when users will later search for it by name;
- remove a file with confirmation proportional to whether it can be recovered;
- report per-file failures without discarding the files that succeeded;
- keep storage folders, internal file names, and provider names out of the interface.

State the accepted types and the size limit before the user picks a file, not only after a
rejection.

## Date, Time, Number, And Option Inputs

- Use the shared `DateField` instead of a restyled `DatePicker` per module, so typing,
  calendar selection, and culture formats behave the same everywhere.
- Popups close on outside click and `Esc`, return focus to their field, and commit values
  predictably.
- Use `NumericStepper` for bounded integer quantities; reject unsupported decimals.
- Use `MoneyBox` for amounts: it formats for display in the user's culture and binds a
  `decimal`, never `double`.
- Use one business-facing tax or rate selector instead of separate technical code and
  percentage fields when one option determines both.
- Load configurable catalogs through use cases; do not duplicate mutable business options
  in ViewModel constants.

## Dialog Contract

Every dialog opened through `IDialogService` has:

- a concise title and optional one-line purpose;
- an owner window, so it centers on it and never hides behind it;
- focus moved to the first field on open and restored to the invoking control on close;
- `Esc` to cancel unless the operation cannot be safely abandoned;
- `Enter` for the default button, except where the confirming action is destructive;
- a scrollable body with a stable header and footer for long forms;
- `Cancelar` and one clear primary action;
- a size that fits the minimum supported window size at 200 percent scale;
- no nested dialog unless the workflow has no clearer full-screen alternative.

Close the dialog after a successful save, refresh the owning list, and let `IFeedbackService`
show the notification. Keep it open with values intact after a validation or other failed
result.

## Full-Screen Workflow Contract

Use a full screen when users must assemble, compare, calculate, review, or submit a complex
record. A good structure is:

1. `PageHeader` with the reset or new command.
2. Main form in logical full-width sections.
3. Contextual summary or readiness panel only when it helps prevent mistakes.
4. Consistently located draft, validate, and submit commands that stay visible while the
   body scrolls.
5. Clear lifecycle state and recovery path.

Warn before navigating away or closing the window with unsaved changes. Do not hide
essential context in nested dialogs. Do not consume a permanent side panel for generic
instructions that could be concise helper text.

## Settings And Configuration

Configuration is not one long form. Group it by the user's intention, one destination per
group, for example `General`, `Permisos`, `Notificaciones`, `Integraciones`, `Seguridad`,
and `Avanzado`.

- Keep advanced or irreversible settings in their own group instead of mixing them into
  everyday ones.
- Apply progressive disclosure inside a group: show the setting, and reveal its details once
  it is enabled.
- Use a wizard with review and summary when configuration must be assembled in order or
  validated before it takes effect.
- Show the effect where the setting is made: a preview, the resulting value, or the records
  it will affect.
- State whether a change applies immediately or on save, and keep that rule consistent
  across groups.

## Import And Background Work

- Provide a template file when a structured import format is required.
- Parse and validate before execution; report detected rows and actionable issues.
- Run durable imports as persisted pending operations, not inside the dialog's command.
- Show progress and allow the dialog to close without cancelling the work.
- Show the work's state again after navigating away, closing the window, or restarting the
  application.
- Report partial success with counts and an exported error file when useful.

## Edit And Delete Semantics

- Edit forms load current values through the use case, not from the row in the grid.
- Immutable fields appear as labels, not disabled mystery inputs.
- Prefer archive, deactivate, revoke, or soft delete when history and relationships matter.
- Explain consequences before destructive actions.
- Never let ViewModel cascade assumptions replace database integrity rules.

## Confirmation Ladder

Match the friction to how hard the action is to undo:

| Action                                                 | Confirmation                                                    |
| ------------------------------------------------------ | --------------------------------------------------------------- |
| Reversible change such as hide, archive, or deactivate | One step, then undo from the notification                       |
| Irreversible deletion or permanent data loss           | `TypedConfirmDialog`: intent, then typing the exact record name |
| Irreversible bulk action                               | `TypedConfirmDialog` plus the number of affected records        |

Rules:

- Name the record and the consequence in the dialog. Never ask only "Are you sure?".
- Keep the confirming action disabled until the typed value matches exactly.
- Ask for the label the user already sees in the list, never an internal identifier.
- Style the confirming action as destructive, never give it initial focus, and never make
  it the `IsDefault` button.
- Do not spend a second step on actions the user can undo; reserve the cost for real loss.
- Prefer undo to confirmation. An action that can be undone from the notification needs no
  dialog before it, and every unnecessary dialog trains users to click through the ones
  that matter.
- Offer undo only when a use case can actually revert the change.
