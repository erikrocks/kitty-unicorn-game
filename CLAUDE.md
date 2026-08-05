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
- **Collectibles**: gems (10 pts), donuts (50 pts), hearts (+1 life). They bob and drift in.
- **Obstacles**: animated spiky shells underwater; hitting one costs a life.
- **Lives (3 to start) and a GAME OVER screen** with the final score and a restart prompt.
- **Start screen** with the title, Vivian's credit, and a blinking prompt.
- Clouds, a scrolling sandy seafloor, and a responsive HUD that scales to the screen.

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

## Hosting
Published with GitHub Pages from `erikrocks/kitty-unicorn-game`, live at
<https://erikrocks.github.io/kitty-unicorn-game/>. Pushing to `main` redeploys it.

**Target URL: `kudr.eriksheridan.com`.** Why not `eriksheridan.com/kudr`: the apex domain is
served by a *project* repo (`erikrocks/eriksheridan.com`) rather than a user site — there is
no `erikrocks.github.io` repo. When a custom domain sits on a project repo it serves ONLY
that repo, so a path pointing at a different repo always 404s. A subdomain is the fix, and it
keeps the game in its own repo where `main` → push → live already works.

Setup order matters: **add the DNS record first.** Setting the custom domain in GitHub before
DNS resolves makes the working `erikrocks.github.io` URL redirect to a dead hostname.
1. DNS: `CNAME` record, host `kudr`, target `erikrocks.github.io.`
2. Then repo Settings → Pages → custom domain `kudr.eriksheridan.com`, and tick Enforce HTTPS
   once the certificate issues (~15 min).

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
