# Assets

Drop your files in this folder using the names below and the game picks them up
automatically. **Nothing here is required** — any file that is missing or fails
to load falls back to the built-in canvas drawing, so the game always runs.

All paths and sizes live in the `ASSETS` block at the top of the `<script>` in
`index.html`. Change a filename there if you'd rather use your own names.

## Files

| File | What it is | Notes |
| --- | --- | --- |
| `bg.png` | Background | Tiled horizontally and scrolled. Any size; scaled to the 640px canvas height. **Make the left and right edges match** so the loop is seamless. |
| `ninja_stand.png` | Standing pose | Shown on the start screen, planted on the ground. Already in the repo. |
| `ninja_jump.png` | Jump pose | Used in the air and on death. Already in the repo. |
| `ninja_fall.png` | Fall *(optional)* | If absent, the jump pose is reused. Enable by setting its `src` in `ASSETS`. |
| `music.mp3` | Background music | Loops. Starts on your first tap (browsers block autoplay before a gesture). |
| `sfx_jump.wav` | Jump sound | Optional — a synthesized blip is used if missing. |
| `sfx_score.wav` | Point scored | Optional. |
| `sfx_hit.wav` | Crash | Optional. |
| `ground.png` | Ground strip *(optional)* | Tiled. 76px tall on screen. Set its `src` in `ASSETS` to enable. |
| `bamboo.png` + `bamboo_cap.png` | Obstacle art *(optional)* | Body is tiled along the column, cap is drawn at the gap end. Set both in `ASSETS`. |

## Sprite sheet format

A **horizontal strip** — every frame in one row, all the same size, no padding
or margin between them:

```
+--------+--------+--------+--------+
| frame1 | frame2 | frame3 | frame4 |
+--------+--------+--------+--------+
```

Frame count is **auto-detected assuming square frames**, so a 4-frame strip of
64x64 art should be a 256x64 file. If your frames aren't square — including a
single non-square pose — say how many frames there are:

```js
stand: { src: "assets/ninja_stand.png", frames: 1, rows: 1, fps: 10, loop: true  },
jump:  { src: "assets/ninja_jump.png",  frames: 1, rows: 1, fps: 14, loop: false },
```

For a grid sheet, set `rows` as well as `frames`.

A **single pose is fine** — that's what the shipped sprites are. The airborne
spin is done in code by rotating the sprite, not by animation frames, so one
jump pose is all it needs. If you later draw a proper run cycle, drop the strip
into the `stand` slot with its real `frames` count and it animates instead.

### Keeping poses aligned

The two shipped sprites were cropped to **one shared bounding box**, so the
character sits at the same scale and position in both frames and doesn't jump
around when the pose switches. If you redraw them, export both at the same
canvas size with the character in the same place rather than trimming each one
tight to its own outline.

Transparent PNG is what you want. The ninja should **face right** — that's the
direction of travel.

## Sizing and positioning

```js
ninja: {
  drawH:   58,   // on-screen height in px
  offsetX: 0,    // nudge if the art doesn't sit centred in its frame
  offsetY: 0,
  ...
}
```

`drawH` is the on-screen height of the whole sprite **frame**, and it's purely
visual — the collision hitbox is a fixed 17px radius circle centred on the
ninja and does not change with it. That's deliberate: you can resize or restyle
the art without altering the difficulty.

The hitbox deliberately covers roughly the torso, not the full sprite, so hands
and feet can clip past bamboo without killing you. For a humanoid that spins in
the air that reads as generous rather than broken — tightening it to the art
would make the game punishing. If your sprite has a lot of empty padding and
feels unfairly large, lower `drawH` rather than touching the hitbox.

The standing pose is placed so the **bottom edge of the frame rests on the
ground line**, so draw the feet at the bottom of the frame with no padding
underneath (or correct it with `offsetY`).

## Audio notes

- `music.mp3` cannot start until the player taps, clicks or presses a key —
  that's a browser autoplay rule, not a bug.
- Volume is set by `musicVol` and `sfxVol` in `ASSETS` (0 to 1).
- `M` mutes, or tap the speaker in the top-right. The setting is remembered.
- MP3 and WAV are the safest formats; OGG is not supported everywhere.
