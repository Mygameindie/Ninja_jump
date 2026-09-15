# Assets

Drop your files in this folder using the names below and the game picks them up
automatically. **Nothing here is required** — any file that is missing or fails
to load falls back to the built-in canvas drawing, so the game always runs.

All paths and sizes live in the `ASSETS` block at the top of `game.js`. Change a
filename there if you'd rather use your own names.

## Files

| File | What it is | Notes |
| --- | --- | --- |
| `bg.png` | Background | **Still needed.** Tiled horizontally and scrolled. Any size. Edges do not need to match — see below. |
| `ninja_stand.png` | Standing pose | Shown on the start screen, planted on the ground. Already in the repo. |
| `ninja_jump.png` | Jump pose | Used in the air and on death. Already in the repo. |
| `ninja_fall.png` | Fall *(optional)* | If absent, the jump pose is reused. Enable by setting its `src` in `ASSETS`. |
| `music.mp3` | Background music | **Still needed.** Loops. Starts on your first tap (browsers block autoplay before a gesture). |
| `sfx_jump.mp3` | Jump | In the repo. Plays on every jump. |
| `sfx_hit.mp3` | Obstacle hit | In the repo. Fires the instant you clip a pole or the ground. |
| `sfx_gameover.mp3` | Game over | In the repo. Fires once the ninja comes to rest, not on impact. |
| `sfx_score.mp3` | Point scored *(optional)* | Not supplied — a synthesized blip stands in until you add one. |
| `ground.png` | Ground strip *(optional)* | Tiled. 76px tall on screen. Set its `src` in `ASSETS` to enable. |
| `pole.png` | Obstacle | In the repo. Tiled along each column from the gap end outwards. |

## The background

Drop any image in as `bg.png`. Two knobs in `ASSETS` control it:

```js
bgZoom:   1,      // 1 = fit the canvas height; raise it to zoom in
bgMirror: true,   // flip alternate tiles so the loop has no seam
```

**`bgMirror` means a photo does not need to tile.** Normally a scrolling
background must have matching left and right edges or the loop shows a hard
vertical line sliding past every few seconds — and stock photos almost never
tile. With mirroring on, every other copy is flipped, so each tile's edge always
meets an identical copy of itself and the seam disappears. Measured on a
deliberately non-tiling test image, the colour jump at the tile boundary drops
from 221-318 to 1-14, which is below the image's own detail. Turn it off only if
your image already tiles properly.

**`bgZoom` makes the bamboo bigger.** At `1` the image is scaled so its full
height fits the 640px canvas. At `2` it is drawn twice that size, centred, with
the top and bottom cropping off — so the stalks read twice as large. Anything
between works; use it to get the scale you want without re-cropping the file.

A big source image is fine and will look sharper on high-DPI screens; it is
scaled down to fit. Dark, low-contrast images work best, or the white shuriken
and the blue ninja get lost against them.

## The pole

`pole.png` is tiled along each column, starting at the **gap end** so the art's
own finished edge always faces the opening and any seam is pushed off-screen.
Columns are taller than one tile about half the time, and the repeat reads as a
joint between two sections of timber.

The shuriken stick out past the sides of the pole, so the image is wider than
the column it fills:

```js
obstacleBody:     "assets/pole.png",
obstacleOverhang: 1.3029,   // image width / pole width
```

`obstacleOverhang` is what keeps the **wood** aligned to the 52px collision box
while the shuriken hang outside it. Get it wrong and the pole is either too fat
or too thin for its own hitbox, so you die on thin air or clip through wood.
To recompute it after redrawing: divide the image width by the width of the
pole itself (ignoring anything sticking out), keeping the pole horizontally
centred in the image. Set it to `1` if your art has no overhang.

Decoration that overhangs is **cosmetic only** — the collision box is always
the plain 52px column, so a shuriken can never kill you.

A `cap` image is optional and unused here, since the pole draws its own ends.

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

**Export alpha as drawn and don't post-process it.** Stripping "white
background" from an image by colour will also hollow out white *details* — the
shuriken on the pole, the ninja's eyes and chest emblem are all white and sit
inside the artwork. If a file genuinely has an opaque white background, remove
it with a flood fill from the edges, which leaves enclosed white areas alone.

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
and feet can clip past a pole without killing you. For a humanoid that spins in
the air that reads as generous rather than broken — tightening it to the art
would make the game punishing. If your sprite has a lot of empty padding and
feels unfairly large, lower `drawH` rather than touching the hitbox.

The standing pose is placed so the **bottom edge of the frame rests on the
ground line**, so draw the feet at the bottom of the frame with no padding
underneath (or correct it with `offsetY`).

## Audio notes

### The two death sounds

`sfx_hit.mp3` plays on impact and `sfx_gameover.mp3` plays when the ninja
finishes tumbling and lands, so the sting lands on the thud rather than talking
over the crash. `gameOverDelay` in `ASSETS` (30 frames, half a second) is the
floor on the gap between them, which matters when you die at ground level and
land instantly — without it the two clips would stack. If you swap in a longer
hit sound, raise it to match.

Retrying while the sting is still playing cuts it off rather than letting it run
under the new game.

### General

- `music.mp3` cannot start until the player taps, clicks or presses a key —
  that's a browser autoplay rule, not a bug.
- Volume is set by `musicVol` and `sfxVol` in `ASSETS` (0 to 1).
- `M` mutes, or tap the speaker in the top-right. The setting is remembered.
- MP3 and WAV are the safest formats; OGG is not supported everywhere.
- Keep effects short. Anything longer than the action it marks will overlap the
  next one — the shipped clips are 0.37s, 0.44s and 0.60s.
