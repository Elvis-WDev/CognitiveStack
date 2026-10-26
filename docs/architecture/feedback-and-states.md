# Feedback And Interface States

## Feedback Principle

Every user action receives timely, semantic, human-readable feedback. The message should
say what happened and what the user can do next, without exception messages, metadata,
error codes, or framework terminology.

Timely means the interface reacts within about 100 ms, even when the result takes longer.
Every interactive control also carries its own visible states: default, hover, keyboard
focus, pressed, disabled, and loading. See `design-foundations.md` for the state matrix,
durations, and easing.

## Choose The Feedback Surface

| State                                                          | Preferred surface                                 |
| -------------------------------------------------------------- | ------------------------------------------------- |
| Routine save, update, export start, or completion              | Notification in `ToastHost`                       |
| Validation tied to one field                                   | Inline error in `FormField`                       |
| Collection loading or refresh                                  | Skeleton rows inside the surface                  |
| Empty collection                                               | `EmptyState` inside the surface                   |
| Recoverable module load failure                                | Compact `ErrorState` with retry                   |
| Persistent session, permission, outage, or configuration block | Screen-level banner                               |
| Destructive confirmation                                       | `ConfirmDialog` or `TypedConfirmDialog`           |
| Durable background work                                        | Progress surface plus notification on transitions |

Do not use persistent success banners for routine actions. Do not use a notification as the
only place to explain how to correct a specific invalid field.

## Notification Contract

`ToastHost` lives once in `ShellWindow` and is driven only by `IFeedbackService`. It has
semantic variants:

- success;
- error;
- warning;
- information;
- pending, which resolves into one of the others.

Requirements:

- theme-aware opaque background and readable contrast, including high-contrast themes;
- icon and close button, vertically centered;
- enough margin from the window edge that borders and controls are not clipped;
- no decorative accent strip unless it is part of the global design system;
- a width that fits the minimum window size, with wrapping text;
- concise title and optional one-sentence detail;
- optional retry or action when safe;
- an undo action for reversible changes, with enough time to use it and no dismissal while
  it has keyboard focus or the pointer over it;
- announced to screen readers through UI Automation;
- deduplication for repeated identical events.

Map known error codes to business messages. Unknown failures receive a safe generic message
and a support reference only when support can use it to find the log entry.

## Bind Feedback To The Command

Run every create, update, delete, file operation, and state change through
`IFeedbackService`. Whether the user hears back should not depend on each module
remembering to add it.

The shared service owns:

- the pending state of the initiating command and duplicate-execution prevention;
- the idempotency key for use cases that declare one: one key per user intent, reused on
  retry, renewed when the input changes;
- the success notification and the list refresh that follows it;
- the mapping from known `Result` error codes to business messages;
- the safe fallback message for unknown failures and exceptions;
- deduplication of repeated identical results.

The ViewModel supplies the use-case call and the human-readable messages only.

No asynchronous path may end without a resolved outcome. Every command has a handled
failure branch that leaves the interface usable and the user informed. A command that
changes data with no feedback path is an incomplete implementation, not a minor omission.

## Loading

Acknowledge the interaction immediately, then show the shape of the wait: skeleton rows
when content will replace the surface, a pending state in the control when the surface
stays, real progress when it can be measured.

- Keep geometry stable.
- Use skeleton rows for screen sections and grids.
- Use the shared `LoadingIndicator` for compact indeterminate actions.
- Disable the initiating command while a non-idempotent use case runs.
- Avoid blocking the whole window for a local refresh.
- Cancel or ignore stale queries after filters or scope change.
- For slow operations, move from an indeterminate indicator to meaningful progress or status.
- Never freeze the UI thread; a window that stops repainting is the worst loading state.

## Success

- Confirm the completed business action, not the technical operation.
- Refresh or optimistically update the affected data.
- Close the completed dialog and return focus to the control that opened it.
- Do not leave a success banner occupying the screen.
- Do not notify passive initial data loading as a success.

## Errors

An error answers three questions: what failed, what was preserved, and what the user can do
next.

```text
No pudimos guardar los cambios.
Tus datos siguen en el formulario.
[Intentar nuevamente]
```

- Preserve form input after recoverable failures.
- Distinguish validation, permission, not found, conflict, unavailable resource, and
  unexpected failures by their `Result` error codes.
- Show field errors beside fields and operation errors at the operation surface.
- Keep retry safe through idempotency or lifecycle rules. A disabled button reduces
  duplicates; only the use case and the database prevent them. See `idempotency.md`.
- Never render exception messages, stack traces, SQL errors, raw provider payloads, file
  paths, or secret values.
- Log diagnostic detail with context through `ILogger<T>`. See
  `../quality/observability.md`.
- Catch unhandled exceptions globally, log them, and show a sanitized message instead of
  letting the application crash silently.

## Optimistic Updates

Update the interface before the use case completes only when the action is reversible and
almost always succeeds: toggling a flag, reordering a list, marking as read.

- Keep the previous state so a failure restores it exactly.
- On failure, revert and explain. Never leave an optimistic value on screen.
- Do not apply optimistic updates to payments, approvals, deletions, stock, or anything
  whose reversal is not trivial.
- An optimistic update still changes data: it goes through `IFeedbackService` and refreshes
  the data it changed.

## Empty And Partial States

An `EmptyState` answers three questions: what this means, why it is empty, and what the user
can do now. `No hay datos.` answers none of them.

```text
Todavia no has creado proyectos.
Crea el primero para empezar a organizar el trabajo del equipo.
[Crear proyecto]
```

An empty state should distinguish:

- no records exist yet;
- current filters have no matches;
- data is unavailable because of permissions;
- a prerequisite configuration is missing.

Offer only actions the user can perform. Partial screens should render available data and
identify the missing section without blanking unrelated content.

## Validation Timing

- Validate simple format on lost focus.
- Validate cross-field and use-case rules on submit or explicit validation.
- Avoid red errors before the user interacts.
- Avoid green success text under every valid field.
- Use clear constraints in labels or concise helper text before an error occurs.
- Expose validation errors to screen readers through UI Automation, not only as color.

## Tooltips

- Use tooltips for icon-only actions and unfamiliar compact controls.
- Tooltips must be opaque, above surrounding content, theme-aware, and readable.
- Tooltip text names the action; it does not repeat a visible label.
- Disabled controls still need an explanation, through `ToolTipService.ShowOnDisabled`, when
  the reason is not obvious.
- Tooltips do not replace labels for form fields or primary navigation, and they are not
  available on touch: never put required information only in a tooltip.

## Background Work

Long-running work is a persisted pending operation when it must survive closing a dialog,
navigating away, closing the window, or restarting the application. See
`dotnet-architecture.md`. The interface exposes queued, running, partial, completed,
failed, and cancelled states, with progress when measurable.

Closing a progress dialog must not silently cancel durable work. Cancellation requires an
explicit action and support in the handler. Closing the application while work is running
warns the user and states whether the work will resume on the next start.
