# Peggle — Single‑File Build

A polished, single‑file Peggle‑style game you can open directly in any modern browser. No build, no network — just play.

## Overview
- Fixed‑step physics at 120 Hz with dynamic substeps for fast shots
- Aiming guide with up to 3 predicted peg bounces and fade‑out at final touch
- Clean visuals: glow pegs, trail, particles, responsive canvas, HUD
- Moving bucket with catch sensor (+1 ball)
- Local, deterministic RNG for fresh layouts on restart

## Play
- Open `peggle.html` in Chrome, Edge, Firefox, or Safari
- The canvas scales to your window; no server required

## Controls
- Aim: mouse
- Shoot: mouse click or `Space`
- HUD buttons:
  - `Restart`: new seed/layout
  - `Options`: open the Options panel (or press `O`)

## Options Panel
- `Guide` On/Off: turns the aiming guide on or off. When Off, bounce numbers are disabled.
- `Guide bounces`: choose 0–4. Click the active number again to clear it.
- `Guide length`: prediction duration in milliseconds (800–3000).
- `Guide density`: breadcrumb dot spacing; smaller is denser.
- `Guide fade`: start and end alpha for breadcrumb visibility.
- Physics tweaks:
  - `Gravity`: adjust fall speed.
  - `Restitution`: ball, peg, and wall bounce energy.
  - `Air drag`: velocity damping per step.
  - `Time step`: simulation rate (60–144 Hz).
  - `Launch speed`: shot velocity.
  - `Ball radius`: size affecting collisions and visuals.
  - `Balls per game`: starting ball count.
  - `Bucket speed`: horizontal sweep of the catch bucket.
  - `Bucket size`: width and height of the bucket (edge margin is 24 px).
  - `Columns` / `Rows`: peg grid dimensions.
  - `Peg spacing` and `Peg radius`: layout density controls.
  - `Start Y`: vertical offset of the peg grid.
  - `Target ratio`: percentage of pegs marked as targets.
  - `Seed`: set a deterministic layout seed; `Restart` still generates a random seed.
  - `Normal peg score` and `Target peg score`.
  - `Streak step`: multiplier increase per consecutive hit.
  - `Catch bonus`: award an extra ball, points, or both when the bucket catches the ball.
  - `Particle density`, `Particle speed`, `Particle life`, and `Particle gravity`.
  - `Trail length`: how long the shot trail persists.
  - `DPR cap`: limit device pixel ratio for performance.
  - `Color blind palette`: alternate peg colors.
  - `Preset`: quickly apply predefined playstyle settings.

## Notes
- The guide uses the same integration order, wall handling, restitution, and substep logic as gameplay for high fidelity.
- Score increases per peg and scales with a hit streak. Target pegs are required for a win.
- Catching the ball in the bucket grants an extra ball.

## Known Limits (V1)
- No audio or save system
- Single integrated HTML file for portability; assets are procedural

## Development
- Everything lives in `peggle.html`. Edit and refresh.
- Suggested next features for Options: audio toggle, particle density, color‑blind palettes, and DPR cap.

Enjoy!
