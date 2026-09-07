# Handoff: Trip Muncher — Retro Home-Computer Browser Game

## Overview
A playable browser game recreating the home-computer scene from the film "Die Schrillen Vier auf Achse" (National Lampoon's Vacation-style road trip). The player sabotages the family's cross-country road trip: a monster chases and eats the family station wagon across a stylized US map, stage by stage (city to city), before the wagon reaches its destination. Presented inside a retro CRT monitor + beige home-computer chassis ("OKTA-80").

## About the Design Files
The file in this bundle (`Trip Muncher.dc.html`) is a **design reference created in HTML** — a working prototype showing intended look and behavior, not production code to copy directly. The task is to **recreate this game in the target codebase's existing environment** (React, Vue, plain Canvas/TS, game framework, etc.) using its established patterns — or, if no environment exists yet, choose an appropriate stack (a single `<canvas>` + requestAnimationFrame loop is entirely sufficient; no game engine needed).

The prototype uses a proprietary component format: the markup lives between `<x-dc>` tags and the game logic in a `class Component` inside the `<script data-dc-script>` block. All game logic is plain JavaScript on a 2D canvas — it ports directly.

## Fidelity
**High-fidelity.** Colors, layout, sprite shapes, copy, timings, and game balance are final and were matched to screenshots of the actual film scene. Recreate pixel-perfectly.

## Architecture Summary
- One logical canvas: **280 × 192 px**, upscaled with `image-rendering: pixelated` to fill the CRT screen area (aspect-ratio 280/192).
- Fixed game loop via `requestAnimationFrame` with delta-time (clamped to 50 ms).
- A simple mode/state machine: `boot → title → intro → play → (chomp | fail) → … → gameover | win`.
- All rendering is immediate-mode canvas 2D; text via the Google font **VT323** (load before first draw).
- Sound: Web Audio API square/triangle/sawtooth beeps, created lazily on first user gesture.

## Screens / Views (canvas modes)

### 1. Boot ("boot")
- Black screen, teletype effect (~42 chars/sec) printing:
  `OKTA-80 PERSONAL COMPUTER`, `64K RAM   SYSTEM READY`, blank, `]LOAD "VACATION"`, `]RUN`
- 13px VT323, left-aligned at x=10, 15px line height, blinking block cursor (7×10 px, 3 Hz).
- Auto-advances to title; Space/tap skips.

### 2. Title ("title")
- Filled US map (see Design Tokens) with all 5 dotted route legs and destination marker.
- "TRIP MUNCHER" 22px centered at y=16; subtitle "THE VACATION SABOTAGE GAME" 11px.
- Hi-score top right (persisted in `localStorage` key `tripMuncherHi`).
- Red banner strip (y=155, height 20) alternating every ~0.8s between `TOTAL MILES 2408` and `PRESS SPACE OR TAP TO START` (white 14px text).
- Footer: `(C)1983 OKTA SOFT INC.` 10px, 55% alpha.

### 3. Stage intro ("intro", 2.4 s)
- Same US map; the current leg's dots render brighter/larger (3px orange vs 2px cream).
- HUD: `DAY n/5` left, `WEAPON: MUNCHER|SPIDER` right.
- Banner: `DAY n: <FROM> TO <TO>`. After 1.4 s a blinking `GO!` (20px).

### 4. Gameplay ("play") — zoomed view
- Black background with decorative jagged "state border" polylines: 3 horizontal + 2 vertical, drawn twice — 3px orange (#c05a20, 80% alpha, offset +1px y) under 1.6px cream (#f8f0dc). Generated per-stage from a seeded PRNG (sin-hash) so retries look identical.
- The wagon follows a fixed polyline route (per stage), normalized/zoomed to fit a 200×116 box centered at (140,104), leaving an orange dotted trail every 8 route-px (3×3 px squares).
- Destination city: 5×5 px marker + city name label (10px) at route end.
- HUD (12px): `SCORE 000000` left, `DAY n/5` center, `TRIES n` right, at y=9.
- **Wagon** (green #41d98c): 18×4 body, 9×3 cabin, 4×1 hood stripe, two 3×2 wheels; flips horizontally with travel direction. Waits 0.9 s grace before moving. Blinks (8 Hz) + orange `!` above when player is within 42 px (panic: speed ×(1.35+0.09·stage)).
- **Player, days 1–3 — Muncher** (red-orange #d94f28): 14×14 square with a wedge mouth cut toward heading (kerb-cut pentagon), mouth oscillates 14 Hz while moving; speed 78 px/s; eats trail dots within 9 px (+5 pts each, alternating 640/480 Hz blips); catches wagon within 11 px.
- **Player, days 4–5 — Spider** (orange #d96b3a): 45°-rotated 10×10 diamond body, 3 zigzag legs per side animating at 8 Hz (2×2 px segments), dark center pixel; speed 70 px/s; auto-fires a 4×4 bullet (#e8a060, 150 px/s) every 0.5 s in facing direction; bullet hit within 9 px of wagon.
- Wagon base speeds per stage: 30, 36, 42, 47, 52 px/s × difficulty (kids 0.78 / normal 1 / hard 1.18).

### 5. Catch ("chomp", 1.6 s)
- Muncher: wagon blinks out (<0.5 s), muncher chomps at its position; banner `CHOMP!!`.
- Spider: pixel explosion burst (14 2×2 particles radiating, radius 3–10 + t·14) at wagon; banner `DIRECT HIT!!`.
- Score: `100·(stage+1) + floor(remainingRouteLength)`; descending jingle 520/380/260/160 Hz.

### 6. Fail ("fail")
- Playfield stays; wagon shown at route end; banner `THE WAGON MADE IT TO <CITY> - TRIES n`; blinking retry prompt. Descending sad jingle 300/240/180/120 Hz.

### 7. Game over / Win
- Both over the US map + banner. Game over: `GAME OVER`, final score, banner `DAD DROVE ALL 2408 MILES. GROSS.`
- Win: `ROUTE 100% EATEN`, `RUSTY 1 - DAD 0`, banner `VACATION CANCELLED FOREVER`, muncher marching across at y=100.

## Route Data (logical 280×192 coordinates)
Cities: CHICAGO [168,70] → ST.LOUIS [160,98] → DODGE CITY [118,104] → SOUTH FORK [90,94] → GRAND CANYON [62,106] → FUNLAND PARK [36,124].
Route legs (polylines, exact):
1. [168,70],[190,76],[194,96],[172,106],[160,98]
2. [160,98],[150,120],[128,128],[112,116],[118,104]
3. [118,104],[132,88],[112,74],[94,72],[78,86],[90,94]
4. [90,94],[104,112],[84,124],[64,120],[48,104],[62,106]
5. [62,106],[80,120],[60,132],[40,136],[26,120],[36,124]

US outline polygon (33 points, closes on itself): [22,72],[14,92],[16,112],[24,132],[38,158],[52,158],[66,164],[88,156],[104,164],[112,178],[120,160],[140,164],[158,158],[178,162],[196,152],[206,164],[214,184],[220,166],[212,152],[228,142],[238,120],[244,100],[236,88],[228,84],[232,72],[244,58],[228,54],[196,60],[160,50],[120,54],[84,50],[52,56],[22,72]. Fill #2444d0, then clipped grid lines #6b84ec at 55% alpha (horizontal every 16px from y=64, vertical every 26px from x=40).

## Interactions & Behavior
- **Keyboard**: Arrows/WASD steer (8-way, normalized). Space/Enter advances mode & starts. M toggles mute. Arrow keys + Space call preventDefault.
- **Touch/Pointer**: pointerdown on canvas = advance (menus) or set steer target (play); pointermove drags target; player steers toward target while > 5 px away. `touch-action: none` + pointer capture.
- Muncher "waka" sound: alternating 260/190 Hz triangle blips every 0.16 s while moving (days 1–3).
- Player bounds: x 10–270, y 24–186.

## State Management
Mode machine as above; per-stage state: path (points + cumulative lengths for arc-length interpolation), carDist, eaten-dot set, bullets array, player pos/angle/mouth, panic flag. Persistent: hi-score in localStorage. Tries: 3 per game; fail decrements, 0 = game over.

## Design Tokens
Colors (color mode): bg #070503 · map fill #2444d0 · map grid #6b84ec · state borders #f8f0dc on fringe #c05a20 · trail/accent #e06a30 · wagon #41d98c · muncher #d94f28 · spider #d96b3a · bullet #e8a060 · HUD text #f0e8d8 · banner #a82418 with #ffffff text · route dots #f0e0c8.
Monochrome modes (settings): green ink #4dff6a on #051005; amber #ffb000 on #0c0903 — single-ink rendering, map outline only (no fill), banner rgba(255,255,255,.14).
Font: VT323 (Google Fonts), sizes 9–24px logical. All sprites axis-aligned filled rects (no anti-aliasing intent).

### CRT chassis (DOM, outside canvas)
- Beige case: linear-gradient(175deg, #ddd5c4, #c9c0ac 55%, #b3a992), radius 18/18/26/26, inset highlights + drop shadow.
- Screen well: #2a2620, radius 12, deep inset shadow; screen bg #060806, radius 10.
- Overlays: scanlines (repeating-linear-gradient, 1px black 28% every 3px), vignette (radial, 55%→black 55%), diagonal glare (linear 115deg, white 6%→22%), subtle flicker keyframes (opacity .90–.98, 3.7 s).
- Badge row: "OKTA-80" + "HOME COMPUTER SYSTEM", two fake knobs, pulsing green PWR LED (#86e01e).
- Page bg: radial gradient #221d17 → #131009; help line in #8a8170 VT323 17px.

## Settings (exposed as tweakable props in the prototype)
- `screen`: "color" | "green" | "amber" (default color)
- `scanlines`: boolean (default true)
- `difficulty`: "kids" | "normal" | "hard" (default normal) — multiplies wagon speed 0.78/1/1.18

## Assets
None. Everything is drawn procedurally (canvas rects/paths) + one Google Font (VT323). No copyrighted film assets are used; names ("Rusty", "Funland Park") are generic homage copy.

## Files
- `Trip Muncher.dc.html` — the complete prototype (markup + all game logic in one file). Game logic lives in the `class Component` script block; template/DOM chrome between `<x-dc>` tags.
