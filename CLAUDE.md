# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**DOOMPG** — a single-file HTML5 wave shooter inspired by Doom. Open `doom-shooter.html` directly in a browser (no build step, no server needed).

## Running the game

```bash
open doom-shooter.html        # macOS — opens in default browser
```

For local image loading to work reliably, Chrome or Firefox are preferred over Safari. The game degrades gracefully if any `.jpg` asset is missing — monsters render as colored placeholders.

## Architecture

Everything lives in `doom-shooter.html`. The script is structured in clearly commented sections, top to bottom:

| Section | What it does |
|---|---|
| **CONFIG** | All tunable constants (canvas size, ammo, damage, per-lane scale math) |
| **ASSETS** | `imgs` object pointing at hidden `<img>` elements; `imgOk` tracks load status |
| **AUDIO** | Web Audio API, lazy-initialised on first click. All sounds are synthesized — no audio files |
| **Gun class** | Per-gun state: recoil, flash, ammo, cooldown. `fire()` returns bool |
| **Monster class** | `progress` (0=FAR → 1=reaches player) drives position/scale. State machine: `alive → dying → dead`. Shield logic lives here |
| **Particles** | Simple array of `Particle` objects; `fireParticles` is a separate plain-object array for Level 3 floor fire |
| **Hit detection** | `hitTest(mx, my)` sorts active monsters close-first and checks rectangular bounds |
| **Corridor renderer** | `drawCorridor()` draws ceiling/floor/walls as filled polygons with linear gradients converging at vanishing point `(VPX, VPY)` |
| **HUD / Screens** | Separate functions per screen state |
| **Wave manager** | `waveQueue` feeds monsters into `activeMonsters` (max 5 alive at once). Detects wave/level/victory clear |
| **Game state** | Single `state` string: `TITLE → PLAYING → WAVE_CLEAR → LEVEL_CLEAR → GAME_OVER / VICTORY` |
| **Input** | `canvas` click fires left gun immediately, right gun via `setTimeout(..., 150)` |
| **Game loop** | `requestAnimationFrame` loop; delta-time capped at 50 ms |

## Key data

**Monster depth math** — `progress` lerps between constants:
- Scale: `SCALE_FAR (0.22)` → `SCALE_CLOSE (0.92)` × `sizeScale`
- Y feet position: `YFEET_FAR (316)` → `YFEET_CLOSE (490)`
- X spread per slot: `XSPREAD_FAR (38)` → `XSPREAD_CLOSE (158)`
- Slots: −2, −1, 0, +1, +2 (max 5 simultaneous monsters)

**Adding a new monster type** — add an entry to `MONSTER_TYPES` with `{ hp, color, imgKey, sizeScale, hasShield? }`, drop the matching `.jpg` into the project folder, and reference the key in any wave definition inside `LEVELS`.

**Adding a level** — append an object to the `LEVELS` array with `{ name, subtitle, bgR, bgG, bgB, advanceMs, waves[] }`. Each wave is an array of monster-type strings.

## Assets

| File | Role |
|---|---|
| `lesha.jpg` | Hero 1 — shown in HUD portraits and title/victory screens |
| `yura.jpg` | Hero 2 — same |
| `marina.jpg` | Monster type `marina` sprite (circle-clipped) |
| `pasha.jpg` | Monster type `pasha` sprite (circle-clipped) |

## Git workflow

After completing any meaningful unit of work — a bug fix, a new feature, a balance change — commit immediately so progress is never lost. Push to `origin main` after each commit.

```bash
git add doom-shooter.html
git commit -m "short description of what changed and why"
git push
```

Commit messages should say **what changed and why**, not just "update file". Examples:
- `fix: shield regen condition always evaluated false`
- `feat: add Level 4 with new enemy type`
- `balance: increase marina HP to 80 in Level 2`

## Debug / cheat keys

| Key | Effect |
|---|---|
| `D` | Toggle debug overlay (FPS, state, monster count, bounding boxes) |
| `K` | Kill all active monsters and clear the wave queue instantly |
| `S` | Cycle through sound cues one by one (useful when adding new audio) |
