# Typography

On-screen text names what the image shows and carries the story when the sound is off. It is
read in passing, on a phone, at speed. Every rule below protects that reading.

## Typeface

- One family for the whole film: a clean sans serif with clear numerals, such as Inter,
  Geist, Manrope, or DM Sans, or the brand typeface when one exists.
- Two weights, for example 500 for supporting text and 800 for statements. A third weight
  needs a reason.
- Tabular figures for any number that changes, so counters do not jitter.
- Embed the font files with the project. A render must never depend on a network font.

## Placement

- Copy the viewer must read lives flat in screen space, never skewed onto isometric planes.
- Place text in the empty canvas planned in the storyboard, beside or above the diorama,
  never on top of busy geometry.
- Keep a consistent text position per role across the film: statements in one area,
  captions in another. The eye learns where to look.
- Align text to the frame's text grid, left-aligned for statements of more than one line.
  Center only short single lines.

## Scale

Sizes are a share of the frame's short side, so they hold across formats.

| Role                   | Share of short side | At 1080 px | Weight |
| ---------------------- | ------------------- | ---------- | ------ |
| Hero statement         | 8-9%                | 88-96 px   | 800    |
| Headline               | 6-7%                | 64-76 px   | 800    |
| Supporting line        | 4-4.5%              | 44-48 px   | 500    |
| Caption or subtitle    | 3.7-4%              | 40-44 px   | 500    |
| Label or number tag    | 3.3-3.7%            | 36-40 px   | 500    |

- Nothing smaller than 3.3 percent of the short side. If it does not fit at that size, it
  has too many words.
- Hierarchy comes from size and weight contrast. Do not use five similar sizes.
- Tight tracking on large statements, default tracking on captions and labels.

## Contrast Without Borders Or Shadows

- No text outlines, strokes, drop shadows, or glows.
- Text sits on a solid area with at least 4.5:1 contrast, 7:1 for captions. Large statements
  may use an accent color only if it reaches 3:1 against its background.
- When text must cross the diorama, place a solid plate behind it: one flat color from the
  token file, no border, radius from the project scale.

## Amount And Reading Time

- At most about 12 words on screen at once, in at most two lines.
- At most about 28 characters per line for statements and 42 for captions.
- Hold each text at least 1.0 second plus 0.3 seconds per word, and never less than
  1.5 seconds. Measure it in the animatic, muted, at real size.
- Text never starts exiting while it is still being read, and never moves while it must be
  read, beyond its entrance.

## Copy

- Write in the viewer's words, in the second person and the present tense: "Your table 12
  has waited 25 minutes", not "Customers experience delays".
- Name the concrete thing: the table, the dish, the time, the amount.
- Sentence case. No period at the end of a single-line statement; questions keep their mark.
- One emphasis per line, through weight or the accent color, never both.
- Format numbers, currency, times, and dates in the audience's locale.
- No jargon, internal product terms, or provider names unless the viewer must know them.

## Kinetic Type

Text motion is restrained; it supports reading and never competes with the scene.

- Enter by rising through a mask line or by a short fade with an 8 to 16 px rise. One style
  per film.
- Reveal word by word only for emphasis on a key line, not for every sentence.
- Exit faster than the entrance, or hand the text to the next scene by moving it.
- Never scramble letters, bounce characters, or animate each letter independently on
  running text.
- Counters roll in tabular figures and settle on the final value before the viewer reads it.

## Captions And Subtitles

- When there is a voiceover, burn in captions or deliver an SRT file, as the channel requires.
- At most two lines of 42 characters, in the lower safe area, above any platform interface.
- Captions follow the spoken phrase, not single words, unless the style deliberately
  highlights words on their timestamps.

## Verification

- [ ] One family, two weights, tabular figures for changing numbers, fonts embedded.
- [ ] No text below 3.3 percent of the short side.
- [ ] No outlines, strokes, shadows, or glows on text.
- [ ] Every text meets its contrast ratio, using a solid plate where needed.
- [ ] At most about 12 words on screen and two lines per text.
- [ ] Every text holds at least 1.0 s plus 0.3 s per word, and never less than 1.5 s.
- [ ] Copy is concrete, second person, present tense, and in the audience's locale.
- [ ] One kinetic entrance style is used across the film.
