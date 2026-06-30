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
On mobile (≤520px) Accuracy and Combo are hidden to fit the compact single-row HUD.

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
- `startGame()` — resets all counters, hides the main overlay, starts loops, calls `focusMobileInput()`.
- `pauseGame()` — cancels RAF + interval, shows `#pause-overlay`, freezes tiles in place, calls `blurMobileInput()`.
- `resumeGame()` — hides pause overlay, resets `lastMove`, restarts RAF + interval, calls `focusMobileInput()`.
- `resetGame()` — full teardown (removes all tiles, cancels loops), restores the start-screen overlay, calls `blurMobileInput()`.
- `gameOver()` — stops loops, saves high score, shows game-over stats in the main overlay, calls `blurMobileInput()`.

**`syncMenuButtons()` must be called after every state transition** — it updates the disabled/enabled state and label of the three HUD menu buttons (`#btn-start`, `#btn-stop`, `#btn-reset`). Missing a call will leave buttons in a stale state.

### Menu controls (HUD)
Three compact buttons live in `#menu-controls` inside the HUD bar, between the level badge and the hearts:
- **▶ Start / Resume** (`#btn-start`) — calls `startGame()` when idle, `resumeGame()` when paused. Label changes dynamically via `.menu-label` span (targeted by `syncMenuButtons()`).
- **⏸ Pause** (`#btn-stop`) — calls `pauseGame()`. Disabled unless actively playing.
- **↺ Reset** (`#btn-reset`) — calls `resetGame()`. Disabled when idle. Styled with `.is-danger` for pink hover.

Button text spans use class `.menu-label`; keyboard hint spans use `.menu-key`. Both are hidden on mobile via media query (buttons show icon only).

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
- HUD is `position:fixed`, `z-index:10`; pause overlay is `z-index:15`; main overlay (start/game-over) is `z-index:20`; level-up splash is `z-index:25`; tap hint is `z-index:6`.
- `@media (max-width: 520px)` — compact HUD (44px), icon-only buttons, hidden Acc/Combo stats, smaller tiles and hearts.

### Mobile support
The game is fully playable on smartphones. Key implementation details:

**Virtual keyboard** — a hidden `<input id="mobile-input">` (1×1px at `top:0; left:0; opacity:0`) is focused programmatically on game start/resume. This is the only reliable cross-browser way to trigger the software keyboard. `font-size: 16px` prevents iOS Safari from auto-zooming on focus. The input must be within the visible viewport — iOS silently refuses to show the keyboard for `position:fixed` inputs positioned off-screen.

**Input routing** — `isMobile` (`'ontouchstart' in window || navigator.maxTouchPoints > 0`) gates the two input paths:
- Desktop: `keydown` on `document` (unchanged, fires regardless of focus)
- Mobile: `input` event on `#mobile-input`; `e.data` holds the typed character; value is cleared immediately after each keystroke so `e.data` is always a single char

Both paths call `processChar(key)` — a shared function extracted from the original keydown handler.

**Viewport** — `resizeCanvases()` and `spawnWord()` both read `window.visualViewport.height` (fallback: `window.innerHeight`) so words never spawn behind the software keyboard. A `visualViewport.resize` listener re-runs canvas resize when the keyboard slides in/out.

**Tap-to-type hint** — `<div id="tap-hint">` is shown when `#mobile-input` loses focus during active play (keyboard dismissed). Tapping it, or tapping `#game-area`, re-focuses the hidden input. The hint hides again on `focus`.

**Mobile input helpers** (called by lifecycle functions):
- `focusMobileInput()` — no-op on desktop; on mobile calls `mobileInput.focus()` **synchronously** (no setTimeout) so the call stays within the user-gesture event handler iOS requires to show the keyboard.
- `blurMobileInput()` — blurs input and hides tap hint; called on pause/game-over/reset to dismiss the keyboard.

### Language support
The game supports English and Arabic. Language is chosen on the start/game-over overlay before every game; it persists across resets but not page reloads (defaults to `'en'`).

**State:**
- `state.language` — `'en'` | `'ar'`; read by `pickWord()` and `spawnWord()`.
- `overlayMode` — `'start'` | `'gameover'`; set by `resetGame()` and `gameOver()` so `applyOverlayStrings()` knows which text set to apply.

**Word lists:**
- `WORDS` / `ALL_WORDS_BY_TIER` — English, 5 tiers.
- `WORDS_AR` / `ALL_WORDS_AR_BY_TIER` — Arabic, same 5-tier structure (grouped by character length).
- `pickWord()` selects the correct tier list based on `state.language`.
- `spawnWord()` adds class `arabic` to the tile element when Arabic is active.

