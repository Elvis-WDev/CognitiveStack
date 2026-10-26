# Navigation, Windows, Input, And Themes

## Shell Window

The application runs in one `ShellWindow`: `SideNav` on the leading edge, `PageHeader` above
the content, the current screen in a content region, and `ToastHost` over it.

- `INavigationService` swaps the screen ViewModel; a `DataTemplate` per ViewModel type
  renders it. Views never create other views.
- The shell owns session context, theme, and global shortcuts. Screens own their content.
- Persist the window size, position, and maximized state per user, and restore them only
  when they still fit a connected monitor.

## Navigation Hierarchy

Use at most two visible levels:

1. Module or workflow group.
2. Direct destination.

Avoid third-level navigation, boxed submenus inside boxed modules, and multiple menu
entries that represent only filtered states of the same destination. Put those states in
the destination's filters or a `SegmentedControl`.

When a module has only one destination, link the module directly instead of forcing an
unnecessary expand or collapse step.

## Submodules Instead Of Tabs

A module that covers several areas exposes them as submodules under its `SideNav` group,
never as a `TabControl` that splits one screen into several.

A section is a submodule when it has any of:

- its own collection of records;
- its own create, edit, or delete commands;
- its own permission;
- a destination worth reaching directly from navigation or a shortcut.

Tabs and segmented controls remain valid only for switching lenses over data already on
screen: a filtered view of the same collection, a grid and chart toggle, or grouped
read-only detail of one open record. They must never stand in for a destination.

This keeps every destination visible in `SideNav`, keeps navigation state restorable, and
stops one screen from silently growing into several unrelated ones.

## SideNav

- Use plain labels with icons from the project icon family, not a stack of decorative boxes.
- Chevrons belong only to expandable parents.
- Apply the solid active treatment only to the selected destination.
- Keep a selected child visually distinct without making its parent look selected.
- Use a small consistent radius for active and hover states.
- Group destinations by user workflow and permissions, not by project or namespace structure.
- Name destinations in the user's words: `Facturas`, `Clientes`, `Productos`. Internal
  service, module, or class names do not belong in navigation.
- Keep configuration and help in predictable lower positions.
- Collapse to icons with tooltips below a documented window width, and keep the same
  destinations in the same order.

## Permission-Aware Navigation

- Bind each destination's visibility to an `IsAllowed`-style property computed from the
  session port, and each navigation command's `CanExecute` to the same permission.
- Hide destinations the user cannot access. This is presentation only: the use case behind
  every screen enforces the permission again.
- Recompute the navigation when the session or role changes, and leave a screen the user no
  longer may see.

## Reorderable Navigation

Let users drag navigation groups and destinations into their own order when the product
serves operators who return to the same few screens every day.

- Persist the order per user; restarting the application must not reset it.
- Provide a keyboard alternative to dragging, such as move up and move down commands.
- Show a clear drag affordance, an obvious drop target, and the resulting position.
- Reordering changes presentation only. It never changes permissions or which destinations
  exist.
- Keep the drag gesture from competing with normal selection, especially on touch.
- Offer a way back to the default order.

## Page Header

`PageHeader` stays compact and stable:

- current context or breadcrumb and screen title;
- the screen's primary command when the task depends on it;
- company or workspace selector, when the product has several;
- theme control and the user menu for profile, lock, and sign-out.

Use borderless icon controls when their grouping is already clear. Keep icon size medium,
targets at the documented minimum, and avoid displaying a long account name when the user
menu can provide it.

Do not duplicate the same module title immediately below the header.

## Screens And Windows

Default to in-shell screens. Open a separate window only when the user must see two things
side by side for a sustained task, such as a customer display, a second-monitor report, or
comparing two records.

| Need                                             | Surface                               |
| ------------------------------------------------ | ------------------------------------- |
| A destination in the product                     | In-shell screen through navigation    |
| A short decision or form tied to the current one | Modal dialog through `IDialogService` |
| Context beside a list without leaving it         | Side panel inside the screen          |
| Sustained parallel work or a second monitor      | Separate owned window                 |

Multiple windows policy:

- One `ShellWindow` per running instance; a second launch activates the existing instance
  unless the product documents otherwise.
- Every secondary window and dialog has an owner, closes with it, and never opens off
  screen.
- Two windows editing the same record rely on conflict detection in the use case, never on
  hoping it does not happen. See `idempotency.md`.
- Closing the shell with unsaved changes or running work asks once, names what is pending,
  and states what resumes on the next start.

## Navigation State

- Returning to a list restores filters, sort, page, scroll position, and selection; the list
  ViewModel owns that state and `INavigationService` keeps or rebuilds it.
- Back navigation works with `Alt+Left` and the mouse back button.
- Restore the last destination on startup only when it is safe; never reopen a half-done
  destructive operation.

## Window Size And Scaling

- Declare Per-Monitor V2 DPI awareness in the application manifest, so text and vectors stay
  sharp when the window moves between monitors.
- Document a minimum supported window size in DIP, for example 1024 x 640, and set
  `MinWidth` and `MinHeight` on the shell to it.
- Design at the minimum size first, then let larger windows add density: more columns, a
  side panel, wider forms. Do not merely stretch the minimum layout.
