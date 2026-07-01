# Word Strike ⌨️

A neon cyberpunk typing-speed trainer. Words drift left across the screen as glowing tiles — type them before they escape, chain combos for score multipliers, and climb through 10 levels of escalating speed and difficulty.

Single self-contained HTML file. No build step, no dependencies, no external assets.

## Play

Live on Vercel: deploys automatically from `master`.

## Run locally

Requires Python (no Node/npm needed):

```bash
python -m http.server 3737
```

Then open `http://localhost:3737/typing-game.html`.

## How to play

- Type the currently highlighted (glowing) word to destroy it.
- Letting a word scroll off the left edge costs a life (♥) and resets your combo.
- Game ends at 0 HP. Reach level 10 for **Infinite Mode**.

**HUD:** WPM (rolling 10s window) · Accuracy · Score · Combo multiplier · Level · Hearts

**Scoring:** `word length × 10 × combo` points per word. Combo escalates ×1 → ×2 → ×4 → ×8 and resets on any miss. High score is saved locally in your browser.

**Keyboard shortcuts:**

| Key | Action |
|-----|--------|
| `Esc` | Pause / Resume |
| `Enter` | Start / Resume |
| `R` | Reset (when paused or on game-over screen) |

## Features

- 10 levels of escalating speed and word difficulty, plus Infinite Mode at level 10
- Combo multiplier system with particle-explosion feedback
- Full mobile support — virtual keyboard, responsive HUD, touch-friendly layout
- English and Arabic word sets with a language selector and RTL rendering
- Procedural sound effects (Web Audio API — no audio files)
- Neon starfield background and screen-shake on hit

## Tech

Everything — HTML, CSS, and JavaScript — lives in `typing-game.html`. Rendering uses two `<canvas>` layers (starfield + particle effects) plus DOM elements for word tiles, which lets Arabic text shape and render correctly.

See `CLAUDE.md` for the full architecture writeup.

## Deployment

`vercel.json` rewrites `/` → `/typing-game.html` so the root URL serves the game directly.

## License

[MIT](LICENSE) © 2026 Mo
