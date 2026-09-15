# Ninja Jump

A Flappy Bird–style game: tap to flap, slip through the gaps in the bamboo, don't touch anything.

Written as a single self-contained `index.html` — no build step, no dependencies. It ships with
placeholder graphics drawn with the Canvas 2D API, and is set up so you can **drop in your own
background, music and ninja sprites** without touching the game logic.

## Adding your own art and music

Put your files in `assets/` using these names and the game picks them up on reload:

```
assets/bg.png            background (tiled + scrolled)
assets/ninja_run.png     run cycle, sprite strip - plays on the start screen
assets/ninja_jump.png    jump, sprite strip - plays while rising
assets/music.mp3         looping background music
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

- Gravity pulls you down constantly; each jump sets a fixed upward velocity.
- You score a point for every bamboo gate you pass.
- The gap narrows by 5px every 5 points, down to a floor of 124px, so it gets harder but stays clearable.
- Hitting bamboo or the ground ends the run. Bumping the **ceiling** does not kill you — you just stop
  rising, so the top of the screen is a safe wall rather than an invisible death line.
- Your best score is kept in `localStorage`.

## Tuning

All the feel-related numbers are constants at the top of the script, so they're easy to fiddle with:

```js
const GRAVITY       = 0.42;
const FLAP_VELOCITY = -7.4;
const MAX_FALL      = 11;
const SCROLL_SPEED  = 2.5;
const GATE_SPACING  = 190;   // horizontal distance between gates
const GATE_GAP      = 158;   // starting vertical opening
const MIN_GAP       = 124;   // hardest it ever gets
```

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
- **The hitbox is independent of the art** — a fixed 15px radius circle regardless of `drawH`, so
  swapping sprites can't accidentally change the difficulty.
- **Music starts on the first tap**, because browsers block autoplay before a user gesture. Missing
  sound effects fall back to short WebAudio blips.
