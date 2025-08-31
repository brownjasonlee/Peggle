# Settings Implementation Review

This review walks through each settings group, verifies current behavior, notes impacts on other systems, and recommends improvements to ensure a smooth user experience.

## Summary
- Most controls update the live game immediately and safely.
- Level generation parameters (columns, rows, spacing, peg radius, start Y, target ratio) previously required a manual restart to take effect. This is now automated with a debounced reset.
- Bucket speed did not previously apply live because only `opts.bucket.speed` was updated. Fixed by also updating `bucket.speed`.

## What Changed
- Automated reset for generation-affecting settings: Changing any of `Columns`, `Rows`, `Peg spacing`, `Peg radius`, `Start Y`, or `Target ratio` schedules a level rebuild after 300 ms of inactivity. The seed is preserved, so runs remain deterministic.
- Live bucket speed: The slider now updates both `opts.bucket.speed` and the runtime `bucket.speed` used by movement.

## Feature-by-Feature Check

1) Physics
- Gravity, Restitution (ball/peg/wall), Air drag: Update immediately and affect simulation on the next step. No cross-impact issues observed.
- Time step (Hz): Updates `DT` immediately; render loop continues smoothly. Guide prediction references the same `DT`, so it remains coherent.

2) Ball and Launch
- Launch speed: Updates immediately and only affects the next shot.
- Ball radius: Updates `BALL_R` immediately. Existing pegs and layout remain; this is intentional and works without instability.
- Balls per game: Updates `ballsLeft` and HUD immediately. No adverse side effects.

3) Bucket
- Speed: Now updates live (bug fixed). Movement uses `bucket.speed` correctly.
- Size (W/H): Updates the runtime `bucket` dimensions immediately. Wall-margin clamping in movement handles larger widths without error.

4) Guide and Aiming
- Guide on/off, bounces, length, density, fade: All update live and affect the next aim visualization. Bounce selector disables when guide is off.

5) Level Generation
- Columns, Rows, Spacing, Peg Radius, Start Y, Target Ratio: Now trigger an automatic, debounced `resetLevel(seed)` to rebuild pegs. This ensures changes are visible without manual restart and prevents rapid rebuilds while the user types/adjusts.
- Seed: “Use” button and Enter key both rebuild with the specified hex value.

6) Scoring and Difficulty
- Normal/Target peg scores, Streak step, Catch bonus (ball/points/both): Update immediately; scoring logic reads from `opts.scoring` on hit/catch. No unexpected interactions found.

7) Particles and Visuals
- Density, speed range, life range, gravity: Apply to newly emitted particles only, as expected.
- Trail length: Updates `MAX_TRAIL` immediately.
- DPR cap: Triggers a resize; drawing remains crisp and stable.

8) Accessibility and UX
- Color-blind palette: Toggles live; draw routines read from `opts.colorBlind` each frame.
- Options hotkey (O) and Restart button: Work as expected.
- Preset tooltips: Each preset now includes a short description shown on hover; the select’s title mirrors the current choice.

## Recommendations

- Debounce consistency: The 300 ms debounce on level-generation inputs feels good; keep it consistent for any future generation-affecting controls.
- Visual feedback on rebuild: Consider a subtle “Regenerating…” flash or a small toast to indicate the level rebuilt after changes.
- Input typing vs. change: For number fields (`Cols`, `Rows`, etc.), the current `input` event is ideal for debounced rebuilds. If we ever add validation or min/max clamps, surface inline warnings near the field.
- Preset awareness: When the user changes any option after picking a preset, consider updating the `Preset` dropdown to `Custom` to indicate divergence.
- Persist options: Optionally save settings to `localStorage` and restore on load for a smoother return experience.

## Conclusion

All settings are wired and behaving correctly. Generation-related changes now automatically rebuild the level (with preserved seed), and bucket speed updates live. No cross-system regressions observed during this audit. The above UX improvements would make configuration clearer and more ergonomic.

