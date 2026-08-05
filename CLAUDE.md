# CLAUDE.md — Kitty Unicorn Dolphin Rider

## What this is
A cute, gentle browser game built with my young daughter Vivian (7), who designed it. This
project is also how I'm learning Claude Code, so favor small, readable, incremental changes
over big clever ones, and explain what you changed in plain language.

## Golden rules (do not break these)
- Everything lives in ONE self-contained file: `index.html` (HTML + CSS + JS together).
- No build step, no npm, no frameworks, no external libraries, no separate asset files.
- Must run by just opening the file in a browser, fully offline.
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
- **Hand-drawn hero**: gradient dolphin with an animated tail and flipper, ridden by a pink
  cat with a gradient horn, ears, whiskers and a waving paw. This is the best part of the game.
- **Two-zone physics** with a wavy animated water surface; sky above, ocean below.
- **Splash system**: 30 particles at each surface crossing.
- **Web Audio from scratch** — collect, donut, heart, hit and splash sounds, all synthesized
  in code (no audio files). Unlocked on first tap.
- **Collectibles**: gems (10 pts) spawn anywhere; donuts (50 pts) and hearts (+1 life) are
  **air-only**, so the valuable stuff costs you a leap. They bob and drift in.
- **Obstacles**: animated spiky shells underwater; hitting one costs a life. They shimmer
  **red** for the last 600px (~1.7s) as a warning.
- **Lives (3 to start) and a GAME OVER screen** with the final score and a **Try Again
  button** you have to actually hit — tapping anywhere used to wipe the score before you'd
  read it. Enter still works.
- **Start screen** with the title, Vivian's credit, and a blinking prompt.
- Clouds, a scrolling sandy seafloor, and a responsive HUD that scales to the screen.
- **The dolphin's x position is responsive** (`playerHomeX()`): a quarter of the screen
  width, capped at 200px, floored at 90px. On a phone a fixed 200px was over half the screen
  and you couldn't see what was coming.

## Design rules learned from playtesting (don't undo these)
- **Hazards must not look shiny.** A gold/white sparkle on the shells read as *treasure* and
  made players swim toward them. Danger is red; rewards are bright/cyan/pink.
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

Separately: `erikrocks/eriksheridan.com` still has **Enforce HTTPS off** — unrelated to this
game, but worth flipping.

## Roadmap (later — one at a time, don't build ahead)
1. ~~Gem types worth different amounts~~ — done (gem 10 / donut 50 / heart +1 life).
2. Underwater section the dolphin enters when it dives deep. (`// TODO:` hook in `update()`.)
3. Customization shop: spend points on kitty outfits/colors and dolphin colors.
   (`// TODO:` hooks in `drawHero()` and `draw()`.)
4. Dolphin upgrades: colors or a horn that affect speed or jump height.
5. Save progress (points + unlocks) to `localStorage`.

## How I like to work
- I'm learning, so after a change, tell me how to run/test it and what to look for.
- When you finish a feature, offer a one-line "next we could…" suggestion, but don't build it
  until I ask.
- Keep it concrete — I'd rather get the file running than read a long explanation.
- `main` holds the last known-good version. Do work on a branch so we can always get back.
