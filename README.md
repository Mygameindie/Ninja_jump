# Ninja Jump

A Flappy Bird–style game: tap to flap, slip through the gaps in the bamboo, don't touch anything.

Written as a single self-contained `index.html` — no build step, no dependencies, no image or audio
assets. Everything is drawn with the Canvas 2D API.

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
