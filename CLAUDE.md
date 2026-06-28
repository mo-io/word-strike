# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Word Strike** — a neon cyberpunk typing-speed trainer built as a single self-contained HTML file (`typing-game.html`). No build step, no dependencies, no external assets. Everything is inline HTML/CSS/JS. The game teaches faster typing through progressive difficulty, real-time feedback, and a combo/score system.

## Deployment

**Live (Vercel):** deployed from the `master` branch of `github.com/mo-io/word-strike` (private repo). Vercel auto-redeploys on every push to `master`.

`vercel.json` rewrites `/` → `/typing-game.html` so the root URL serves the game (Vercel's static server defaults to `index.html`, which doesn't exist here).

**GitHub:** `git@github.com:mo-io/word-strike.git` — private repo, SSH auth via `C:\Program Files\GitHub CLI\gh.exe` (account: `mo-io`).

## Running locally

The launch config serves the project root with Python's built-in HTTP server:

```
python -m http.server 3737
```

Then open `http://localhost:3737/typing-game.html`. The `.claude/launch.json` is already configured for the `preview_start` tool under the name `typing-game`. Node/npm is not available in this environment — Python is the only runtime.

## Game overview

Words drift left across the screen as glowing neon tiles. The player types the currently highlighted word to destroy it with a particle explosion. Letting a word escape off the left edge costs a life (♥) and resets the combo. The game ends at 0 HP. 10 levels of escalating speed and word difficulty; level 10 is "Infinite Mode" with no further scaling.

**HUD stats:** WPM (rolling 10-second window) · Accuracy (%) · Score · Combo multiplier · Level badge · Hearts (HP)

**Scoring:**
- Base: `word.length × 10 × combo` points per word
- Combo escalates: ×1 → ×2 (2 words) → ×4 (4 words) → ×8 (8 words); resets on any miss
- High score persisted to `localStorage` key `ws_hs`

**Keyboard shortcuts (always active):**
| Key | Action |
|-----|--------|
| `Esc` | Pause (during play) / Resume (when paused) |
| `Enter` | Start (from start/game-over screen) / Resume (when paused) |
| `R` | Reset to start screen (only when paused or on game-over screen — not during active play) |

## Architecture

`typing-game.html` is self-contained with three logical layers:

### Rendering (two `<canvas>` elements)
- `#bg-canvas` — animated starfield; `drawStars()` runs its own `requestAnimationFrame` loop independently of game state, always moving.
- `#fx-canvas` — particle explosions on word destruction; `fxLoop()` always running. Particles are plain objects `{x, y, vx, vy, life, decay, r, color}` updated and drawn each frame.

### DOM word tiles
- `<div class="word-tile">` elements, `position:absolute` over the canvases inside `#game-area`.
- Each tile tracked in `state.tiles[id]` as `{word, typed, x, y, el, speed}`.
- Characters rendered as individual `<span class="char">` elements; typed chars get class `typed` (neon green glow), wrong key triggers `wrong-flash` class (pink border flash) via `flashError()`.
- One tile is always "active" (`state.activeId`) — glowing cyan border, receives all keystrokes. Auto-selects the left-most tile after a word is destroyed or escapes.

### Game loop
- `moveLoop(ts)` — `requestAnimationFrame` loop that translates tiles left by `speed × (dt / 16.67)` px per frame. Bails immediately if `!state.running || state.paused`. Restarted explicitly on `resumeGame()` and `startGame()`.
- `spawnTimer` — `setInterval` that calls `spawnWord()` at the interval defined by the current level config. Cleared on pause/stop, restarted via `restartSpawnTimer()`. Spawn is also called immediately on game start so the first word appears instantly.

### State machine
Two booleans govern everything: `state.running` and `state.paused`.

| State | `running` | `paused` |
|-------|-----------|----------|
| Idle / game over | `false` | `false` |
| Playing | `true` | `false` |
| Paused | `true` | `true` |

Lifecycle functions:
- `startGame()` — resets all counters, hides the main overlay, starts loops.
- `pauseGame()` — cancels RAF + interval, shows `#pause-overlay`, freezes tiles in place.
- `resumeGame()` — hides pause overlay, resets `lastMove`, restarts RAF + interval.
- `resetGame()` — full teardown (removes all tiles, cancels loops), restores the start-screen overlay.
- `gameOver()` — stops loops, saves high score, shows game-over stats in the main overlay.

**`syncMenuButtons()` must be called after every state transition** — it updates the disabled/enabled state and label of the three HUD menu buttons (`#btn-start`, `#btn-stop`, `#btn-reset`). Missing a call will leave buttons in a stale state.

### Menu controls (HUD)
Three compact buttons live in `#menu-controls` inside the HUD bar, between the level badge and the hearts:
- **▶ Start / Resume** (`#btn-start`) — calls `startGame()` when idle, `resumeGame()` when paused. Label changes dynamically.
- **⏸ Pause** (`#btn-stop`) — calls `pauseGame()`. Disabled unless actively playing.
- **↺ Reset** (`#btn-reset`) — calls `resetGame()`. Disabled when idle. Styled with `.is-danger` for pink hover.

The pause overlay (`#pause-overlay`) is semi-transparent so the player can see the frozen word positions. It contains its own Resume and Reset buttons wired to the same functions.

### Level progression
- `LEVEL_CFG[1..10]` — each entry: `{maxWords, spawnMs, speedPx, tiers[1..5]}`.
- `LEVEL_SCORE[1..10]` — cumulative score thresholds; checked in `checkLevelUp()` after every word completion.
- Word pool has 5 tiers (2–3 chars → 11+ chars). `tiers` array in each level config is a percentage weight array; `pickWord()` rolls against it to select a tier, then picks a random word from that tier avoiding on-screen duplicates.
- Level-up triggers a brief `#level-splash` card animation and calls `restartSpawnTimer()` with the new interval.

### Audio
- Procedural via Web Audio API (`state.audioCtx`). Initialized lazily on first keystroke via `initAudio()` to satisfy browser autoplay policy.
- All sounds synthesized with `playTone(freq, type, dur, vol, delay)` — oscillator + gain node, no audio files.
- Events: key tick (triangle wave), word complete (ascending arpeggio), miss (sawtooth thud), level up (chord), wrong key (square blip).

### Visual / CSS
- CSS custom properties on `:root` for all neon colors (`--neon-cyan`, `--neon-green`, `--neon-pink`, `--neon-yellow`, `--neon-purple`) — change here to retheme.
- `body.shake` triggers a keyframe animation for screen shake on HP loss.
- `.menu-btn.is-active` → cyan glow. `.menu-btn.is-danger` → pink on hover. Both toggled by `syncMenuButtons()`.
- HUD is `position:fixed`, `z-index:10`; pause overlay is `z-index:15`; main overlay (start/game-over) is `z-index:20`; level-up splash is `z-index:25`.
