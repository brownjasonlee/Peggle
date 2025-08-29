# Peggle-like Game PRD
**Project**: Peggle-like Arcade Physics Game  
**Engine**: Phaser 4 + Matter.js  
**Language**: TypeScript  
**Version**: 1.1  
**Date**: 2025-08-21  
**Status**: Draft

## 1. Purpose
Deliver a silky smooth Peggle-like experience that feels clean and professional. Build a polished, deterministic 2D physics game that runs great on desktop and mobile browsers, with optional desktop packaging. The first release will be a focused vertical slice that proves core feel, performance, and replayability.

## 2. Goals and Non-Goals
### 2.1 Goals
- Single player only. No accounts, no login, no online services.
- Hit a stable 60 fps on mid hardware, with 120 fps on high refresh where available.
- Frame pacing that feels smooth. 99.9th percentile frame time under 16.7 ms on a mid laptop.
- Tight input response. Button or click to on-screen action under 60 ms on desktop and under 80 ms on mobile.
- Deterministic physics for saved replays and ghost runs.
- One click web share. Export a WebAssembly or JS build that loads quickly.
- A short, satisfying campaign. Ten handcrafted levels plus a score attack mode.
- Professional presentation. Strong audio, tasteful particles, responsive UI, and subtle camera work.

### 2.2 Non-Goals
- No multiplayer of any kind.
- No login or account system.
- No free to play monetization in the first release.
- No network features such as online leaderboards in the first release.
- No complex character progression or inventory.
- No procedural soundtrack generator. Precomposed stingers only.

## 3. Target Audience and Personas
- **Casual arcade player**. Plays short sessions, values spectacle and ease of use.
- **Score chaser**. Replays levels for high scores, cares about precision and determinism.
- **Mobile commuter**. Short plays, touch first, battery sensitive, quick resume.

## 4. Platforms and Devices
- Primary target: modern desktop browsers with WebGL 2. Chrome, Edge, Firefox, Safari.
- Secondary target: iOS Safari and Android Chrome.
- Optional: Desktop packaging using Tauri for Windows and macOS.
- Resolution: responsive layout. 16 by 9 as baseline, supports 4 by 3 and ultrawide. DPR capped to control overdraw.

## 5. Success Metrics
- Technical: p99.9 frame time under 16.7 ms on a 4 core laptop with integrated graphics. Input latency budget under 60 ms end to end on desktop.
- Engagement: average session length at least 8 minutes by week 2. Retention day 1 at least 25 percent.
- Quality: crash free sessions at or above 99.8 percent. No reported soft lock defects after week 4 of beta.
- Content: 10 levels shipped with 2 that users replay over 5 times on average.
- Feel: internal playtest NPS 50 or higher. Subjective feel checklist passed.

## 6. Game Overview
### 6.1 Core Loop
Aim. Shoot a ball into a field of pegs. Hit target pegs. Catch ball with a moving bucket for a bonus. Clear the objective to finish the level. Replay to improve score and medals.

### 6.2 Modes
- Campaign. Ten handcrafted levels with increasing difficulty and new peg types.
- Score attack. One level rotation that tracks **device local** daily and all time scores.
- Practice. Sandbox with physics visualization for QA and tuning.

### 6.3 Progression
- Stars per level based on score thresholds.
- Cosmetic badges for perfect clears and no aim assist runs.
- No power creep. Score and mastery drive replay.

## 7. Gameplay Requirements
### 7.1 Aiming and Shooting
- Mouse or right stick aims a guide arc that extends 25 to 40 percent of the shot path. The guide stops at the first predicted collision.
- Adjustable aim sensitivity per input device. Toggle for guide arc off for advanced players.
- One ball on screen during normal play. Queueing is allowed during end of level fanfare but shots do not spawn until fanfare ends.

### 7.2 Pegs and Collisions
- Round pegs in a grid and some special placements. All pegs are circles in physics.
- Peg states: unlit, lit on hit, cleared when scoring is tallied. Special pegs may persist.
- Peg types:
  - Normal peg. Base score and clears on hit.
  - Target peg. Required for level completion. Distinct color and emissive edge.
  - Multiplier peg. Temporarily doubles score for the next N hits. Default N equals 5.
  - Power peg. Triggers power up listed in section 7.4.
