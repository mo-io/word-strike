# Word Strike — Development Memory

Running log of what has been built, decisions made, and what comes next. Read this alongside `CLAUDE.md` (which covers architecture and how to work with the code).

---

## Current State (2026-06-30)

The game is fully playable on **desktop and mobile** in both **English and Arabic**. It is live on Vercel and auto-deploys from `master`.

### What works
- 10-level progression with escalating speed, word length, and spawn rate
- 5-tier word pool (English + Arabic); weighted random selection per level
- Combo multiplier (×1 / ×2 / ×4 / ×8) with score scaling
- WPM (rolling 10-second window), Accuracy %, Score, HP, Level HUD
- Particle explosions, starfield background, procedural Web Audio
- Screen shake on HP loss
- Pause / Resume / Reset via HUD buttons and keyboard shortcuts
- High score persisted to `localStorage`
- **Mobile**: virtual keyboard via hidden `<input>`, responsive compact HUD (≤520px), `visualViewport`-aware word spawning, tap-to-type hint
- **Arabic language**: full word list (5 tiers), RTL tile rendering, translated overlay (title, subtitle, instructions, buttons, stat labels, high-score line)
- **Language selector**: English / العربية buttons on the start and game-over overlays; language persists across resets

---

## Build History

### Initial release
- Single-file architecture (`typing-game.html`) — no build step, no dependencies
- Two-canvas rendering (starfield + particle FX)
- DOM word tiles with per-character typed/error feedback
- State machine: idle → playing → paused → game over
- 10 levels, 5 word tiers, combo system, Web Audio procedural sounds

### Mobile support (2026-06-30)
- Added `<input id="mobile-input">` (hidden 1×1px at top-left) as keyboard anchor
- Dual input routing: `keydown` on `document` (desktop) vs `input` event on hidden input (mobile), both call shared `processChar(key)`
- `isMobile` detection: `'ontouchstart' in window || navigator.maxTouchPoints > 0`
- `focusMobileInput()` / `blurMobileInput()` called by all lifecycle transitions
- `visualViewport.height` used for canvas resize and word spawn Y-position
- `@media (max-width: 520px)`: 44px HUD, icon-only buttons, Acc + Combo stats hidden, 16px tiles
- `#tap-hint` badge shown when keyboard dismissed mid-play; tapping game area or hint re-focuses input
- `touch-action: manipulation` on all buttons (removes 300ms tap delay)
- **Key fix**: `focus()` must be called synchronously (no setTimeout) and input must be within viewport — iOS silently blocks keyboard otherwise

### Arabic language + translated overlay (2026-06-30)
- Added `WORDS_AR` (5 tiers, grouped by Arabic character length) and `ALL_WORDS_AR_BY_TIER`
- `.word-tile.arabic` CSS: `direction: rtl; letter-spacing: 0` — browser shaping engine renders connected Arabic letterforms across `<span>` elements correctly
- `state.language` ('en' | 'ar') controls word selection in `pickWord()` and tile class in `spawnWord()`
- `mobileInput.lang` / `mobileInput.dir` hint OS to show Arabic keyboard layout
- Added `STRINGS.en` / `STRINGS.ar` constants with all overlay-visible UI text
- `overlayMode` ('start' | 'gameover') tracks which overlay screen is showing; set by `resetGame()` and `gameOver()`
- `applyOverlayStrings()` applies the current language's strings to the overlay and toggles `#overlay.rtl` for text direction
- `updateLangButton()` syncs `.lang-btn` active states + mobile input hints + calls `applyOverlayStrings()`
- Language selector (`#lang-selector`) moved from HUD to the overlay — two buttons (English / العربية) visible on start and game-over screens

---

## Known Gaps / Future Ideas

### Gameplay
- No game modes beyond the default (could add: Time Attack, Survival/endless, Practice mode with no HP loss)
- No power-ups or special events between levels
- Combo resets on any miss — could add a "shield" mechanic that absorbs one miss
- Words only move left; could add varied paths (diagonal, bouncing) at high levels
- No touch-based word selection on mobile — active tile is always the leftmost word

### UX / Polish
- No settings screen (sound on/off, theme color, word difficulty cap)
- No share/social feature after game over
- Leaderboard (would need a backend — currently local `localStorage` only)
- No orientation hint for landscape mode on phones (very little vertical space)
- Could add more visual flair: scanlines overlay, CRT flicker, tile spawn animation

### Language / Internationalization
- Only English and Arabic currently; framework (`STRINGS`, `WORDS_XX`, `ALL_WORDS_XX_BY_TIER`) is in place to add more languages
- High score is shared across both languages — could track separately per language
- Arabic word difficulty tiers could be further tuned (current grouping is by character count, not linguistic complexity)

### Technical
- Word lists are hardcoded; could load from an external JSON for easier curation
- No unit tests; all testing is manual in-browser
