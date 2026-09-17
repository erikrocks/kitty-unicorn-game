# CLAUDE.md — Kitty Unicorn Dolphin Rider

## What this is
A cute, gentle browser game built with my young daughter Vivian (7), who designed it. This
project is also how I'm learning Claude Code, so favor small, readable, incremental changes
over big clever ones, and explain what you changed in plain language.

## Golden rules (do not break these)
- Everything lives in ONE self-contained file: `index.html` (HTML + CSS + JS together).
- No build step, no npm, no frameworks, no external libraries, no separate asset files.
- Must run by just opening the file in a browser. **The GAME is fully offline** — the only
  network call in the project is the high-score board, and every one of those is wrapped so
  failure is normal: no scores shown, nothing submitted, gameplay untouched. If you add a
  feature that *requires* the network to play, that's a rule change to discuss first.
- All art is drawn with canvas 2D (plus a few emoji for scenery). No image files, no sprite
  sheets, no loaded fonts.
- Works with BOTH touch (tap/hold) and mouse (click/hold), plus Space/Enter on a keyboard.
- Kid-friendly: gentle pace, forgiving, bright and happy. No violence, nothing scary.
- Vivian's art and design choices are the heart of this. Don't "improve" the look or the
  feel without asking — fix bugs, add what's requested.

## The game
You play a "kitty unicorn" — a pink cat with a unicorn horn and ears — riding a dolphin
across the ocean. Press and hold to rise, release to dive; collect gems, donuts and hearts
while dodging spiky shells. Endless.

## Controls
- In water: press/hold (touch, mouse, or Space) → dolphin rises. Release → it sinks.
- In the air: you're ballistic — gravity brings you back down. Leaping out of the water and
  arcing through the sky is the signature move.
- The world auto-scrolls; the dolphin holds a fixed x position and only moves up/down.
- Enter or click starts / restarts the game.

## What's actually built (all working)
- **Hand-drawn hero**: the dolphin (gradient body, animated tail and flipper) ridden by
  Vivian's redesigned kitten — lavender, chibi proportions, a rainbow mane that drifts, a
  flower crown, a rainbow-striped horn, closed happy eyes, blushed cheeks and a paw that
  waves. Everything on the cat is outlined; that dark line is what makes it read as a
  drawing rather than soft blobs.
- **DO NOT REDRAW THE DOLPHIN.** It was attempted five times across one session and every
  version was worse — "closer", then "looks like a shark", then "worse". It already has what
  matters: a long snout, a rounded body, a friendly eye and a big smile curve. When someone
  says the hero looks wrong, change the cat. Isolate one animal at a time.
- **Two-zone physics** with a wavy animated water surface; sky above, ocean below.
- **Splash system**: 30 particles at each surface crossing.
- **Web Audio from scratch** — collect, donut, heart, hit and splash sounds, all synthesized
  in code (no audio files). Unlocked on first tap.
- **Collectibles**: gems (10 pts) spawn anywhere; donuts (50 pts), hearts (+1 life) and
  **rainbows (116 pts)** are **air-only**, so the valuable stuff costs you a leap. They bob
  and drift in.
- **The rainbow** is Vivian's design: a big rainbow inside a rainbow-rimmed bubble with a
  shine and twinkling sparkles. It's the rarest thing in the game (3% of items, rarer than a
  heart — about three per game, but a long run can have more) and the only one with its own
  sound, a five-note rising arpeggio. 116 because 16 is her lucky number. It gets wider spawn
  margins (`RAINBOW_MIN_Y`, `RAINBOW_MARGIN`) than the other sky items because the bubble is
  much bigger, so it stays fully on screen and clear of the waves.
- **Obstacles**: three kinds, all underwater, all costing a life. The **spiky shell** is the
  usual one (~76%); a **sea urchin** and a **jellyfish** turn up now and then (~12% each,
  about 2.6 of each per minute) — Vivian wanted them occasional, not constant. The jellyfish
  is her colour scheme: deep pink dome, darker tentacles. All three shimmer **red** for the
  last 600px (~1.7s), drawn by one shared `drawHazardShimmer()`.
