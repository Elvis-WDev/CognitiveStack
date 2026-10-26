# Design Foundations

Scales, tokens, states, and motion that every screen inherits. A visual value that is not
on a scale does not belong in a view, style, or template.

## Token Discipline

Centralize color, spacing, radius, typography, elevation, layering, motion, and sizes in
XAML resource dictionaries merged by the application. Views consume tokens; they do not
invent their own visual language.

| Dictionary           | Contents                                                        |
| -------------------- | --------------------------------------------------------------- |
| `Tokens.Light.xaml`  | Color and brush tokens for the light theme                      |
| `Tokens.Dark.xaml`   | The same keys with dark values                                  |
| `Tokens.Shared.xaml` | Spacing, radius, type sizes, durations, easing, and fixed sizes |
| `Styles/*.xaml`      | Shared control styles and templates built only from tokens      |

- Reference theme-switchable brushes with `DynamicResource`, so a theme change applies
  without restarting. Reference constants, such as spacing and durations, with `StaticResource`.
- No hard-coded values in views: `Margin="13"`, `Background="#3B82F6"`, `Duration="0:0:0.37"`,
  `Panel.ZIndex="9999"`. A hard-coded value is a defect unless it is `0` or a layout ratio.
- No module-local theme colors. Semantic keys only, so light, dark, and high contrast all
  stay correct.
- Adding a value to a scale is a system decision, documented where the scale lives.
- When a design needs a value the scale lacks, either the scale or the design is wrong.
  Resolve that before writing the view.

## Spacing

Use one arithmetic scale in device-independent pixels (DIP). Base 8, with 4 reserved for
optical correction. Store each step as a `sys:Double` and as `Thickness` resources for the
common margin and padding shapes.

| Token      | Value  | Use                                       |
| ---------- | ------ | ----------------------------------------- |
| `Space.05` | 4 DIP  | Optical adjustment, such as icon to label |
| `Space.1`  | 8 DIP  | Tightly bound elements                    |
| `Space.2`  | 16 DIP | Related elements inside a group           |
| `Space.3`  | 24 DIP | Groups inside a section                   |
| `Space.4`  | 32 DIP | Subsections                               |
| `Space.6`  | 48 DIP | Sections                                  |
| `Space.8`  | 64 DIP | Major divisions                           |
| `Space.10` | 80 DIP | Exceptional separation                    |

Rules:

- Space expresses relation. Keep more space between groups than inside them; the gap states what belongs together.
- Reach for spacing, alignment, and typography before a border, separator, card, or background. See the container rules in `interface-design.md`.
- Equal padding everywhere flattens hierarchy. Vary it deliberately.
- Whitespace is structural, not leftover. It isolates the decision; it does not turn an operational screen into a sparse scroll.
- Lay out with `Grid` rows and columns sized `Auto` and `*`, not fixed widths, so content survives long text, other locales, and display scaling.

## Typography

One small scale, one ratio, few weights. Sizes are DIP and live as tokens; text styles
(`Text.PageTitle`, `Text.Section`, `Text.Body`, `Text.Meta`) apply them.

| Role            | Size      | Weight   |
| --------------- | --------- | -------- |
| Page title      | 26-28 DIP | Bold     |
| Section         | 18-20 DIP | SemiBold |
| Component title | 15-16 DIP | SemiBold |
| Body            | 14 DIP    | Normal   |
| Metadata        | 12 DIP    | Normal   |

- Keep weights to roughly Normal, Medium, and Bold. Hierarchy comes from contrast in weight and size, not from five similar sizes.
- Use one font family that ships with Windows, such as Segoe UI, or one embedded in the application. Never depend on a font that may be missing on the target machine.
- Design for scanning: titles, short labels, prominent values, grouping. Operational screens are scanned in an F or Z path, not read line by line.
- Keep body lines around 60-80 characters with a `MaxWidth` on long text blocks.
- Align numbers consistently in grids, with tabular figures; see `data-tables.md`.
- Do not scale type with the window size. Windows display scaling already enlarges every DIP; see `navigation-and-windows.md`.

## Color

Define semantic brush keys and never raw values in views:

```text
Background  Surface  SurfaceSubtle  Foreground  ForegroundMuted  Border  FocusRing
Primary  PrimaryForeground
Success  Warning  Danger  Info  Neutral
```

- Define the palette in a perceptual space such as OKLCH, with HSL as an acceptable fallback, then store the resulting values in the theme dictionaries, so lightness and chroma relationships hold across states and themes.
- Derive hover, pressed, selected, and disabled from the base token by a defined lightness or opacity step, and store each one as its own key, not as a color chosen per control.
- Keep a dedicated `FocusRing` brush that stays visible on every surface.
- When Windows high contrast is active (`SystemParameters.HighContrast`), switch to a dictionary built from `SystemColors` keys instead of the light or dark tokens. Never hard-code a color that would survive into high contrast.