**UI strings:**
- `STRINGS.en` / `STRINGS.ar` — all overlay-visible text: `title`, `sub`, `instructions` (HTML), `startBtn`, `playAgain`, `best(n)`, `gameOver`, `statScore`, `statWpm`, `statAcc`, `statWords`.
- `applyOverlayStrings()` — applies `STRINGS[state.language]` to the overlay. In `'start'` mode updates title/subtitle/instructions/start-button. In `'gameover'` mode updates game-over title/stat-labels/play-again-button. Always updates the high-score line. Also toggles `#overlay.rtl` for RTL text direction.
- `updateLangButton()` — syncs `.lang-btn` `.active` class, sets `mobileInput.lang` and `mobileInput.dir`, then calls `applyOverlayStrings()`.

**Language selector UI:**
- `#lang-selector` in `#overlay` — two `.lang-btn` buttons (`data-lang="en"` / `data-lang="ar"`). Active button glows cyan. Visible on both the start screen and game-over screen.

**Arabic tile rendering:**
- `.word-tile.arabic` — `direction: rtl; letter-spacing: 0; font-family: system-ui, …`. The browser's shaping engine correctly renders connected Arabic letterforms across individual `<span class="char">` elements when `letter-spacing` is 0.
- `mobileInput.lang = 'ar'` and `mobileInput.dir = 'rtl'` hint the OS to suggest the Arabic keyboard layout on mobile.

---

## Handoff Notes

### Current Project State and Architecture

Everything below is fully working as of 2026-06-30:

- 10-level progression with escalating speed and word difficulty; Level 10 is Infinite Mode (no further scaling)
- Real-time WPM (rolling 10-second window), Accuracy, Score, and Combo multiplier in the HUD
- Combo system: ×1 → ×2 → ×4 → ×8; resets on any miss or escaped word
- High score persisted to `localStorage` key `ws_hs`
- Pause / Resume / Reset via HUD buttons and keyboard shortcuts (Esc / Enter / R)
- Particle explosion on word completion; starfield runs independently at all times
- Full mobile support: hidden `#mobile-input` triggers software keyboard; `visualViewport` keeps words above keyboard
- English and Arabic word sets (5 tiers each); language selector on start/game-over overlay; Arabic tiles render RTL with correct letterform shaping
- Deployed on Vercel (auto-deploy from `master`) and served locally via `python -m http.server 3737`

**File structure:**

| File | Responsibility |
|------|---------------|
| `typing-game.html` | Entire game — CSS, HTML, JS, word lists, all inline |
| `vercel.json` | Rewrites `/` → `/typing-game.html` for Vercel |
| `CLAUDE.md` | Architecture guide for AI sessions |
| `.claude/launch.json` | `preview_start` config (`prs-game` profile, port 3737) |

**Non-obvious architectural patterns:**
- Two always-running RAF loops (`drawStars` / `fxLoop`) that ignore `state.running` and `state.paused` — they keep the background alive even on the start screen and pause overlay.
- `moveLoop` is the only game-gated RAF; it bails at the top if `!state.running || state.paused`.
- `spawnTimer` is a `setInterval`, not RAF — cleared on pause/stop, restarted via `restartSpawnTimer()`.
- Z-index stack: canvas (0) → game-area tiles (5) → HUD (10) → pause-overlay (15) → main overlay start/gameover (20) → level-splash (25) → tap-hint (6).

### Key Decisions Made and Why

**Single self-contained HTML file**
- Why: zero-setup deploy, no build step, works anywhere Python can serve files. The user explicitly chose this over a bundled app.
- Rules out: importing external libraries, splitting JS/CSS into separate files, using `import` statements.

**DOM word tiles over canvas text**
- Why: HTML text rendering handles Arabic RTL ligatures and right-to-left shaping natively. Canvas `fillText` would require a custom Arabic shaper.
- Rules out: rendering tiles on `#fx-canvas` or `#bg-canvas`. Tiles are always `<div class="word-tile">` DOM elements inside `#game-area`.

**Two-boolean state machine (`running` + `paused`)**
- Why: simpler than an enum; two booleans cover every valid state with no invalid combinations at runtime.
- Rules out: a third boolean to distinguish "idle" from "game-over" — both are `running:false, paused:false`. The overlay content distinguishes them via `overlayMode`.

**Always-on starfield and particle RAF loops**
- Why: visual continuity — the background keeps moving while the game is paused or on the start screen.
- Rules out: gating `drawStars()` or `fxLoop()` on `state.running`. Never cancel these RAFs in lifecycle functions; only `moveLoop` gets cancelled.

**Hidden 1×1px input for mobile keyboard (`#mobile-input`)**
- Why: the only reliable cross-browser way to programmatically trigger the software keyboard on iOS and Android.
- Rules out: `focus()` calls inside `setTimeout` — iOS requires the focus call to happen synchronously inside a user-gesture handler. Any async path silently fails to open the keyboard.