- **Hit boxes are per-kind** (`OBSTACLE_HITBOX`), not one square for all three. The jellyfish
  is a narrow dome with long trailing tentacles, so a wide box would sting you through empty
  water at its sides; its box is also shifted down (`oy`) to cover the tentacles. Don't
  collapse these back into a single box.
- **Seaweed**: background scenery rooted in the seafloor — bubble weed (beaded stems) and
  leafy clumps, mixed, swaying gently. It scrolls at `SCROLL_SPEED` to stay locked to the
  sand, and `updateSeaweed()` is called from `loop()` rather than `update()` on purpose:
  `update()` stops on the start screen but the floor keeps sliding, and frozen weeds growing
  out of moving sand looks broken.
- **Difficulty ramps.** The game used to run at one speed forever, so a good run became a
  marathon. `difficulty()` goes 0 → 1 over `DIFFICULTY_RAMP` (150s) and then **plateaus** —
  scroll and hazard speed reach `MAX_SPEED_MULT` (1.7x), hazard spawn rate reaches
  `MAX_HAZARD_MULT` (2x). Tune those three constants to change the whole escalation.
  `warnDistance()` scales with speed so the red shimmer always gives ~1.7s of warning;
  without that, the warning shrinks exactly when you need it most.
- **The seafloor position is ACCUMULATED (`worldScroll`), never `elapsed * SCROLL_SPEED`.**
  Multiplying total elapsed time by the current speed makes the floor lurch the moment the
  speed can change. Anything that scrolls must integrate distance, not recompute from `elapsed`.
