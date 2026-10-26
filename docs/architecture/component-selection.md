# Component Selection

Choose the control that represents the user's intention, not the one that can merely store
the value. A screen built by mapping each column type to its primitive control is the
database schema rendered as a form, which `cognitive-architecture.md` already forbids.

The target is the simplest interaction that feels natural, intentional, efficient,
accessible, and refined. Not the simplest control: the simplest control that gives the
user a strong experience.

## Intent Over Primitive Mapping

Do not map data types to controls mechanically.

| Mechanical mapping          | Ask instead                                                          |
| --------------------------- | -------------------------------------------------------------------- |
| string -> `TextBox`         | Is it free text, a known value to search, or a choice from a set?    |
| number -> `TextBox`         | Is it a small quantity, a relative position, or an exact amount?     |
| bool -> `CheckBox`          | Does the change apply immediately, or when the form is saved?        |
| enum -> `ComboBox`          | How many options, and does seeing them all help the decision?        |
| `DateOnly` -> `TextBox`     | Is it one day, a range, a common preset, or an available slot?       |
| collection -> repeated rows | Is it a set of tags, a selection from a catalog, or an ordered list? |

For every interaction, weigh the user's intent, the number of possible values, the value
range, how often it is used, the precision it needs, how important it is, whether comparing
options matters, whether visual information helps, the space available, error prevention,
accessibility, touch and scanner use, and the patterns the design system already provides.

## Decision Framework

1. **Intent**: name what the user is trying to accomplish, not the data type.
2. **Number of choices**: see the table below.
3. **Precision**: see the table below.
4. **Simultaneous visibility**: when seeing options side by side improves the decision, do not hide them in a dropdown.
5. **Visual nature**: when the choice is inherently visual, show it visually.
6. **Frequency**: optimize frequent actions for fewer interactions and for the keyboard.
7. **Danger**: give dangerous actions safeguards proportional to their risk.
8. **Keyboard, touch, and scanner**: the control works with keyboard alone, with touch alone when the product supports it, and accepts scanner input where codes are entered.

| Choices            | Default control                                           |
| ------------------ | --------------------------------------------------------- |
| 1                  | Often no selector at all; show the value                  |
| 2                  | `ToggleSwitch`, `SegmentedControl`, or two `RadioButton`s |
| 3-5                | `SegmentedControl`, `SelectionCards`, radios, or chips    |
| 5-15               | `ComboBox` or `ListBox`, searchable when labels are long  |
| Large or unbounded | `LookupBox`, or a command palette                         |

| Precision needed                       | Control                                             |
| -------------------------------------- | --------------------------------------------------- |
| Exact, arbitrary value                 | `TextBox` or `MoneyBox` with parsing and formatting |
| Relative position in a known range     | `Slider` with the current value visible             |
| Small increments of a bounded quantity | `NumericStepper`                                    |

These are defaults, not laws. Context decides.

## Control Catalog

Each row maps to built-in WPF controls with project styles and to the shared contracts in
`component-system.md`.

### Numbers

| Intent                                                                    | Control                                                                     | Stack                                   |
| ------------------------------------------------------------------------- | --------------------------------------------------------------------------- | --------------------------------------- |
| Small bounded quantity adjusted by one: guests, seats, rooms, items       | Stepper `[-] 3 [+]`, with direct typing and arrow keys                      | `NumericStepper`                        |
| Approximate value in a known range: volume, opacity, intensity, threshold | Slider with the current value, min and max labels, and ticks when useful    | `Slider` with a value label             |
| Minimum and maximum: price, age, distance                                 | Two bounded fields or two sliders, with the selection in text: `$80 - $320` | Two `NumericStepper`s or `Slider`s      |
| Exact or arbitrary amount: invoice total, salary, measurement             | Right-aligned field with culture-aware parsing and formatting to `decimal`  | `MoneyBox`, or `TextBox` in `FormField` |

A slider never captures a precise arbitrary value: an invoice amount of `137.42` is typed.
When a slider value must also be exact, pair it with a numeric field. WPF has no range
slider; do not build one unless two separate bounds are genuinely worse.

### Choices

