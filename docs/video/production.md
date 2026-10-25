# Production

How a video moves from brief to delivered file. Each stage has a gate; a stage that fails
its gate goes back, not forward.

## Formats

| Format   | Size        | Use                                 |
| -------- | ----------- | ----------------------------------- |
| 16:9     | 1920 x 1080 | Master, website, YouTube, LinkedIn  |
| 9:16     | 1080 x 1920 | Reels, TikTok, Shorts, Stories      |
| 1:1      | 1080 x 1080 | Feeds                               |
| 4:5      | 1080 x 1350 | Instagram and LinkedIn feeds        |

- Produce at 60 fps when micro-animations dominate; 30 fps is acceptable for a channel that
  requires it. Timing is defined in seconds either way.
- Recompose each format from the same scenes and timeline. Never crop the master.

### Safe Areas

- Keep text and focal points inside a 5 percent margin on every side of every format.
- In 9:16, keep text out of the top 250 px, the bottom 400 px, and the right 120 px, where
  platform interfaces sit.
- Check every format against its platform overlay before delivery.

## Pipeline

| Stage          | Output                                                   | Gate                                                       |
| -------------- | -------------------------------------------------------- | ---------------------------------------------------------- |
| 1. Brief       | The seven answers in `index.md`                          | Every question answered or the assumption recorded         |
| 2. Script      | Beats, on-screen text, and optional voiceover            | Reads muted; within the word and timing budgets            |
| 3. Style frames | Three finished stills: hook, cost, and resolution       | Pass `isometric-style.md` and `typography.md`              |
| 4. Storyboard  | The table in `storytelling.md`                           | One focal point per row and a named transition out of each |
| 5. Animatic    | Timed blocks and text at real duration, no polish        | Reading times hold; rhythm works muted                     |
| 6. Build       | Scenes animated from the shared kit and tokens           | Each scene renders deterministically on its own            |
| 7. Beat stills | One still per beat and a contact sheet of the whole film | Reviewed at real size; defects fixed before full render    |
| 8. Full render | Every format                                             | Decoded frames from the final file reviewed, not previews  |
| 9. Mix         | Music, effects, and voice                                | Loudness targets met; reads muted                          |
| 10. Delivery   | The files listed below                                   | `../quality/video-checklist.md` passes                     |

The person or agent that builds a stage does not approve it. Review each gate with fresh
eyes, from the rendered output, not from what the build was meant to do.

## Build Requirements

The tooling is a project decision, recorded as an ADR in `../architecture/adr/`. Whatever
it is, the build must provide:

- **Deterministic time:** every frame is a pure function of time; see `motion.md`.
- **Vector rendering** at the output resolution, so solid edges stay crisp.
- **One token file** for colors, type sizes, durations, easings, and the tile size.
- **A reusable isometric kit:** floor slabs, walls, furniture, kitchen equipment, devices,
  and characters, built once and shared by every scene and every video.
- **Scenes as components** that can be rendered and reviewed alone.
- **Seekable preview** of any moment, and a still render of any timestamp.

## Audio

- Music is original or licensed for the intended commercial use, with the license recorded.
  Never use a commercial track without permission.
- Effects are quiet, short, and placed on the frame of the action they belong to.
- Voiceover is optional; when present, music ducks beneath it.
- Master to about -14 LUFS integrated and -1 dBTP true peak, measured after encoding.
- The video must work muted; audio reinforces, it does not explain.

## Accessibility

- Captions for every spoken line.
- Text contrast as defined in `typography.md`.
- No more than three flashes per second.
- The message is understandable without color perception: pair accents with shape or text.

## Deliverables

- One MP4 per format, H.264, yuv420p, at the agreed frame rate.
- The editable source and the commands to reproduce the render.
- A poster frame for each format.
- Subtitles as SRT when there is a voiceover.
- The storyboard and the contact sheet used for the final review.
- The list of licensed assets with their licenses.

## Verification

- [ ] Each format is recomposed, not cropped, and respects its safe areas.
- [ ] Every pipeline gate was passed in order and reviewed from rendered output.
- [ ] The tooling decision is recorded as an ADR.
- [ ] Every frame renders deterministically; scenes render alone.
- [ ] Audio is licensed or original and mastered to the loudness targets.
- [ ] Captions, contrast, and flash limits are met.
- [ ] All deliverables are present.
