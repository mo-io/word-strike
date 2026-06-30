# Word Strike — Development Memory

Running log of what has been built, decisions made, and what comes next. Read this alongside `CLAUDE.md` (which covers architecture and how to work with the code).

---

## Current State (2026-06-30)

The game is fully playable on **desktop and mobile**. It is live on Vercel and auto-deploys from `master`.

### What works
- 10-level progression with escalating speed, word length, and spawn rate
- 5-tier word pool; weighted random selection per level
- Combo multiplier (×1 / ×2 / ×4 / ×8) with score scaling
- WPM (rolling 10-second window), Accuracy %, Score, HP, Level HUD
- Particle explosions, starfield background, procedural Web Audio
- Screen shake on HP loss
- Pause / Resume / Reset via HUD buttons and keyboard shortcuts
- High score persisted to `localStorage`
- **Mobile**: virtual keyboard via hidden `<input>`, responsive compact HUD (≤520px), `visualViewport`-aware word spawning, tap-to-type hint

---

## Build History

### Initial release
- Single-file architecture (`typing-game.html`) — no build step, no dependencies
- Two-canvas rendering (starfield + particle FX)
- DOM word tiles with per-character typed/error feedback
- State machine: idle → playing → paused → game over
- 10 levels, 5 word tiers, combo system, Web Audio procedural sounds

### Mobile support (2026-06-30)
- Added `<input id="mobile-input">` (hidden, off-screen) as keyboard anchor
- Dual input routing: `keydown` on `document` (desktop) vs `input` event on hidden input (mobile), both call shared `processChar(key)`
- `isMobile` detection: `'ontouchstart' in window || navigator.maxTouchPoints > 0`
- `focusMobileInput()` / `blurMobileInput()` called by all lifecycle transitions
- `visualViewport.height` used for canvas resize and word spawn Y-position
- `@media (max-width: 520px)`: 44px HUD, icon-only buttons, Acc + Combo stats hidden, 16px tiles
- `#tap-hint` badge shown when keyboard dismissed mid-play; tapping game area or hint re-focuses input
- `touch-action: manipulation` on all buttons (removes 300ms tap delay)

---

## Known Gaps / Future Ideas

### Mobile UX
- No touch-based word selection — active tile is always the leftmost word; player cannot tap a specific word to target it
- Landscape orientation on phones gives very little vertical room; could show an orientation hint or flatten tile layout
- No swipe gestures (e.g., swipe down to pause)

### Gameplay
- No game modes beyond the default (could add: Time Attack, Survival/endless, Practice mode with no HP loss)
- No power-ups or special events between levels
- Combo resets on any miss — could add a "shield" mechanic that absorbs one miss
- Words only move left; could add varied paths (diagonal, bouncing) at high levels

### Polish
- No settings screen (could expose: sound on/off, theme color, word difficulty cap)
- No share/social feature after game over
- Leaderboard (would need a backend — currently local `localStorage` only)
- Could add more visual flair: scanlines overlay, CRT flicker, tile spawn animation

### Technical
- Word lists are hardcoded; could load from an external JSON for easier curation
- No unit tests; all testing is manual in-browser
