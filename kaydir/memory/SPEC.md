# Reverse Arcade — SPEC

## What the app is
A single-page neon arcade reflex game (based on "Right, Left, Up, Down, Reverse").
Player swipes (touch) or drags (mouse) in the direction the center arrow points.
If the arrow is RED, the swipe must be the OPPOSITE direction. Wrong move or a
beating clock ends the run.

## Stack note
The game is intentionally 100% client-side (user choice): score/best live in
localStorage, no server features. The FastAPI backend is untouched — only the
template's `/api/status` health endpoints exist.

## Core rules
- Center arrow: one of 4 directions (up/down/left/right), cyan (#38BDF8) = normal,
  red (#FF3366) = reverse (swipe opposite).
- 3-arrow queue: main arrow + two small previews above it (next two moves).
  On a correct answer, queue pops front: preview 1 springs down into the main
  slot, preview 2 → preview 1, new random arrow spawns as tail. Previews carry
  the same color rules.
- No time limit: the player can take as long as they want on each arrow (timer removed at user request).
- Score +1 per correct move; streak badge flashes at every ×10 milestone (visual
  only, no scoring effect).
- Red arrows only start appearing after 5 points (25% chance).
- Game over: wrong direction only → shake + red flash, modal with final
  score, best, new-record badge, one-tap PLAY AGAIN.

## Persistence (localStorage keys)
- `ra_best_score` — best score (number)
- `ra_sound_enabled` — "1"/"0" sound toggle (default on)

## Sound
Web Audio API synthesized SFX (frontend/src/lib/audio.ts) — no audio assets,
no network. Toggle in header.

## Controls
- Mobile: touch swipe anywhere on the stage (pointer events, 30px threshold,
  `touch-action: none`).
- Desktop: press-and-drag with mouse (user explicitly chose mouse-swipe-only;
  no keyboard controls).

## Key files
- `frontend/src/lib/game.ts` — pure rules (directions, expected swipe, difficulty curve, storage)
- `frontend/src/lib/audio.ts` — Web Audio SFX engine
- `frontend/src/hooks/useGameEngine.ts` — state machine (idle/playing/over), queue
- `frontend/src/components/game/*` — NeonArrow glyph, GameStage (arena, queue animation, particles), StartOverlay, GameOverOverlay, HowToPlayDialog
- `frontend/src/pages/Game.tsx` — page shell (header, score HUD, stage, overlays)

## Auth / credentials
None — no login, no accounts. See memory/test_credentials.md.

## Testing hooks
Current puzzle state lives on the arena plate `data-testid="main-arrow-box"`:
`data-dir="up|down|left|right"` and `data-reversed="true|false"` — expected swipe =
dir unless reversed=true. Read it from the plate, NOT from `main-arrow-svg`: exiting
arrows linger ~200ms in the DOM during queue transitions (AnimatePresence) and can
carry stale testids.
All required testids from design_guidelines.json
are implemented.