- Each level defines count and placement of target pegs and specials.

### 7.3 Bucket Mechanics
- One bucket moves horizontally on a track at constant speed with easing at edges.
- Catching the ball grants one extra ball and a small score bonus. A catch pauses the despawn and resets the next shot sooner.
- Bucket is a kinematic body driven in the physics step to ensure no tunneling.

### 7.4 Power Ups
- Super shot. Next shot has increased restitution and light homing toward the nearest target peg within a small cone.
- Splinter. On first impact, spawns two short lived sub balls that inherit fractional velocity. Sub balls score at 25 percent value and do not grant extra balls on catch.
- Time stretch. On last target peg, enters slow motion and adds a pitch ramp on audio.

### 7.5 Scoring
- Base score per peg.
- Hit streak multiplier that decays over time. Streak resets on floor contact.
- Bucket catch bonus. End of level bonus for remaining balls.
- Medals based on score thresholds. Bronze, silver, gold.

### 7.6 Win and Fail
- Win when all target pegs are cleared. If sub balls exist, they finish scoring then despawn.
- Fail when out of balls. End of level view shows pegs left and a suggested tip if aim assist is off.

## 8. Physics and Feel
### 8.1 Matter.js Configuration
- Fixed time step. 120 Hz physics step with render interpolation.
- World gravity Y equals 1.2 in Phaser units. Units are pixels since Matter is pixel based here.
- Collision categories for ball, pegs, bucket, walls, sensors. Sensors for bucket and floor zones.
- Continuous collision on the ball using increased iterations when velocity exceeds a threshold.
- Disable sleeping for the ball. Allow sleeping for pegs that are static anyway.
- Restitution values:
  - Ball restitution 0.96. Linear damping 0.01. Angular damping 0.
  - Peg restitution 0.92 to 0.98 by type. Static bodies with zero friction.
  - Bucket restitution 0.90. Friction 0.2.
- Constraint iterations. Position iterations 8 or higher. Velocity iterations 6 or higher. Tune for stability.

### 8.2 Time Step Strategy
- Use a custom runner. Accumulate real time. Step physics in fixed slices. Cap catch-up to avoid spiral of death. Interpolate render transforms between physics states.
- Clamp delta spikes to 33 ms in the accumulator. Drop frames rather than stepping more than 2 times per render.
- Pause simulation during loading and end of level fanfare.

### 8.3 Determinism and Replays
- All random choices use a seedable PRNG. Default splitmix or mulberry for speed.
- Record input events and seed only. Replays feed the same inputs at the same physics steps.
- Lock the physics time step and disable variable time integrations. Device independent results.
- Save ghost data as a compact path of ball positions per N frames for spectating.

### 8.4 Juice and Presentation
- Camera. Small screen shake on high energy impacts with a frequency damped envelope. Shake amplitude scaled by impact energy.
- Particles. GPU friendly sprite particles for hits and confetti. Object pool for all emitters.
- Trails. Velocity scaled ghost trail on the ball. Disable on low power devices.
- Post effects. Mild bloom. Short motion blur only during slow motion. Chromatic aberration off by default.
- Audio. Layered SFX per hit. Sidechain music to SFX by 2 to 3 dB on big hits. Quantize stingers to musical bars.

## 9. UI and UX
### 9.1 Screens
- Title. Start, settings, credits.
- Level select. Grid of cards with medals and best scores.
- Gameplay HUD. Ball count, score, multiplier, guide toggle, pause.
- Results. Score breakdown, medals, replay, next, return.
- Settings. Graphics, audio, input, accessibility.

### 9.2 Input and Controls
- Mouse and keyboard. Mouse to aim and left click to shoot. Space to shoot. R to replay level.
- Controller. Right stick to aim. A or bottom face button to shoot. Back to pause.
- Touch. Drag to aim with a visible guide. Release to shoot. Two finger tap to pause.

### 9.3 Accessibility
- Motion reduction toggle. Disables shake and strong trails.
- Color blind safe palettes. Deuteranopia, protanopia, tritanopia presets.
- Text scale options. 90 percent to 140 percent.
- Aim assist levels. Off, short, long.
- Audio dynamic range. Night mode reduces peak to average by 6 dB.