- Lay out with `Grid` star sizing and `MinWidth` on content; avoid fixed pixel positions and
  `Canvas` for layout.
- Let toolbars wrap by control group rather than compressing inputs below usable width.
- Use one column for forms when side-by-side fields become cramped.
- Keep dialog height within the owner window and scroll its body.
- Wrap or trim long names, emails, references, and translated labels, with the full value in
  a tooltip.
- Use vector icons and images with enough resolution for 200 percent scale.
- Test at the minimum window size and at display scales of 100, 125, 150, and 200 percent,
  plus Windows text size enlargement.

## Keyboard

Operational users repeat the same work all day; the keyboard path must be complete, not an
afterthought.

- Every command is reachable without a mouse. Set a logical `TabIndex` order, and use
  `KeyboardNavigation` scopes so `Tab` cycles inside panels and grids predictably.
- Give labels and buttons access keys with `_` in their content, unique within a screen.
- Define global shortcuts in the shell as `InputBindings` bound to commands, documented in
  one place and shown in tooltips and menus: for example `Ctrl+N` new, `Ctrl+F` search,
  `F5` refresh, `Ctrl+S` save, `Esc` cancel or close.
- Never assign a shortcut that conflicts with Windows or the text box editing keys.
- Use function keys for high-frequency point-of-sale actions, shown on the buttons that run
  them.

## Focus Management

- Move focus to the first useful control when a screen or dialog opens: the search box on
  lists, the first field on forms.
- Return focus to the invoking control when a dialog closes, and to the affected row after a
  grid refresh.
- Keep keyboard focus visible with the shared focus visual in every theme.
- Never leave focus on a control that just became hidden or disabled.
- Keep the focused element unobscured by notifications, side panels, or sticky footers.

## Touch And Point-Of-Sale Input

- Size touch targets at about 40 DIP or more on touch stations, with spacing between
  adjacent targets. Define this as a density setting, not per screen.
- Never rely on hover alone. Anything revealed on hover is also reachable by tap and by
  keyboard.
- Keep the primary action in a fixed, reachable position on touch screens.
- Use `NumericStepper`, numeric keypads, and `SelectionCards` instead of small dropdowns
  where operators tap.
- Set `InputScope` on text boxes so the touch keyboard offers the right layout: `Number`,
  `EmailSmtpAddress`, or `TelephoneNumber`.
- Test touch scrolling and panning in grids, lists, and side panels.

## Barcode Scanners

Most scanners act as a keyboard: they type the code quickly and end with `Enter`.

- Keep focus on the scan target, such as the item entry box on a sale screen, and return it
  there after every action and dialog.
- Treat a fast burst of keystrokes ending in `Enter` as a scan: look the code up, add or
  open the match, and clear the box for the next scan.
- Never let a scan land in an unrelated field or press a default button by accident; a scan
  arriving while a dialog is open goes to the dialog only when it expects one.
- Give clear feedback for an unknown code without a modal that stops the next scan.
- Do not depend on the keyboard layout the scanner assumes; verify the configured layout
  during setup.

## Themes

- Use semantic brush resources from the theme dictionaries, never module-local hardcoded
  colors. See `design-foundations.md`.
- Ship light and dark themes, and follow the Windows app mode until the user chooses one.
  Persist the explicit choice per user.
- When Windows high contrast is on, switch to `SystemColors` resources and drop decorative
  color, so text, borders, focus, and selection follow the user's contrast theme.
- Verify background, foreground, border, input, popup, tooltip, notification, destructive,
  selection, and focus colors in every theme.
- Keep charts legible and distinguishable without relying only on hue.
- Apply the theme before the first window renders, so it never flashes in the wrong one.
- Respect the Windows show animations setting through `SystemParameters`, and skip
  nonessential animation when it is off.

## Density And Sizing

- Use medium, consistent icons throughout headers, forms, grids, dialogs, and navigation.
- Keep icon buttons and counters dimensionally stable so hover or loading states do not
  shift layout.
- Keep compact headings inside dense surfaces.
- Prefer a restrained multi-neutral palette with semantic accents over a one-hue interface.
- Offer a compact density for keyboard and mouse stations and a touch density for point of
  sale, from the same tokens.

## Accessibility

The baseline is WCAG 2.2 level AA, applied to desktop through UI Automation. Accessibility
is part of the initial architecture, not a later pass.

- Give every icon-only control an `AutomationProperties.Name`.
- Associate labels with fields through `Label.Target` and `AutomationProperties.LabeledBy`.
- Expose errors and helper text through `AutomationProperties.HelpText` or the validation
  template.
- Preserve visible focus and a logical keyboard order.
- Provide a pointer-free alternative to every drag interaction.
- Keep help, account, and support controls in the same place across screens.
- Do not require a memory or puzzle step to sign in, and allow pasting into password fields.
- Ensure dialogs take focus and expose their title as the window name.
- Do not communicate status only through color.
- Meet contrast in every theme and interaction state: 4.5:1 for body text, 3:1 for large
  text and meaningful non-text elements.
- Check screens with Narrator and Accessibility Insights for Windows before release.
