# How Kitty Unicorn Dolphin Rider works

A technical map of `index.html`, written so a fresh session (or a future you) can pick this up
without re-reading 1,300 lines. **CLAUDE.md is the rules; this is the mechanics.**

Line numbers are hints and will drift — search by function name, which won't.

---

## The shape of it

One file, ~1,270 lines, ~52KB. No build step, no dependencies, no network. Open it in a
browser and it runs, offline. All art is canvas 2D drawing code; the only "assets" are three
emoji characters (🦪 ⭐ ❤️) used for seafloor scenery and the lives counter.

`index.html` is a 14-line HTML shell (a full-window `<canvas>`, a viewport meta tag that
blocks pinch-zoom, and `touch-action: none` so swiping doesn't scroll the page) wrapping one
`<script>` in five labelled sections:

| Section | ~Line | What lives there |
|---|---|---|
| SETUP | 17 | Constants, canvas sizing, geometry helpers, game state, audio |
| INPUT | 380 | Pointer + keyboard handling, button hit-testing |
| UPDATE | 461 | Spawning, physics, collision — everything that changes state |
| DRAW | 612 | Every sprite, the world, the HUD, and the three full-screen pages |
| MAIN LOOP | 1242 | `requestAnimationFrame`, delta-time calculation |

---

## The two invariants that matter most

### 1. Everything moves per SECOND, never per frame

`loop(now)` receives a timestamp, computes `dt` in seconds, and every movement multiplies by
it. Break this and the game runs at double speed on a 120Hz screen — which is exactly what it
did before, and the bug was invisible on a 60Hz monitor.

```js
let dt = (now - lastTime) / 1000;
if (dt > MAX_DT) dt = MAX_DT;   // 0.05 — a backgrounded tab would teleport the dolphin
elapsed += dt;                  // seconds since load; drives every wobble/pulse animation
```

Two consequences worth internalising:

- **Speeds are large numbers** (~300–8000), because they're per-second. Any constant you
  multiply by a speed must be scaled to match. Forgetting this made the dolphin spin: `tilt =
  dy * 0.03` was tuned for a per-frame `dy` of ~20, and became 36 *radians* once `dy` was
  ~1200. It's `TILT_PER_SPEED = 0.0005` now, with a `MAX_TILT` clamp as a backstop.
- **Drag can't be a plain multiply.** `applyDrag(v, keep, dt)` does `v * keep^(dt*60)` so
  "keep 92% of speed every 1/60s" means the same thing at any frame rate.

There is one deliberate exception: **`updateSeaweed(dt)` is called from `loop()`, not
`update()`.** `update()` returns early unless you're PLAYING, but the seafloor keeps scrolling
on the start screen, and weeds rooted in moving sand have to move with it.

### 2. Game logic uses `viewW`/`viewH`, never `canvas.width`/`canvas.height`

`resizeCanvas()` sets the canvas backing store to CSS pixels × `devicePixelRatio` (so art is
sharp on Retina), then applies `ctx.setTransform(dpr, 0, 0, dpr, 0, 0)` so all drawing code
can keep thinking in normal pixels. `viewW`/`viewH` hold the CSS-pixel size.

`canvas.width` is now in *device* pixels and is wrong for any layout maths. Setting
`canvas.width` also resets the context, which is why `setTransform` is re-applied inside
`resizeCanvas()` on every resize.

---

## Game states

Four, held in `gameState`:

- **`START`** — title, Vivian's credit, a *How to play* button, blinking prompt. The hero
  bobs on the waterline behind a dark overlay.
- **`HOWTO`** — full-page instructions. Any tap, the Back button, or Enter returns to START.
- **`PLAYING`** — the game.
- **`GAMEOVER`** — final score and a **Try Again** button you must actually hit.

`update()` and `spawn()` no-op outside PLAYING. `draw()` always runs.

---

## Draw order — and why it's load-bearing

`draw()` paints in this order, and **the order is the trick**, not an accident:

1. Sky
2. Clouds
3. Sandy seafloor + emoji scenery
4. **Seaweed**
5. **The hero**
6. **The translucent water** (`rgba(30,144,255,0.45)`) from the wavy surface down
7. Splash particles
8. Items and obstacles
9. HUD, then whichever full-screen page applies

The hero is drawn **before** the water so that when she dives, the water layer paints over her
and she genuinely looks submerged. Seaweed sits behind her and gets the same tint, which is
what pushes it into the background.

**This distorts colour, and you must judge underwater art through it.** A pale pink jellyfish
reads as lavender once tinted. Items and obstacles are drawn *after* the water (step 8), so
they are *not* tinted — a deliberate inconsistency so hazards stay clearly visible.

The wavy surface is traced in 20px steps, then **pinned to `viewW` explicitly**. Without that
final point, a screen whose width isn't a multiple of 20 (375, say) left a wedge of missing
water in the top-right corner.

---

## The moving parts

| Array | Holds | Movement | Recycling |
|---|---|---|---|
| `items` | gem / donut / heart / rainbow | `SCROLL_SPEED` 300px/s | Spliced past `x < -50` |
| `obstacles` | shell / urchin / jelly | `OBSTACLE_SPEED` 360px/s | Spliced past `x < -50` |
| `splashes` | 30 droplets per surface crossing | Own gravity, fades by `SPLASH_FADE` | Spliced at `life <= 0` |
| `clouds` | 6, fixed pool | 12–42px/s | Wrap to the right edge |
| `seaweed` | 12, fixed pool | `SCROLL_SPEED`, locked to the sand | Replaced by `makeSeaweed()` |

`player` is `{x, y, dy}`. **x never changes during play** — the world scrolls past a fixed
dolphin. `playerHomeX()` sets that x responsively: a quarter of screen width, capped at 200,
floored at 90. A fixed 200 was over half a phone screen and left no reaction time.

### Physics

Two zones, split at `waterLineY()` (`viewH/2 + 20`):

- **In water:** holding applies `RISE_ACCEL` up, releasing applies `SINK_ACCEL` down, then
  `WATER_DRAG`.
- **In air:** purely ballistic — `AIR_GRAVITY` plus `AIR_DRAG`, and **input does nothing.**
  You commit to a leap when you launch. This is the signature mechanic, and the one thing a
  new player can't guess, which is why the how-to page leads with it.

Crossing the surface in either direction fires `createSplash()`.

---

## Spawning

Two independent rolls per frame, each a per-second rate × `dt`:

**Items — `ITEM_SPAWN_RATE` 1.2/sec**

| Type | Chance | Value | Where |
|---|---|---|---|
| Gem | 72% | 10 | Anywhere, sky to seafloor |
| Donut | 20% | 50 | Sky only |
| Heart | 5% | +1 life | Sky only |
| Rainbow | 3% | **116** | Sky only, extra margins |

Sky-only is the risk/reward spine: dive for cheap gems in safety, or leap for the good stuff.
The rainbow gets its own `RAINBOW_MIN_Y` / `RAINBOW_MARGIN` because its bubble is far bigger
than the other sprites and would otherwise clip off the top or dip into the waves.

**Obstacles — `OBSTACLE_SPAWN_RATE` 0.36/sec**

| Kind | Chance | Notes |
|---|---|---|
| Spiky shell | 76% | The default hazard |
| Sea urchin | 12% | Spawns 12px lower (spines reach higher) |
| Jellyfish | 12% | Spawns 22px lower, 40px higher off the floor (tentacles hang) |

All three are underwater and all cost a life.

### Collision

Items use one generous box (`< 50` on each axis) — forgiving on purpose.

Obstacles use **per-kind boxes** in `OBSTACLE_HITBOX`, which is not optional tidiness:

```js
shell:  { hw: 45, hh: 45, oy: 0 }
urchin: { hw: 36, hh: 36, oy: 0 }
jelly:  { hw: 26, hh: 30, oy: 10 }   // narrow; oy shifts the box down over the tentacles
```

The jellyfish is a narrow dome with long trailing tentacles. In the shell's 90px-wide box it
would sting you through empty water at its sides. **Don't collapse these back into one box.**

### The warning shimmer

Every hazard calls one shared `drawHazardShimmer(warn)`. `warn` ramps 0→1 over the last
`SHELL_WARN_DISTANCE` (600px, ~1.7s) and fades out over 120px *behind* the player — without
that tail the glow popped off the instant a hazard slipped past, which looked like a glitch.

It is **red**, deliberately. A first attempt used gold and white sparkles and read as
*treasure*, so players swam toward the hazards.

---

## Input

`handlePointerDown` records the pointer position, then branches on `gameState`. Mouse, touch,
Space and Enter are all wired; `mousemove` tracks the pointer for button hover states.

**The pattern to copy for any new button:** its rectangle lives in *one function*
(`howToButtonRect`, `howToBackRect`, `gameOverButtonRect`), called by both the drawing code
and the click test. The classic bug here is drawing a button in one place and hit-testing
hardcoded coordinates somewhere else, so they silently drift apart when the layout changes.

On GAMEOVER only the Try Again button restarts — tapping anywhere used to wipe your score
before you'd read it.

---

## Audio

Web Audio synthesised in code; no files. `initAudio()` runs on first interaction (browsers
block audio before a gesture). Five sounds: collect, donut (same shape, higher pitch), heart,
hit, splash (sine bloop + band-passed white noise), and the rainbow's five-note rising
arpeggio — the only bespoke one, for the rarest item.

---

## The how-to page

Laid out in fixed "natural" units, then scaled to fit. Two shapes, chosen by
`howToLayout().wide` (`viewH < 520 && viewW > 560`):

- **Single column** (portrait, desktop) — 330×480 natural units.
- **Two columns** (landscape) — 660×330. A sideways phone is only ~375px tall; scaling one
  column to fit drove the text to ~9px. **Scaling alone is not a responsive strategy** when a
  child has to read the result.

Two things stop this page rotting, and both should survive any refactor:

- **The icons call the game's own drawing functions** via `drawLegendIcon`, not redrawn
  copies. Restyle a sprite and the legend follows.
- **The values are printed from `GEM_VALUE` / `DONUT_VALUE` / `RAINBOW_VALUE` /
  `START_LIVES`**, not typed in. A legend claiming 50 while the code awards 60 is an easy lie
  to ship.

Hazards are drawn mid-warning (`warn: 0.55`) so the red glow is learnt there, not the first
time it costs a life.

---

## The leaderboard

The only network in the project. Supabase REST, called with plain `fetch` — **no Supabase JS
library**, so the no-dependencies rule holds.

```
project  kudr-leaderboard      table  public.scores
columns  id, initials, score, created_at
RLS      SELECT + INSERT only — no UPDATE or DELETE policy exists
```

With RLS on, an operation with no policy matches zero rows, so PATCH/DELETE return **204 with
nothing changed**. That looks like success and isn't — the board can be added to but never
edited or wiped with the public key. Constraints reject anything but `^[A-Z]{3}$` and a score
outside 0–1,000,000.

The publishable key sits in the HTML on purpose; that's how Supabase is designed. It lives in
its own project so it can never reach anything else. **Scores are forgeable** by anyone with
devtools — unavoidable without accounts, and accepted.

**Every call returns `null`/`false` on any failure rather than throwing.** `loadScores`,
`submitScore` and `qualifiesForBoard` all swallow errors and time out at 6s. The game must
survive a dead network mid-frame, so nothing in the loop may ever await or throw.

`qualifiesForBoard` returns **false when offline** — deliberately. Better to never ask for
initials than to ask and then fail to save them.

Periods are calendar-based, computed client-side in `periodStart()` and passed as a
`created_at=gte.` filter: month = the 1st, week = Monday. All-time sends no filter.

---

## The app icons

`icon-192.png`, `icon-512.png` and `apple-touch-icon.png` (180px) are **generated from canvas
2D code**, not hand-drawn, so they stay "art is code" like everything else. To redraw them,
paste this into the console on the running game, then save each data URL:

```js
function drawAppIcon(c, S) {
  const g = c.getContext('2d'), u = S / 512;
  g.fillStyle = '#0B2F52'; g.fillRect(0, 0, S, S);      // FLAT: a gradient
  g.save(); g.translate(S/2, S/2); g.scale(u, u);        // made the PNG 143KB
  g.fillStyle = '#15558F'; g.beginPath(); g.arc(0,0,198,0,Math.PI*2); g.fill();
  g.fillStyle = '#7CC6F0'; g.beginPath(); g.arc(0,0,150,0,Math.PI*2); g.fill();
  const bands = [[110,'#FF2D20'],[90,'#FF8A00'],[70,'#FFD400'],[50,'#22B34B'],[30,'#0A63C9']];
  g.lineWidth = 20; g.lineCap = 'butt';
  bands.forEach(b => { g.strokeStyle = b[1]; g.beginPath();
    g.arc(0, 55, b[0], Math.PI, Math.PI*2); g.stroke(); });
  const rim = ['#FF2D20','#FF8A00','#FFD400','#22B34B','#0A63C9','#A312C4'];
  g.lineWidth = 17; g.lineCap = 'round';
  rim.forEach((col, i) => { g.strokeStyle = col; g.beginPath();
    g.arc(0, 0, 150, -Math.PI/2 + (i/6)*Math.PI*2, -Math.PI/2 + ((i+1)/6)*Math.PI*2);
    g.stroke(); });
  g.fillStyle = '#fff';
  g.beginPath(); g.ellipse(-64, -76, 32, 19, -0.61, 0, Math.PI*2); g.fill();
  [[-176,-58,16],[174,-32,13],[0,-192,14],[130,160,11]].forEach(p => {
    const [sx, sy, sz] = p; g.fillStyle = '#fff'; g.beginPath();
    g.moveTo(sx, sy-sz);
    g.lineTo(sx+sz*0.33, sy-sz*0.33); g.lineTo(sx+sz, sy);
    g.lineTo(sx+sz*0.33, sy+sz*0.33); g.lineTo(sx, sy+sz);
    g.lineTo(sx-sz*0.33, sy+sz*0.33); g.lineTo(sx-sz, sy);
    g.lineTo(sx-sz*0.33, sy-sz*0.33); g.closePath(); g.fill(); });
  g.restore();
}
const make = S => { const c = document.createElement('canvas');
  c.width = c.height = S; drawAppIcon(c, S); return c.toDataURL('image/png'); };
({ 512: make(512), 192: make(192), 180: make(180) });
```

**Keep the background flat.** A three-stop gradient produced a 143KB 512px PNG; flat colour
gives 40KB, because smooth gradients make every row unique and defeat PNG's row filters.

Getting the bytes to disk without pushing ~90KB of base64 through a conversation: serve the
project over `python3 -m http.server`, run a tiny local receiver that base64-decodes POST
bodies to files, and have the page POST each data URL to it. The page must be on a real
`http://` origin for that — the preview pane's `data:` URL cannot reach localhost.

---

## Hosting

GitHub Pages from `erikrocks/kitty-unicorn-game`, custom domain `kudr.eriksheridan.com`,
HTTPS enforced. **Push to `main` and it redeploys.**

Verify by bytes, not by the build API — `/pages/builds/latest` reports a stale commit sha and
will look like the deploy hung when it already shipped:

```bash
curl -sS -o /tmp/live.html "https://kudr.eriksheridan.com/?cb=$RANDOM"; diff /tmp/live.html index.html
```

Shell `$(...)` strips trailing newlines, so hashing a curl result in a variable and comparing
it to the file gives a false mismatch. Diff files, not captured strings.

---

## If you change X, check Y

| Change | Check |
|---|---|
| Any speed or acceleration | Every constant multiplied by it — per-second values are ~60× bigger than they look |
| A sprite's size | Its spawn margins and its hit box; big sprites clip screen edges |
| Underwater art | Judge it *through* the water layer, not in isolation |
| A button's position or size | It's one function used by both drawing and hit-testing — don't split them |
| A score constant | Nothing: the how-to page reads it. That's the point |
| Any visual change at all | A 375px-wide viewport, and landscape. Every bug found so far was mobile-only |

## Where the bodies are buried

- The preview pane snapshots the file as a `data:` URL — `reload()` won't show edits. Open a
  fresh preview.
- Freezing the scene for a screenshot: `update = () => {}; spawn = () => {}`, then push
  entities in by hand — and set each item's `y` as well as `baseY`, because `update()` is what
  normally derives `y`. A frozen item with only `baseY` renders at `NaN`.
- `main` is the known-good version. Work on a branch.
