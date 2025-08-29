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
  - `Guide`: toggle aiming guide on/off
  - `Options`: open the Options panel

## Options Panel
- `Guide bounces`: choose 1, 2, or 3. The guide simulates up to that many peg hits. The breadcrumb fades and becomes barely visible near the final predicted peg.

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
