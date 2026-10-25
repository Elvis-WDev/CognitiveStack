# Motion

Motion keeps the world alive and tells the viewer where to look. Subtle does not mean
little: it means small, slow, and purposeful, so the one motion that matters stands out.

## Two Kinds Of Motion

| Kind      | Purpose                                         | Examples                                                  |
| --------- | ----------------------------------------------- | --------------------------------------------------------- |
| Ambient   | Make the scene feel alive without asking for attention | Steam from a pot, a clock hand, a candle flame, a queue shifting |
| Narrative | Explain a change that moves the story          | A ticket falling, a table emptying, an order arriving     |

Every narrative motion answers what changed. If the answer is "it looks nice", remove it.

## Micro-Animations

Ambient motion stays below the threshold of attention:

| Property    | Range at 1080 px short side       |
| ----------- | --------------------------------- |
| Translation | 2-6 px                            |
| Scale       | 1-3 percent                       |
| Rotation    | 3 degrees or less                 |
| Loop period | 1.5-6 seconds                     |

- At most about three ambient loops visible at once, away from the focal point.
- Desynchronize loops with seeded phase offsets so the world never pulses in unison, except
  in the resolution, where synchrony is the point. See `storytelling.md`.
- Ambient motion never flashes. No more than three flashes per second anywhere, ever.
- Restaurant vocabulary: steam, flames, a ticking clock, a swinging door, a blinking card
  reader, a vibrating phone, a printer feeding paper, a bell's small shake, people shifting
  weight.

## Narrative Motion

- One primary motion at a time. Secondary motions follow it with a 40 to 80 ms stagger and
  never start before it.
- Stagger at most about six siblings. Beyond that, move them as a group.
- Objects enter from the ground up: they rise from the floor plane, unfold along an iso axis,
  or drop in from above with a small settle. They never scale up from their center.
- Objects move along the isometric axes, not in arbitrary screen directions.
- A dropping object may squash 4 to 6 percent on contact, once. No repeated bounces.
- A shadow follows its object: it shrinks and separates as the object rises, and returns as
  it lands.

## Durations

| Motion                                   | Duration      |
| ---------------------------------------- | ------------- |
| Micro state change: light on, dot appears | 150-250 ms   |
| Object entrance                          | 300-500 ms    |
| Object exit                              | About 70 percent of its entrance |
| Object travel across the scene           | 500-900 ms    |
| Text entrance                            | 300-450 ms    |
| Scene transition                         | 400-700 ms    |
| Camera move                              | 1.2-2.5 s     |
| Stillness at the turn                    | 400-800 ms    |

Define durations in seconds, not frames, so they hold at 30 and 60 fps.

## Easing

| Token    | Curve                               | Use                                  |
| -------- | ----------------------------------- | ------------------------------------ |
| `enter`  | `cubic-bezier(0.16, 1, 0.3, 1)`     | Entrances and arrivals: fast in, soft landing |
| `exit`   | `cubic-bezier(0.7, 0, 0.84, 0)`     | Exits: accelerate away               |
| `move`   | `cubic-bezier(0.65, 0, 0.35, 1)`    | Travel and camera moves              |
| `spring` | Damping ratio 0.8-1.0               | Settles; overshoot 3 percent at most |
| `linear` | Constant                            | Only constant mechanical motion: a clock hand, a fan, a conveyor |

- No elastic, bounce, or back easing with visible overshoot.
- Springs are computed in closed form from time, so any frame renders the same way.

## Camera

- The camera stays isometric. It may pan along the iso axes, push in to a focal point, and
  pull out to reveal. It never rotates, tilts into perspective, or shakes.
- One camera move per scene, at most. A push-in followed immediately by a pull-out reads as
  indecision.
- The camera moves slower than the objects inside it, and settles before text appears.

## Transitions

Prefer continuity over cuts. The viewer should think the next scene came from the last one.

- **Object carry:** the focal object travels into the next scene; the camera follows it.
- **Shape flood:** a solid shape from the scene, such as a ticket or a plate, grows to fill
  the frame and becomes the next background.
- **Match cut:** an object cuts to one of the same shape, size, and position.
- **Camera reveal:** a pull-out turns a detail into the room it belongs to.

Banned: crossfades, blur or zoom-blur transitions, glitch, spins, whip pans every few
seconds, and generic slide-left wipes.

## Rhythm

- Cut scenes and land key motions on the beat of the music, typically 90 to 120 BPM for
  problem stories.
- Tempo follows the story: it rises through the escalation, stops at the turn, and settles to
  a steady pace in the resolution.
- Every narrative motion that makes a sound gets one quiet effect, timed to the frame of
  contact. Ambient motion gets ambience, not individual effects.
- No dead time: something meaningful changes at least every two to three seconds, except
  the deliberate stillness at the turn and the hold at the close.

## Determinism

Every value on screen is a function of time alone. No timers, no state carried between
frames, and no unseeded randomness. Any frame must render identically, in any order, every
time. This is what makes a correction a small edit and a re-render. See `production.md`.

## Verification

- [ ] Every narrative motion explains a change; nothing moves only to look busy.
- [ ] Ambient motion stays within the amplitude and period ranges, with at most three loops visible.
- [ ] One primary motion at a time, with secondary motions staggered after it.
- [ ] Objects enter from the ground or along iso axes, never by scaling from their center.
- [ ] Durations and easings come from the tables; no bounce or elastic easing.
- [ ] The camera never rotates, leaves isometric, or shakes; one move per scene at most.
- [ ] No crossfade, blur, glitch, or spin transition.
- [ ] Nothing flashes more than three times per second.
- [ ] Every frame renders identically when rendered alone.
