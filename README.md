# Ninja Jump

A Flappy Bird–style game: tap to jump, slip through the gaps between the poles, don't touch anything.

Written as a single self-contained `index.html` — no build step, no dependencies. It ships with
placeholder graphics drawn with the Canvas 2D API, and is set up so you can **drop in your own
background, music and ninja sprites** without touching the game logic.

## Adding your own art and music

The ninja sprites and sound effects are already in `assets/`. Add the rest by
dropping files in with these names — the game picks them up on reload:

```
assets/ninja_stand.png   standing pose          (in the repo)
assets/ninja_jump.png    jump pose              (in the repo)
assets/sfx_jump.mp3      jump sound             (in the repo)
assets/sfx_hit.mp3       obstacle hit           (in the repo)
assets/sfx_gameover.mp3  game over sting        (in the repo)
assets/pole.png          obstacle pole          (in the repo)
assets/bg.png            background, tiled + scrolled    <- still needed
assets/music.mp3         looping background music        <- still needed
assets/sfx_score.mp3     point scored (optional)
```

Sprite strips are one row of equal square frames; the frame count is auto-detected, so a 6-frame
strip of 64px art is a 384x64 file. Non-square frames just need a `frames:` count.

Everything is optional. **Any file that's missing falls back to the built-in drawing**, so the game
runs fine with an empty `assets/` folder and keeps running as you add art one piece at a time.
Optional extras — a fall animation, ground art, obstacle art and sound effects — are wired up too.

Paths, sprite sizes, frame rates and volumes all live in the `ASSETS` block at the top of the
`<script>` in `index.html`. **See [`assets/README.md`](assets/README.md) for the full spec.**

## Play

Open `index.html` in any modern browser. That's it.

Or serve it locally:

```sh
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Controls

| Action | Input |
| --- | --- |
| Jump | `Space`, `↑`, `W`, `Enter`, click, or tap |
| Start / retry | Same — any jump input |
| Mute | `M`, or tap the speaker top-right (remembered) |

## How it plays

- The ninja **stands on the ground** until you start. Your first tap launches him
  off it, so the run begins from ground level rather than mid-air.
- Every jump throws a **full forward flip**, so he's spinning whenever he's climbing.
  The flip takes 30 frames (half a second); change `spinFrames` in `ASSETS` to
  spin faster or slower.
- Gravity pulls you down constantly; each jump sets a fixed upward velocity.
- You score a point for every gate you pass.
- The gap narrows by 5px every 5 points, down to a floor of 136px, so it gets harder but stays clearable.
- Hitting a pole or the ground ends the run: the ninja stops spinning and tumbles head-down to the
  ground, Flappy Bird style. The hit sound fires on impact and the game-over sting waits until he
  lands, so the two never talk over each other. Bumping the **ceiling** does not kill you — you just stop rising, so the
  top of the screen is a safe wall rather than an invisible death line.
- Your best score is kept in `localStorage`.

## Tuning

All the feel-related numbers are constants at the top of the script, so they're easy to fiddle with:

```js
const GRAVITY       = 0.42;
const FLAP_VELOCITY = -7.4;
const MAX_FALL      = 11;
const SCROLL_SPEED  = 2.5;
const GATE_SPACING  = 190;   // horizontal distance between gates
const GATE_GAP      = 172;   // starting vertical opening
const MIN_GAP       = 136;   // hardest it ever gets
```

Sprite size (`drawH`) and spin speed (`spinFrames`) live in the `ASSETS` block just above it.

## Implementation notes

- **Fixed timestep.** Logic runs at a fixed 60Hz inside a `requestAnimationFrame` loop with an
  accumulator, so the game plays identically on a 120Hz display and doesn't fast-forward after a tab
  switch (the accumulator is clamped at 250ms).
- **Collision** is a circle-vs-rectangle test against each gate's two solid halves, which is more
  forgiving at the corners than a box-vs-box test and feels fairer than it looks.
- **Rendering** scales for `devicePixelRatio` (capped at 2) and the canvas is letterboxed to fit the
  viewport without distorting the 400×640 play field.
- **Assets never break the game.** Loading is fire-and-forget: a missing or broken file leaves its
  `ok` flag false and the draw call takes its procedural branch. A 2.5s timeout stops a stalled file
  from holding the loading screen open.
- **Obstacle decoration overhangs the hitbox.** The shuriken stick out past the pole, so the art is
  drawn wider than the column via `obstacleOverhang` while the collision box stays the plain 52px
  column. Tiles are laid from the gap end outwards, so the art's finished edge always faces the
  opening and seams get pushed off-screen.
- **The hitbox is independent of the art** — a fixed 17px radius circle regardless of `drawH`, so
  swapping sprites can't accidentally change the difficulty. It covers the torso rather than the
  whole sprite, which keeps a spinning humanoid forgiving instead of punishing.
- **The flip lands on 0, not 2pi.** They draw identically, but leaving the angle at 2pi makes the
  hand-off to the velocity tilt unwind the entire turn backwards. On death the angle is folded back
  into (-pi, pi] for the same reason, and keeps easing after landing so the ninja can't freeze
  mid-flip lying sideways on the ground.
- **Music starts on the first tap**, because browsers block autoplay before a user gesture. Missing
  sound effects fall back to short WebAudio blips.
- **Sound playback doesn't wait on `readyState`.** Only a load error disables a clip, so one that
  hasn't finished decoding still plays rather than being replaced by its placeholder blip. Jumps
  play on cloned elements so rapid taps overlap; the hit and game-over stings restart instead, and
  retrying cuts the sting off.
