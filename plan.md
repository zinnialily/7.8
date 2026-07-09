# K2 Climb — Rebuild Plan

Rebuild `k2-climb.html` (currently a 3-lane tap-to-dodge game) into a real climbing
platformer inspired by Google Doodle Champion Island's "Climbing" minigame, while
keeping the existing K2/Yahya theme, real mountain facts, camp checkpoints, and
scrapbook art style.

## Keep as-is
- Scrapbook/kraft-paper visual style, tape decorations, Patrick Hand + Caveat fonts.
- `CAMPS` data (Base Camp → Camp 1-4 → Bottleneck → Summit) with real altitudes and facts.
- Camp toast popup (`showCampToast`) shown when player reaches a new camp.
- Win/lose overlay (`showOverlay`/`endGame`) and rest mechanic at camps.
- Overall narrative arc: climb from 5150m Base Camp to the 8611m summit.

## Replace: core mechanic
Swap the 3-lane dodge for real 2D platforming:
- Player has x/y position, gravity, and a jump/grab action (spacebar or tap-to-jump
  on mobile) instead of just left/right lane switches.
- Camera scrolls vertically as the player ascends, revealing more mountain above
  (extend the existing `drawMountainBg` parallax instead of replacing it).
- Falling off the bottom of the visible screen (missing every hold) counts as a
  fall; player respawns at the last camp ledge reached, mirroring the
  lantern/checkpoint behavior in Champion Island.

## New: hold system
Three hold types, colored distinctly, spawned procedurally in a column pattern as
the player climbs:

| Type | Color | Behavior |
|------|-------|----------|
| Stationary | pink/red | Fixed position, safe to hang on indefinitely |
| Weak | green | Shakes after a short hold time, then crumbles and respawns elsewhere a moment later |
| Moving | blue | Slides on a linear or circular path; player must time the jump |

Camp checkpoints are full-width ledges (like the existing camp crossing logic) —
the only spots where the player can rest, matching current `enterRest`/`exitRest`.

## New: obstacles
- Falling ice chunks/rocks drop from above at intervals, telegraphed by a brief
  shadow/particle warning (like the snow-particle tell in the reference game)
  so it's dodgeable, not cheap.
- Getting hit while mid-air (not on a hold) knocks the player off their grip.
- Keep obstacle difficulty scaling with altitude, similar to current
  `spawnObstacle` cadence logic.

## Controls
- Desktop: arrow keys / WASD to move, spacebar to jump.
- Mobile: on-screen joystick or left/right touch zones for movement, single tap
  button for jump — reuse the existing touch-first layout since this is a gift
  for a phone.

## Data model changes
- Replace lane-index obstacle objects with `{x, y, type}` hold objects and
  `{x, y, vx, vy}` falling obstacle objects.
- Track player `{x, y, vx, vy, grabbedHold}` instead of `{lane}`.
- Keep `CAMPS` altitude thresholds, but convert to world-y positions for ledge
  placement instead of pure lane-crossing checks.

## Rendering changes
- Draw holds by type/color with subtle shake/slide animation.
- Draw player with a grab/reach pose when attached to a hold, fall pose in air.
- Extend parallax mountain background to scroll smoothly with camera y.

## Build phases
1. Physics core: gravity, jump, hold-grabbing collision, camera scroll — flat
   test level, no theme.
2. Hold system: implement all 3 hold types with correct behaviors, procedural
   column generation up to summit altitude.
3. Camp ledges: port `CAMPS` data into full-width rest ledges with existing toast
   + rest UI.
4. Obstacles: falling rocks with warning tell, collision knock-off, fall/respawn
   at last camp.
5. Win/lose: summit reached → win overlay; (need to decide fail condition, see
   open questions).
6. Art pass: reskin holds/obstacles/player to match scrapbook style, polish
   parallax background.
7. Mobile controls + playtesting on phone (this is the target device).

## Open questions before implementation
- **Fail condition**: Champion Island uses a 90-second timer. Do we want a timer
  (tense, replayable) or keep it untimed/endless-fall-only (more relaxed, gift-like)?
- **Difficulty curve**: should hold spacing/obstacle frequency ramp up per camp
  (matching the real K2 danger progression, e.g. Bottleneck is hardest), or stay
  flat difficulty throughout?
- **Session length**: rough real climb time target (60-90s like the reference, or
  longer since this is a personal keepsake, not a speedrun target)?
- **Retry behavior**: on falling to the very bottom (past Base Camp), full restart,
  or always resume from nearest camp with no full game-over?