- **x2 bonus at 6 hearts** (Vivian's idea): `scoreMultiplier()` doubles gems, donuts and
  rainbows — not hearts, which still give a life. Shown as a pulsing gold badge next to the
  hearts, sized from the MEASURED text width. Hearts past `HEART_DISPLAY_MAX` (8) collapse to
  `❤️ xN` so a long row can't run off a phone screen.
- **Personal best** in `localStorage` (`kudr.best`), shown on the start and game-over screens.
  Separate from the online board on purpose — something to beat on every run, not just the
  ones good enough for the top ten. Reads/writes are wrapped in try/catch because
  `localStorage` *throws* in some privacy modes rather than returning nothing.
- **Lives (3 to start) and a GAME OVER screen** with the final score and a **Try Again
  button** you have to actually hit — tapping anywhere used to wipe the score before you'd
  read it. Enter still works.
- **Start screen** with the title, Vivian's credit, **How to play** and **High scores**
  buttons side by side, and a blinking prompt. Tapping a button opens that page; tapping
  anywhere else still just starts the game.
- **High scores** (`LEADERBOARD` + `ENTER_INITIALS` states), backed by Supabase. Traditional
  3 initials, with All time / This month / This week tabs. Periods are **calendar** periods
  (week starts Monday), not rolling windows, so everyone's board resets together. You're only
  asked for initials if you beat the 10th all-time score — and never if the server is
  unreachable, so the game can't ask for initials it then fails to save.
- **How-to-play page** (`HOWTO` state): explains the physics (hold to swim up, and that
  dolphins can't fly so you have to jump), then lists every collectible with its value and
  every hazard. Two things keep it honest: the icons are drawn by calling the game's OWN
  drawing functions, and the values are printed from `GEM_VALUE` / `DONUT_VALUE` /
  `RAINBOW_VALUE` / `START_LIVES`. Restyle a sprite or change a score and the page follows —
  don't replace either with hand-drawn copies or typed-in numbers.
  The hazards are drawn mid-warning (`warn: 0.55`) so you learn the red glow here rather than
  the first time it costs a life. The hero is deliberately NOT drawn behind this page — she
  bobs at screen centre, right where the icon legend sits.
- Clouds, a scrolling sandy seafloor, and a responsive HUD that scales to the screen.
- **The dolphin's x position is responsive** (`playerHomeX()`): a quarter of the screen
  width, capped at 200px, floored at 90px. On a phone a fixed 200px was over half the screen
  and you couldn't see what was coming.

## Design rules learned from playtesting (don't undo these)
- **Hazards must not look shiny.** A gold/white sparkle on the shells read as *treasure* and
  made players swim toward them. Danger is red; rewards are bright/cyan/pink.
- **Judge underwater art THROUGH the water layer.** The translucent water is painted over
  everything below the surface, so it shifts colours a lot — a pale pink jellyfish reads as
  lavender. I once flagged the jellyfish as looking too much like the pink donut; it doesn't,
  because the jellyfish is always tinted and the donut is always in the untinted sky. Judging
  an underwater sprite in isolation gives the wrong answer.
- **Full-screen pages need a landscape layout, not just smaller text.** A phone held
  sideways is only ~375px tall. Shrinking the how-to page to fit drove its text to ~9px, so
  it switches to two columns instead (`howToLayout().wide`). Scaling alone is not a
  responsive strategy when a child has to read the result.
- **Tap targets need a floor that does NOT scale with the layout.** Scaled pages shrink
  their buttons along with the text; a Back button at `46 * scale` became 30px in landscape.
  Text may shrink, touch targets may not — use `Math.max(floor, natural * scale)`. And anchor
  what follows a floored element to its ACTUAL rect, not to the natural grid, or the floor
  pushes it into the next thing.
- **Draw optional text only when it fits.** The game-over "(or press Enter)" hint fell off
  the bottom on short landscape screens — it now checks `hintY <= viewH - 6` first. That's
  more robust than another viewport threshold, and a phone held sideways has no keyboard
  anyway.
- **Buttons need a floor of 48px, not 42.** Height-based sizing bottoms out in landscape and
  silently lands under the 44pt minimum tap target.
- **Test at 375px wide.** Every bug Vivian and Erik have hit was mobile-only and invisible on
  a desktop viewport — the dolphin sitting too far right, the game-over misfire, and a wedge
  of missing water caused by a loop that stepped in 20px jumps (1280 divides by 20; 375
  doesn't). Check a phone width before calling any visual change done.

## Code style
- Code is organized into labeled sections: SETUP, INPUT, UPDATE, DRAW, MAIN LOOP.
- Animate with `requestAnimationFrame`. **All movement is frame-rate independent**: every
  speed is expressed per SECOND and multiplied by `dt`. Never write a per-frame value like
  `x -= 5` — it makes the game run at double speed on a 120Hz screen.
  - Gotcha: because speeds are per-second they're large (~1000). Any constant multiplied by
    a speed must be scaled to match (this is what caused the dolphin to somersault once).
- The canvas is scaled by `devicePixelRatio` so art is sharp on Retina screens. Game logic
  uses `viewW`/`viewH` (CSS pixels), never `canvas.width`/`canvas.height` (device pixels).
- Tuning values (scroll speed, gravity, rise force, spawn rates, tilt) are named constants
  in one block at the top of the script.
- Prefer simple, readable code a beginner can follow over clever optimizations.
- Add short comments explaining the "why," not just the "what."

## How the code actually works
See **[ARCHITECTURE.md](ARCHITECTURE.md)** — draw order, the delta-time and `viewW`/`viewH`
invariants, spawn tables, collision boxes, and an "if you change X, check Y" table. Read it
before any non-trivial change; this file is the rules, that one is the mechanics.

## Working on this file
- **`index.html` needs its `<meta charset="utf-8">`.** GitHub Pages sends `charset=utf-8` in
  its headers, which masked the fact that the file never declared one. Serve it from anything
  else (`python3 -m http.server`, say) and every emoji mangles — the hearts in the lives
  counter render as `a ¤i`. Don't remove that tag.
- **The preview pane snapshots `index.html` as a `data:` URL.** `navigate` and
  `location.reload()` will NOT pick up your edits — close the tab and open a fresh preview,
  or you'll screenshot stale art and think a change didn't apply.
- Handy when checking art: freeze the scene with `update = () => {}; spawn = () => {}`, then
  push items/obstacles in by hand. Set each item's `y` as well as `baseY` — `update()` is
  what normally derives `y`, so a frozen item with only `baseY` renders at `NaN`.

## Installable web app (PWA)
`manifest.webmanifest` + `icon-192.png` / `icon-512.png` / `apple-touch-icon.png` sit beside
`index.html`. That's the same layout `erikrocks/bubble` uses, and it's why the one-file rule
still holds: the rule is about the GAME having no build step or loaded assets, not about the
repo containing nothing else.

- **No service worker, and none is needed.** Chrome dropped that requirement; bubble installs
  without one. Don't add one to "fix" installability.
- **The icons are generated from canvas 2D code**, not hand-drawn files — the same drawing
  API as the game's art. The generator is recorded in ARCHITECTURE.md so they can be redrawn.
- **The icon background is flat on purpose.** A gradient pushed the 512px PNG from 40KB to
  143KB, because smooth gradients defeat PNG compression.
- iOS status bar is `default`, not `black-translucent`: the top of the screen during play is
  pale sky blue, and a translucent bar draws its clock in white on top of it.

## Hosting — DONE, live
**<https://kudr.eriksheridan.com>** — GitHub Pages from `erikrocks/kitty-unicorn-game`,
custom domain set, HTTPS enforced, DNS `CNAME kudr → erikrocks.github.io` in place.
**Push to `main` and it redeploys.** Nothing else to configure.

Why a subdomain and not `eriksheridan.com/kudr`: the apex is served by a *project* repo
(`erikrocks/eriksheridan.com`) — there is no `erikrocks.github.io` user-site repo. A custom
domain on a project repo serves ONLY that repo, so a path pointing at a different repo always
404s. Don't retry the path approach; it cannot work without restructuring the whole site.

**Verifying a deploy:** the Pages builds API (`/pages/builds/latest`) reports a *stale commit
sha* — polling it will look like the deploy hung when it already shipped. Check the actual
bytes instead:
```
curl -sS -o /tmp/live.html "https://kudr.eriksheridan.com/?cb=$RANDOM"; diff /tmp/live.html index.html
```

Separately: `erikrocks/eriksheridan.com` now has **Enforce HTTPS on** (flipped Sep 2026),
and the apex is no longer a resume — it's a menu linking to this game, EBAAPL and
`resume.eriksheridan.com`. Full architecture lives in that repo's `CLAUDE.md`.

## Roadmap (later — one at a time, don't build ahead)
1. ~~Gem types worth different amounts~~ — done (gem 10 / donut 50 / heart +1 life).
2. Customization shop: spend points on kitty outfits/colors and dolphin colors.
   (`// TODO:` hooks in `drawHero()` and `draw()`.)
3. Dolphin upgrades: colors or a horn that affect speed or jump height.
4. Save progress (unlocks) to `localStorage` — the personal best already lives there.

~~High scores~~ — done. Supabase project `kudr-leaderboard`, table `public.scores`, RLS
allows SELECT and INSERT only (verified: PATCH and DELETE match zero rows). The publishable
key in `index.html` is meant to be public. **Free-tier Supabase pauses after ~7 days with no
activity** — if the board stops working, check whether the project needs unpausing.

Dropped: an "underwater section" the dolphin dived into. It was one unexplained line in the
original roadmap, Vivian never asked for it, and the ocean is only ~300px deep so there was
nowhere to dive to. Don't reintroduce it without her asking.

## How I like to work
- I'm learning, so after a change, tell me how to run/test it and what to look for.
- When you finish a feature, offer a one-line "next we could…" suggestion, but don't build it
  until I ask.
- Keep it concrete — I'd rather get the file running than read a long explanation.
- `main` holds the last known-good version. Do work on a branch so we can always get back.