| Intent                                                                  | Control                                                           | Stack                                                      |
| ----------------------------------------------------------------------- | ----------------------------------------------------------------- | ---------------------------------------------------------- |
| Two to five modes the user should see at once                           | Segmented control                                                 | `SegmentedControl`, a styled `ListBox`                     |
| Options that differ in price, features, consequences, or recommendation | Selection cards with name, key differences, and a selected state  | `SelectionCards`, a styled `ListBox`                       |
| Short tags, categories, or filter values                                | Chips, toggleable for multi-select                                | `ListBox` with `SelectionMode="Multiple"`                  |
| Five to fifteen short, known options                                    | Dropdown                                                          | `ComboBox`                                                 |
| Large sets: customers, products, people, projects                       | Searchable lookup with rich rows: name, code, and one detail      | `LookupBox`                                                |
| Controlled status, priority, or severity                                | Fixed choices with label and icon, never free text or color alone | `SegmentedControl` or `ComboBox`; `StatusBadge` to display |
| Ratings                                                                 | Stars or a segmented scale, operable by keyboard                  | `RadioButton` group with icon templates                    |

Bind selections to the ViewModel through `SelectedItem` or `SelectedValue`, never through
the displayed text. An editable `ComboBox` (`IsEditable`, `IsTextSearchEnabled`) is enough
for type-ahead over a short list; anything that queries a use case is a `LookupBox`.

### Booleans

| The change...                                             | Control        |
| --------------------------------------------------------- | -------------- |
| Applies immediately: notifications, auto-save, visibility | `ToggleSwitch` |
| Applies on save, as part of a form                        | `CheckBox`     |
| Is one of several independent selections                  | `CheckBox`     |
| Records agreement or confirmation                         | `CheckBox`     |

A checkbox that changes a setting immediately, or a toggle switch that waits for a save
button, misleads the user about when the change happens. An immediate toggle switch runs a
command through `IFeedbackService` and confirms its effect.

### Dates And Time

- Use `DateField` instead of a free `TextBox`. It accepts typed dates, because typing is often faster for keyboard operators, and opens a calendar on demand. Bind to `DateOnly` or `DateTimeOffset`, never to a formatted string.
- Use a range with presets for reports and filters: `Hoy`, `Ayer`, `Ultimos 7 dias`, `Ultimos 30 dias`, `Este mes`, `Personalizado`.
- For booking and scheduling, show availability, instead of letting the user pick an unavailable slot and fail on save.

### Text And Codes

- Free text uses a `TextBox` inside `FormField`, with `MaxLength` from the domain rule and `AcceptsReturn` only for genuinely multi-line content.
- Codes entered by a barcode scanner arrive as keystrokes followed by Enter. Give the field focus by default on scan-driven screens, treat Enter as submit for that field, and never let a scan land in an unrelated control.
- Set `InputScope` (`Number`, `EmailSmtpAddress`, `TelephoneNumber`) on touch-capable products so the touch keyboard shows the right layout.
- Secrets use `PasswordBox`, which cannot be bound directly; pass it to the command or use a reviewed attached property, and never store the value in a plain `string` property longer than needed.

### Color, Images, And Visual Choices

- Choose colors from swatches or theme presets. Offer hex or RGB fields only as an advanced option.
- Show inherently visual choices visually: product photos, themes, layouts, and templates as thumbnails or preview tiles, never as text-only dropdowns.

### Files

Use `FileField`: a file picker through `IFilePickerService`, with drag and drop where it
helps, per-file preview, progress, validation, replacement, and removal. State the accepted
types and the maximum size before the user picks a file. The full lifecycle lives in
`forms-and-workflows.md` and the `FileField` contract.

## Action Hierarchy

Actions are controls too. Give each action a level, and keep each level on the same button
style across the application.

| Level       | Examples                             | Button style                                                |
| ----------- | ------------------------------------ | ----------------------------------------------------------- |
| Primary     | Guardar, Crear, Continuar, Confirmar | `Button.Primary`, one per surface, `IsDefault` when it fits |
| Secondary   | Volver, Vista previa, Duplicar       | `Button.Secondary`                                          |
| Tertiary    | Cancelar, Mas informacion            | `Button.Subtle` or `Hyperlink`; `IsCancel` for Cancelar     |
| Destructive | Eliminar, Quitar, Restablecer        | `Button.Danger`, never mistakable for a safe action         |

How many may be visible at once is set by the action budget in `cognitive-architecture.md`.
How destructive actions are confirmed, or undone, is set by `forms-and-workflows.md`. Never
set `IsDefault` on a destructive button.

