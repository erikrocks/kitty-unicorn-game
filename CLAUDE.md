# CLAUDE.md — Kitty Unicorn Dolphin Rider

## What this is
A cute, gentle browser game I'm building with my young daughter. This project is also how
I'm learning Claude Code, so favor small, readable, incremental changes over big clever ones,
and explain what you changed in plain language.

## Golden rules (do not break these)
- Everything lives in ONE self-contained file: `index.html` (HTML + CSS + JS together).
- No build step, no npm, no frameworks, no external libraries, no separate asset files.
- Must run by just opening the file in a browser, fully offline.
- All art is emoji + canvas 2D drawing. No image files, no sprite sheets, no loaded fonts.
- Works with BOTH touch (tap/hold) and mouse (click/hold).
- Kid-friendly: gentle pace, forgiving, bright and happy. No violence, no scary game-over.

## The game
You play a "kitty unicorn" — a cat with a unicorn horn and unicorn ears — riding a dolphin
across the ocean. Core loop: press and hold to rise, release to dive; fly through gems to
collect them. Endless and relaxing.

## Controls
- Press/hold (touch or mouse) → dolphin rises.
- Release → dolphin dives (gravity pulls it down).
- Cap max up/down speed so it feels smooth, not twitchy.
- The world auto-scrolls at a steady pace; the dolphin holds a fixed x position and only
  moves up/down.

## Current status
Starting from scratch. First goal is a working v1 of the CORE loop only (see v1 scope). Do
NOT build the shop or underwater section yet — but leave clean, commented `// TODO:` hooks
where they'll go, so adding them later is easy.

## v1 scope (build this first)
- Wavy animated water surface: sky above, water below; the dolphin can leap above the surface
  and dive below it.
- Dolphin (🐬) with the kitty-unicorn rider drawn on top: a small cat face (🐱) plus a unicorn
  horn and ears. Compositing emoji with drawn canvas shapes is fine.
- Gems (💎) spawn ahead and drift in; flying through one collects it and increases a gem
  counter shown in a corner.
- A start screen ("Tap to play!") and the visible gem counter.
- No game-over — endless and forgiving is perfect for a young kid.

## Code style
- Organize the code into clearly labeled, commented sections: SETUP, INPUT, UPDATE, DRAW.
- Animate with `requestAnimationFrame`; make ALL movement frame-rate independent (use delta
  time), so it runs the same on fast and slow devices.
- Prefer simple, readable code a beginner can follow over clever optimizations.
- Keep tuning values (scroll speed, gravity, rise force, gem spawn rate) as named constants
  near the top of the script so they're easy to find and tweak.
- Add short comments explaining the "why," not just the "what."

## Roadmap (later — one at a time, don't build ahead)
1. Gem types: kitty, unicorn, and puppy worth different amounts (e.g. 1 / 3 / 5).
2. Underwater section the dolphin enters when it dives deep.
3. Customization shop: spend gems on kitty outfits/colors and dolphin colors.
4. Dolphin upgrades: colors or a horn that affect speed or jump height.
5. Save progress (gems + unlocks) to `localStorage`.

## How I like to work
- I'm learning, so after a change, tell me how to run/test it and what to look for.
- When you finish a feature, offer a one-line "next we could…" suggestion, but don't build it
  until I ask.
- Keep it concrete — I'd rather get the file running than read a long explanation.