### 9.4 Localization
- English at launch. Externalize strings to JSON. Pseudo localization pass for layout checks.

## 10. Content Scope
- Ten handcrafted levels across two themes.
- Two power ups and one extra governed by level rules.
- One music track with layered intensity. Short win stinger. Ten SFX layers for hits and UI.
- Art style. Clean vector look with soft shading. Limited palette that supports color blind presets.

## 11. Data Format and Schemas
### 11.1 Level JSON Schema
```json
{
  "version": 1,
  "name": "Level 03",
  "seed": 123456789,
  "bounds": { "w": 1280, "h": 720 },
  "gravityY": 1.2,
  "ball": { "spawn": { "x": 640, "y": 80 }, "radius": 10 },
  "bucket": { "y": 680, "speed": 220, "amplitude": 520 },
  "pegs": [
    { "x": 320, "y": 200, "r": 12, "type": "normal" },
    { "x": 480, "y": 260, "r": 12, "type": "target" },
    { "x": 760, "y": 240, "r": 12, "type": "multiplier", "mult": 2 }
  ],
  "specials": [
    { "x": 600, "y": 320, "r": 12, "type": "power", "power": "splinter" }
  ],
  "materials": {
    "ball": { "restitution": 0.96, "linDamp": 0.01 },
    "peg": { "restitution": 0.94 },
    "bucket": { "restitution": 0.9 }
  },
  "goals": { "targetsToClear": 25, "scoreMedals": [10000, 25000, 40000] }
}
```

### 11.2 Telemetry Event Schema
- `level_start`: level_name, seed, aim_assist, device_profile.
- `shot`: time_since_start, aim_angle, guide_length, seed, ball_speed.
- `peg_hit`: peg_type, combo, multiplier, energy.
- `bucket_catch`: streak, bonus.
- `level_finish`: score, stars, balls_left, duration, p95_frame_time, device_profile.
- `settings_changed`: name, value_from, value_to.

## 12. Technical Architecture
### 12.1 Systems
- **Game**. High level state machine and scene management.
- **Physics**. Matter world manager, runner, and body factories.
- **Rendering**. Layers for background, pegs, ball, FX, and UI. Sprite batching.
- **Audio**. Mixer with buses for music, SFX, UI. Sidechain compressor on music.
- **Input**. Unified input with device profiles and sensitivity settings.
- **FX**. Particles, camera controller, and post process toggles.
- **Content**. Level loader, JSON validation, and asset catalog.
- **Replay**. Recorder and player. Deterministic time step source of truth.
- **Telemetry**. Optional and anonymous. Event queue and batch uploader with consent. No account or identifiers beyond a local random device token that never leaves the device without explicit opt in.
- **Settings**. Config manager with overrides per device profile.

### 12.2 Asset Pipeline
- Art in SVG to PNG atlases at build time. TexturePacker or similar.
- Audio in Ogg and AAC. Pre decode short SFX on load.
- Fonts as WOFF2. Fallback stack defined.
- LDtk or Tiled to JSON for rapid level authoring.

### 12.3 Build and Delivery
- Vite for development server and bundling.
- GitHub Actions for CI. Lint, unit tests, size budget, and builds.
- Hosting on static site. Cache control with immutable assets and versioned bundles.
- Optional desktop via Tauri. Optional mobile via Capacitor if needed.

## 13. Performance Targets and Quality Settings
- Desktop target. 60 fps on 1080p with integrated graphics. 120 fps on gaming monitors where VSync is off.
- Mobile target. 60 fps on mid devices. 30 fps fallback on low devices with reduced particles and DPR cap 1.5.
- Quality options. Particle count, trail length, bloom toggle, motion reduction mode.
- Draw calls. Pegs and ball in the same material where possible. UI separate.
- GC control. Object pooling for particles, text popups, and peg glow sprites.

## 14. QA and Test Plan
- Automated perf scene that spawns 1000 peg hits over 10 seconds. Record mean, p95, p99, p99.9 frame times. Gate on p99.9 under 16.7 ms.
- Replay determinism tests with golden inputs per build. Frame by frame diff under 1 px average deviation.
- Unit tests for scoring, multiplier decay, and bucket catch states.
- Input device tests across mouse, controller, and touch.
- Accessibility checklist. Color contrast, motion reduction, font scale, input remap.