**`letter-spacing: 0` on `.word-tile.arabic`**
- Why: the browser's text shaping engine connects Arabic letters into correct ligatures only when letter-spacing is exactly zero. Any non-zero value breaks the per-`<span>` character approach and renders disconnected glyphs.
- Rules out: using `letter-spacing` for Arabic tile visual spacing. Use padding or gap instead.

**`visualViewport.height` instead of `window.innerHeight`**
- Why: on mobile, `window.innerHeight` does not shrink when the software keyboard opens. `visualViewport.height` reflects the actual visible area, so words never spawn behind the keyboard.
- Rules out: using `window.innerHeight` anywhere that feeds canvas sizing or spawn Y-coordinate logic.

### Coding Patterns and Conventions Established

**State transitions:** every lifecycle function (`startGame`, `pauseGame`, `resumeGame`, `resetGame`, `gameOver`) must call `syncMenuButtons()` at the end to keep HUD button labels and disabled states in sync. Missing this call leaves stale button states.

**i18n additions:** to add a new UI string, add it to both `STRINGS.en` and `STRINGS.ar`, then apply it inside `applyOverlayStrings()`. Never hardcode English text in DOM-manipulating functions.

**Audio:** always use the `playTone(freq, type, dur, vol, delay)` helper. Never create oscillator/gain nodes directly. Gate everything behind `if (!state.audioCtx) return` — `initAudio()` is called lazily on first keystroke and may not exist yet.

**CSS theming:** all colors are `--neon-*` custom properties on `:root`. Change them there to retheme. Never hardcode hex colors in JS `fillStyle` calls — pull from `getComputedStyle` if you need a canvas color.

**Tile registry:** every live tile has an entry in `state.tiles[id]` as `{word, typed, x, y, el, speed}`. When removing a tile (word complete or escaped), always delete `state.tiles[id]` and `el.remove()` together — orphaned DOM elements or orphaned state entries both cause bugs.

**Active tile selection:** after any tile is removed, call `selectLeftmostTile()` to re-establish `state.activeId`. Never set `state.activeId` directly outside that function.

**Mobile focus:** `focusMobileInput()` must be called synchronously inside the user-gesture event handler (button click, keydown). Never wrap it in `setTimeout`.

### Specific Next Steps with Implementation Details

**1. Mute / sound toggle (low effort)**

Add `muted: false` to `state`. Gate `playTone()`:
```js
function playTone(freq, type, dur, vol = 0.3, delay = 0) {
  if (state.muted || !state.audioCtx) return;
  // ... existing oscillator code ...
}
```
Add a mute button to `#menu-controls` in the HUD (between `#btn-reset` and `#hearts`):
```html
<button id="btn-mute" class="menu-btn" title="Mute">🔇</button>
```
Wire it: `btnMute.onclick = () => { state.muted = !state.muted; syncMenuButtons(); }`.
In `syncMenuButtons()`, update `btnMute.textContent` between `🔇` / `🔊`.

**2. Difficulty / lives selector on start overlay (low effort)**

Add a segmented control above the Start button on `#overlay` with Easy (7 HP) / Normal (5 HP) / Hard (3 HP). Store choice in `state.startHp` (default `5`). In `startGame()`, replace `state.hp = 5` with `state.hp = state.startHp`. Add difficulty labels to `STRINGS.en/ar`.

**3. Per-level WPM leaderboard (medium effort)**

`localStorage` key `ws_wpm_L{n}` per level (1–10). After `gameOver()`, if `state.level > 1`, record best WPM per level reached. Display on game-over overlay as a small table. `STRINGS` needs new keys for the table header and "Best WPM" label.

**4. French / Spanish language support (medium effort — i18n infrastructure ready)**

The entire i18n system is already parameterised. Adding a third language requires:
1. `WORDS_FR` / `ALL_WORDS_FR_BY_TIER` — same 5-tier structure as English
2. `STRINGS.fr` — translate all existing keys
3. A third `.lang-btn` in `#lang-selector`
4. `updateLangButton()` already handles arbitrary language codes — no changes needed there

**5. Timed challenge mode (needs design first)**

A "60 seconds" mode that ends after a fixed duration rather than on HP loss. Needs a countdown timer in the HUD, a new `mode` field in `state`, and a `gameOver()` branch for time-up. The start overlay needs a mode selector (infinite lives vs. timed). Design the UX before implementing.

### Files That Need Attention Next

| File | Why it needs attention |
|------|----------------------|
| `typing-game.html` | Entire codebase — CSS, HTML, and JS all inline; read this before any feature work |
| `CLAUDE.md` | Update the Architecture sections if new patterns are introduced (state fields, lifecycle functions, z-index layers) |
| `memory/project_mobile_support.md` | Documents iOS-specific keyboard constraints — read before touching `focusMobileInput()` or viewport logic |
| `memory/project_arabic_language.md` | Documents the `letter-spacing: 0` shaping requirement and Arabic tier structure — read before modifying tile rendering or adding languages |