## Built-In First For Accessibility

A visually attractive control that reduces accessibility is not an improvement.

- Prefer built-in WPF controls with project styles when they already meet the need. They bring keyboard behavior and UI Automation peers.
- Restyle with a `Style` or `ControlTemplate` before writing a custom control; a retemplated `ListBox` keeps selection, keyboard navigation, and automation.
- A custom control matches its built-in equivalent on keyboard operation, focus, `AutomationProperties.Name`, and screen reader announcements, and provides an `AutomationPeer` when no base class supplies one, or it is a regression.
- Give icon-only and unlabeled controls `AutomationProperties.Name`; link labels to fields with `Label.Target` or `AutomationProperties.LabeledBy`.

## Two Biases To Resist

### Primitive UI Bias

Generated XAML converges on screens made only of `TextBox`, `ComboBox`, `CheckBox`, and
`Button`. Those controls are valid, but they are not automatically the final experience.
Evaluate the higher-level control before settling on one.

### Component Novelty Bias

The opposite mistake replaces every built-in control with a custom one. Do not use:

- a slider where typing an exact value is easier;
- a dropdown for two choices;
- a carousel for static information;
- a modal dialog for everything;
- cards, charts, gradients, or animation as decoration;
- a custom control that is less accessible than the built-in one.

## Context Beats Rules

The same control is excellent in one context and wrong in another.

| Control           | Excellent for                | Wrong for                     |
| ----------------- | ---------------------------- | ----------------------------- |
| Slider            | Volume from 0 to 100         | A monthly salary of `$4,825`  |
| Selection cards   | Choosing between three plans | Choosing one of 200 customers |
| Segmented control | Three viewing modes          | Twelve categories             |
| Toggle switch     | A setting that applies now   | A field saved with a form     |

## Examples

| Avoid                                              | Prefer                                                                     |
| -------------------------------------------------- | -------------------------------------------------------------------------- |
| `Cantidad: [ 3 ]`                                  | `[-] 3 [+]`, editable when quantities can be large                         |
| `Opacidad: [ 65 ]`                                 | A slider with `65%` visible                                                |
| `Modo: [ Equilibrado v ]`                          | `[ Rapido ] [ Equilibrado ] [ Calidad ]`                                   |
| `Plan: [ Profesional v ]`                          | Plan cards with name, price, key features, recommended and selected states |
| `[ ] Activar notificaciones` as an instant setting | `Notificaciones` with a toggle switch                                      |
| `[ Seleccionar cliente v ]` with thousands of rows | A `LookupBox` that searches by name, code, or tax ID                       |
| `Estado: [ En progreso ]` as free text             | `[ Pendiente ] [ En progreso ] [ Revision ] [ Hecho ]` as fixed choices    |

## Agent Behavior

Choosing the control is part of implementing the feature, not an optional polish request.
When a requirement states the behavior but not the control, choose the most appropriate
interaction pattern. "Let the user choose between 1 and 8 guests" does not mean a bare
`TextBox`; evaluate `NumericStepper` first.

Improving the interaction never changes the product. See the scope rules in `wpf-mvvm.md`.

Balance usability, visual quality, accessibility, consistency, and implementation
simplicity. Do not optimize one by severely damaging another.

## Reject The Change When

- a form is a column of generic fields that mirrors the schema;
- a dropdown offers two choices, or thousands without search;
- a slider captures a precise arbitrary number;
- free text captures an enumerable state;
- a checkbox acts as an immediate on/off setting;
- options that fit on screen are hidden in a menu;
- a destructive action looks like a safe one, or is the default button;
- a custom control is less accessible than its built-in equivalent;
- complexity was added that does not improve the experience.

## Verification

- [ ] Each control was chosen from the user's intent, not from the column type.
- [ ] Choices that fit on screen are visible, and large sets are searchable.
- [ ] Exact values are typed, approximate ones may slide, and small quantities step.
- [ ] Toggle switches apply immediately, and checkboxes wait for save.
- [ ] Visual choices are shown visually.
- [ ] Action levels map to consistent button styles, and destructive actions stand apart.
- [ ] Scanner input lands in the intended field and submits with Enter where designed.
- [ ] Custom controls match built-in keyboard, focus, automation name, and screen reader behavior.
- [ ] Nothing was added that the task does not need.
