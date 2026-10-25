# Isometric Style

A flat isometric world built from solid colors: a cutaway diorama of the place where the
problem happens. Clarity comes from geometry, value contrast, and restraint, never from
effects.

## Projection

- Use **true isometric**: the three axes sit 120 degrees apart, and the two ground axes rise
  30 degrees from horizontal. Use the 2:1 pixel isometric ratio (26.57 degrees) only for
  pixel art, never mixed with true isometric.
- The camera is orthographic and fixed for the whole film. No perspective, no vanishing
  points, no rotation. Parallel edges stay parallel in every frame.
- Every object uses the same projection. One object drawn front-on or in perspective breaks
  the world.

## Grid And Geometry

- Build on one module, the tile. Define its size once, for example 64 px along an iso axis at
  1080 px short side, and size everything in tiles.
- Snap footprints to whole tiles and heights to half or quarter tiles. A chair is half a
  tile; a table one tile; a counter two by one.
- Prefer simple solids: boxes, cylinders, wedges, and extrusions. Round only where the real
  object is round, using the same corner radius family everywhere.
- Keep one level of detail across the film. Detail goes where the story is; the background
  stays simpler than the focal object.
- Use the **cutaway diorama**: a floor slab with walls only on the back two sides, so the
  viewer sees inside the restaurant. Never draw the front walls.

## Light And Shading

One fixed light, from the top left, shades every object in every scene.

| Face            | Tone                                         |
| --------------- | -------------------------------------------- |
| Top             | Base tone of the material                    |
| Left (lit side) | Base lightness minus one shading step        |
| Right (shade)   | Base lightness minus two shading steps       |

- Define a shading step once, for example 0.07 of OKLCH lightness, and derive every face
  from its base color with it. Never pick face colors by eye.
- Keep chroma steady across faces, or reduce it slightly on the shaded face. Never shift the
  hue to fake light.
- Contact shadows are one flat shape on the ground, filled with the `shadow` token, offset
  away from the light. Never blurred, never a gradient.
- A raised object casts a shadow that shrinks and separates from it as it rises; the shadow
  sells height without any perspective.

## Solid Color Only

- Every fill is one color. No gradients, glows, blurs, noise, grain, or textures.
- Express material through color and shape, not texture: wood is a warm brown with plank
  divisions as solid tone changes, steel is a cool gray with a solid lighter top face.
- Opacity is allowed only when the result over its background is still a single flat color,
  such as a shadow over a uniform floor. Never stack transparent layers into a gradient.

## No Borders

- Do not outline objects, characters, text, or UI. No strokes of any width.
- An edge exists because two solid faces have different values. Keep at least one shading
  step between adjacent faces and between an object and what lies behind it.
- When an edge disappears, change a tone. Never add a line to rescue it.
- Details such as plank seams, drawer gaps, or screen bezels are thin solid shapes in a
  darker tone of the same material, not strokes.

## Color

Distribute color as roughly 60 percent canvas and ground, 30 percent structure and
furniture, and 10 percent accent. Strong color is scarce; it marks the focal point.

| Token        | Role                                                         | Starter value |
| ------------ | ------------------------------------------------------------ | ------------- |
| `canvas`     | Background outside the diorama                               | `#F3EEE5`     |
| `ground`     | Floor slab top face                                          | `#E4D9C6`     |
| `shadow`     | Contact shadows on the ground                                | `#CFC3AE`     |
| `ink`        | Text, darkest details                                        | `#26242E`     |
| `structure`  | Walls, counters, neutral furniture                           | `#8E99A8`     |
| `material`   | Wood and warm surfaces                                       | `#C98E5F`     |
| `skin` set   | Three to five varied skin tones for characters               | Project set   |
| `problem`    | The friction point: the lost ticket, the late table          | `#E0533D`     |
| `solution`   | The resolved state, used only after the turn                 | `#2A8F6A`     |

- Replace the starter values with brand tokens when a brand exists, and keep the roles.
- Define base colors in OKLCH so shading steps behave the same across hues.
- The `problem` accent marks what hurts. It spreads as the problem escalates and disappears
  after the turn. The `solution` accent appears only in the resolution.
- Never encode meaning with color alone; pair it with shape, motion, or a label.
- Keep the whole palette to roughly 12 to 16 colors including shading steps. A color that is
  not in the token file does not belong in a frame.

## Characters

- Simplified, geometric people: a rounded body block, a sphere head, simple limbs, mitten
  hands. No outlines and no facial detail beyond, at most, two dots for eyes.
- Same proportions for everyone, for example 2.5 tiles tall, with varied skin tones, hair
  shapes, and builds.
- Identify roles by clothing color and silhouette: chef in light jacket and hat, waiter in
  dark apron, customer in the `structure` or `material` family, manager in a distinct but
  muted color.
- Emotion comes from posture and gesture: slumped shoulders, a raised hand, a head turning to
  the clock. Never from exaggerated faces.
- Avoid stereotypes in clothing, body type, and role assignment.

## Objects, Screens, And Signage

- Phones, tablets, tickets, and screens are flat isometric planes aligned to the face they
  sit on, with their content as solid shapes.
- In-scene text, such as a sign or a ticket, is decoration and is not expected to be read.
  Copy the viewer must read lives in screen space; see `typography.md`.
- Icons follow the same rules: solid shapes, no strokes, one style family.

## Composition

- One focal point per moment. Direct attention with, in order of preference: motion, the
  accent color, value contrast, isolation in negative space, and scale.
- To push a non-focal area back, swap its colors to a lower-contrast tone set. Never blur it
  or fade it with a gradient.
- Keep the diorama inside the safe area and leave a deliberate area of empty canvas for text,
  planned in the storyboard, not found afterwards.
- Recompose each aspect ratio. A 9:16 frame stacks the diorama and the text vertically; it
  is not a crop of the 16:9 frame.
- The diorama sits in the lower or central area of the frame with its front corner toward the
  viewer, so the room reads at a glance.

## Verification

- [ ] One true isometric projection everywhere, fixed camera, no perspective or rotation.
- [ ] Footprints and heights are snapped to the tile grid.
- [ ] Every face tone is derived from its base with the shading step, under one light.
- [ ] No gradient, glow, blur, texture, outline, or stroke exists in any frame.
- [ ] Every edge reads through value contrast alone.
- [ ] Every color comes from the token file; the accent stays near 10 percent of the frame.
- [ ] The `problem` and `solution` accents appear only in their phases.
- [ ] Characters share proportions, read by posture and clothing, and avoid stereotypes.
- [ ] Each moment has one focal point, and each aspect ratio is recomposed, not cropped.