### 60 / 30 / 10

Distribute color roughly as 60% neutral background, 30% surfaces and structure, and 10%
accent for actions, states, and emphasis. Strong color is a scarce resource. When
everything stands out, nothing does.

### Color Carries Meaning

| Token   | Meaning                             |
| ------- | ----------------------------------- |
| Success | Completed, healthy, approved        |
| Warning | Needs attention, degraded, expiring |
| Danger  | Failed, blocked, destructive        |
| Info    | In progress, informational          |
| Neutral | Inactive, secondary, draft          |

Never encode a state with color alone. Pair it with an icon or dot and a label, as the
shared `StatusBadge` does. High contrast removes the color; the icon and label remain.

## Radius, Elevation, And Depth

- Keep a small `CornerRadius` set and use it to group, not to decorate. Default to 4-8 DIP in work surfaces.
- Prefer a border or a surface step over a shadow. Reserve elevation for genuinely floating layers: popups, menus, dialogs, and toasts. `DropShadowEffect` is expensive; use it only on those layers.
- Layer floating content by mechanism, not by guessed numbers: `Popup` for menus, dropdowns, and tooltips; the adorner layer for focus and validation decoration; owned windows or a shell overlay for dialogs; `ToastHost` for notifications. Inside one panel, take `Panel.ZIndex` values from named tokens only.

## Interactive States

Every interactive control defines its states in its style before it ships, through
triggers or `VisualStateManager` states.

| State          | WPF mechanism                           | Requirement                                                     |
| -------------- | --------------------------------------- | --------------------------------------------------------------- |
| Default        | Base style                              | Legible and clearly interactive                                 |
| Hover          | `IsMouseOver`                           | Perceptible change, pointer only, never the sole affordance     |
| Keyboard focus | `IsKeyboardFocused`, `FocusVisualStyle` | Always present, never set to `{x:Null}`, visible in every theme |
| Pressed        | `IsPressed`                             | Distinct from hover, so a press is unmistakable                 |
| Disabled       | `IsEnabled="False"`                     | Visually distinct, with the reason shown when it is unclear     |
| Running        | Command `IsRunning`                     | Stable geometry and duplicate execution blocked                 |

Add selected, error (`Validation.HasError`), and read-only where the control supports them.
A control whose pressed state is indistinguishable from its resting state is unfinished.
Disabled controls do not show tooltips by default; set `ToolTipService.ShowOnDisabled` when
the tooltip explains why.

## Motion

### The 100ms Rule

The interface acknowledges an interaction within about 100ms. When the operation takes
longer, show a pending state immediately: skeleton rows when content will replace the
surface, an in-control pending state when the surface stays, real progress when it can be
measured. A frozen window is never a pending state; I/O runs off the UI thread.

### Durations

Store durations as `Duration` tokens and easing as `EasingFunctionBase` resources. Storyboards
reference them; they never hard-code a time.

| Transition                   | Duration  |
| ---------------------------- | --------- |
| Hover and small state change | 100-150ms |
| Popup, menu, tooltip         | 120-180ms |
| Dialog, side panel           | 180-250ms |
| Layout and list reflow       | 200-320ms |
| Feedback such as toast entry | 150-300ms |

Use physical easing: `CubicEase` or `QuinticEase` with `EaseOut` for entrances, `EaseIn` for
exits, and `EaseInOut` for standard moves. Linear motion reads as mechanical.

### Animation Must Explain Something

Animate only to explain a change of position, a change of state, an appearance or
dismissal, progress, or temporal hierarchy. Ask what the animation tells the user. If the
answer is that it looks nice, remove it.

- Do not animate every row, every reload, or every card on scroll.
- Animating five meaningful elements beats animating a hundred irrelevant ones.
- Animate `Opacity` and `RenderTransform`, never `Width`, `Height`, or `Margin`, which force layout on every frame.
- Respect the Windows "show animations" setting (`SystemParameters.ClientAreaAnimation`): when it is off, drop travel and loops, apply the final state at once or with a short fade, and keep functional feedback.

## Verification

- [ ] No hard-coded color, spacing, radius, z-order, or duration value was introduced in a view.
- [ ] Theme brushes use `DynamicResource`; constants use `StaticResource`.
- [ ] Spacing expresses grouping, and no container was added where space would do.
- [ ] Type roles come from the scale, with at most three weights.
- [ ] Accent color stays within roughly the 10% budget.
- [ ] No state is communicated by color alone, including in high contrast.
- [ ] Every interactive control covers default, hover, keyboard focus, pressed, disabled, and running.
- [ ] Feedback appears within about 100ms, with a pending state beyond it.
- [ ] Every animation explains a change and respects the Windows animation setting.