## 15. Privacy and Compliance
- No PII collected. Device profile and performance data only.
- Telemetry is opt in and anonymous. No login, no accounts, no user level tracking.
- Cookies only for settings persistence and telemetry consent.

## 16. Risks and Mitigations
- Physics jitter on mobile browsers. Mitigate with fixed step, interpolation, and a DPR cap.
- Matter circle approximations can cause tiny overlaps. Increase quality settings and iterations. Consider Planck.js if accuracy blocks quality.
- Asset size bloat. Enforce texture and audio budgets with CI checks.
- Replay drift across browsers. Lock step and seeded RNG with consistent math. Add browser specific tolerances.
- Long GC pauses in some JS engines. Pool objects and avoid allocation in the hot path.

## 17. Milestones and Deliverables
### M0 - Project Setup - 1 week
- Repo, CI, Vite, Phaser 4 with Matter. Level JSON schema. Seeded RNG. Basic scene.

### M1 - Physics Lab - 1 to 2 weeks
- Ball, pegs, walls, bucket, scoring. Fixed step runner with interpolation. Two levels. Perf scene.
- Exit criteria. Deterministic replays. p99.9 under 16.7 ms on mid laptop in perf scene.

### M2 - Feel Kit - 2 weeks
- Particles, trails, camera shake, slow motion, audio sidechain, HUD.
- Exit criteria. Zero missed inputs at 60 fps. No hitching during 200 hit burst.

### M3 - Content and Power Ups - 2 weeks
- Ten levels. Two power ups. Medal thresholds. Level select. Results screen.
- Exit criteria. Internal playtest NPS 50 or higher. Score attack loop works with **local** scores.

### M4 - Polish and Beta - 2 weeks
- Accessibility, localization pass, quality toggles, optional telemetry hooks, bug fixing.
- Exit criteria. Crash free sessions at or above 99.8 percent. D1 retention in closed beta at or above 25 percent.

## 18. Acceptance Criteria
- Smooth feel verified by perf gates and playtest checklist.
- Deterministic replays across major browsers for the same seed and input script.
- All levels finishable. No soft locks. No out of bounds without recovery.
- Settings persist across sessions. Telemetry opt in respected.
- Web build loads interactive in under 3 seconds on broadband and under 8 seconds on 4G.

## 19. Out of Scope for V1
- Any multiplayer or online competitive features.
- Social sharing beyond copy link.
- Cloud save. Local save only.
- In app purchases. No ads.

## 20. Appendices
### 20.1 Example TypeScript Types
```ts
export type PegType = "normal" | "target" | "multiplier" | "power";

export interface Peg {
  x: number;
  y: number;
  r: number;
  type: PegType;
  mult?: number;
  power?: "super_shot" | "splinter" | "time_stretch";
}

export interface LevelDef {
  version: number;
  name: string;
  seed: number;
  bounds: { w: number; h: number };
  gravityY: number;
  ball: { spawn: { x: number; y: number }; radius: number };
  bucket: { y: number; speed: number; amplitude: number };
  pegs: Peg[];
  specials: Peg[];
  materials: {
    ball: { restitution: number; linDamp: number };
    peg: { restitution: number };
    bucket: { restitution: number };
  };
  goals: { targetsToClear: number; scoreMedals: [number, number, number] };
}
```

### 20.2 Feel Checklist
- Aiming arc is readable on all backgrounds.
- First collision prediction matches the actual hit within 1 px 95 percent of the time.
- Camera shake never obscures the ball. Max amplitude under 6 px at 1080p.
- Audio mix leaves space for impacts. Stingers do not clip.
- Particles do not overwhelm the screen. Cap counts on low devices.

### 20.3 Build Size Budget
- JS bundle under 1.2 MB gzip.
- Art under 3 MB combined.
- Audio under 1.5 MB combined at launch quality.

### 20.4 Open Questions
- Do we want a daily seed for score attack that rotates at midnight local time.
- Should we include an on board tutorial level or rely on interactive hints.
- Do we need cloud save for desktop packaging in V1.

---

**Owner**: JB  
**Engineering Lead**: TBD  
**Art and Audio**: TBD  
**QA**: TBD
