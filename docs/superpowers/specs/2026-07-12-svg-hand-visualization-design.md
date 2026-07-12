# SVG Hand Visualization Design — Word Strike

**Date:** 2026-07-12  
**Phase:** 3 (UI enhancement)  
**Status:** Design approved

---

## Overview

Add a visual SVG hand skeleton to the on-screen keyboard panel, showing both hands in a typing posture. Fingers light up with neon glow and color-fill when they're needed to type the next character. This provides immediate visual feedback about which finger should move next, reinforcing touch-typing muscle memory.

---

## Goals & Success Criteria

- **Primary**: Visual finger guidance during gameplay—player sees at a glance which finger to use
- **Secondary**: Neon cyberpunk aesthetic consistency (wireframe style, color-coded by finger)
- **Non-goal**: Animation or realistic hand modeling (static highlighting only)

**Success criteria:**
- Fingers highlight correctly in sync with keyboard key highlighting
- Works for both English (QWERTY) and Arabic (101) layouts
- No performance impact or layout shift (overlay mode, pointer-events: none)
- Responsive (hidden on mobile ≤520px, same as keyboard)

---

## Design Details

### SVG Structure & Finger Representation

A single `<svg id="hand-svg">` element renders both hands as wireframe skeletons:

```svg
<svg id="hand-svg" viewBox="0 0 280 140" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <!-- filter for glow effect -->
    <filter id="finger-glow">
      <feGaussianBlur stdDeviation="2" result="coloredBlur"/>
      <feMerge>
        <feMergeNode in="coloredBlur"/>
        <feMergeNode in="SourceGraphic"/>
      </feMerge>
    </filter>
  </defs>

  <!-- Left hand -->
  <g class="hand" data-hand="left">
    <g class="finger" data-finger="thumb">
      <path class="finger-outline" d="..."/>
    </g>
    <g class="finger" data-finger="index">
      <path class="finger-outline" d="..."/>
    </g>
    <g class="finger" data-finger="middle">
      <path class="finger-outline" d="..."/>
    </g>
    <g class="finger" data-finger="ring">
      <path class="finger-outline" d="..."/>
    </g>
    <g class="finger" data-finger="pinky">
      <path class="finger-outline" d="..."/>
    </g>
  </g>

  <!-- Right hand (mirrored) -->
  <g class="hand" data-hand="right">
    <!-- same structure -->
  </g>
</svg>
```

**Geometry:**
- Left hand: 5 fingers positioned from left side, spread upward
- Right hand: 5 fingers positioned from right side, spread upward (mirrored)
- Each finger is a simple path of 2–3 line segments (representing metacarpal + phalanges), no complex geometry
- Sizing: SVG viewBox 280×140, renders at full keyboard panel width on desktop

---

### Visual Style: Wireframe & Neon Highlighting

**Unlit state** (default, all fingers):
- Stroke: 1.5px, `color: var(--dim)`, `opacity: 0.2` (very faint)
- Fill: none
- No glow

**Lit state** (when `.active` class is present on the finger group):
- Stroke: 2px, `color: var(--kc)` (where `--kc` is the finger's neon color)
- Fill: `var(--kc)` with `opacity: 0.2` (semi-transparent fill)
- Filter: `drop-shadow(0 0 8px var(--kc))` for glow effect
- Subtle transform: `scale(1.08)` on the stroke to emphasize the highlight

**Color mapping** (matches keyboard key colors):
- Pinky: `--neon-pink` (#ff2d78)
- Ring: `--neon-purple` (#bf5fff)
- Middle: `--neon-cyan` (#00f5ff)
- Index: `--neon-green` (#39ff14)
- Thumb: `--neon-cyan` (both hands share space, so use middle finger's cyan for consistency)

---

### Positioning & Layout

The SVG is positioned **absolutely within the keyboard panel**:

- **Position**: `position: absolute; inset: 0; width: 100%; height: 100%`
- **Z-index**: above the keyboard keys (keys have `position: relative` within flex rows)
- **Interactivity**: `pointer-events: none` so clicks pass through to keys

**Hand placement within the SVG**:
- Left hand: positioned toward left side, slightly higher (suggest natural home position)
- Right hand: positioned toward right side, slightly lower (offset from left for visual balance)
- Hands do NOT overlap
- Keyboard keys remain visible beneath and around the fingers

---

### Finger-to-Key Mapping

The keyboard layout already has `data-finger` attributes on each key (pinky, ring, middle, index). The SVG hand mirrors this mapping:

**English QWERTY:**
- Left Pinky: Q, A, Z
- Left Ring: W, S, X
- Left Middle: E, D, C
- Left Index: R, T, F, G, V, B
- Left Thumb: Space
- Right Index: Y, U, H, J, N, M
- Right Middle: I, K
- Right Ring: O, L
- Right Pinky: P
- Right Thumb: Space

**Arabic 101:**
- Same finger assignments, different keys (by character not layout name)

---

### Highlighting Logic & Integration

When `updateKeyboardHighlight()` runs and determines the next key needed:

1. It finds the keyboard key element and its `data-finger` attribute
2. It lights the key with the `.lit` class (existing behavior)
3. **NEW**: It also lights the corresponding finger in the hand SVG:
   ```js
   const finger = keyEl.dataset.finger;
   const hand = /* determine from key position or a lookup map */;
   const fingerGroup = handSvg.querySelector(
     `.hand[data-hand="${hand}"] .finger[data-finger="${finger}"]`
   );
   if (fingerGroup) fingerGroup.classList.add('active');
   ```

4. CSS will apply the glow + fill effect when `.active` is set

**Key insight**: The same finger may be needed by multiple keys (e.g., left index handles R, T, F, G, V, B). The SVG highlights the entire finger group when *any* of its keys is the next character.

---

### Hand Determination Logic

During keyboard layout build (`buildKeyboard()`), add `data-hand="left"` or `data-hand="right"` to each key element based on its position in the layout. This makes finger highlighting simple:

```js
// In updateKeyboardHighlight, after finding keyEl:
const hand = keyEl.dataset.hand; // "left" or "right"
const finger = keyEl.dataset.finger; // "pinky", "ring", etc.
const fingerGroup = handSvg.querySelector(
  `.hand[data-hand="${hand}"] .finger[data-finger="${finger}"]`
);
if (fingerGroup) fingerGroup.classList.add('active');
```

Hand assignment is based on the keyboard layout (e.g., all keys in the left half of QWERTY are `data-hand="left"`).

---

### Responsive Behavior

- **Desktop (>520px)**: SVG hand and keyboard panel both visible, SVG overlaid on keys
- **Mobile (≤520px)**: Keyboard panel hidden via `display: none !important`, SVG also hidden

---

## Implementation Tasks

1. **Create SVG paths** for both hands (wireframe geometry)
2. **Embed SVG in HTML** (within `#keyboard-panel` or as a peer element)
3. **Add CSS** for finger colors and `.active` state styling
4. **Update `updateKeyboardHighlight()`** to also highlight the corresponding finger in the SVG
5. **Add hand determination logic** (determine left/right from key position or data attribute)
6. **Test** with both English and Arabic layouts
7. **Verify** finger highlighting syncs correctly with key highlighting during gameplay

---

## Scope & Constraints

- **Single SVG**, both hands in one element (not separate)
- **No animation** — fingers light up instantly when needed, dim instantly when no longer needed
- **No interaction** — SVG is non-interactive (overlay only)
- **Backward compatible** — existing keyboard functionality unchanged
- **Self-contained** — all SVG paths and styling inline in HTML (no external assets)

---

## Open Questions / Decisions Pending

- **Exact SVG geometry**: Finger paths will be finalized during implementation to match keyboard layout visually
- **Hand positioning offsets**: Precise pixel offsets for left/right hand placement will be tuned during CSS refinement

---

## Files to Modify

| File | Change |
|------|--------|
| `typing-game.html` | Add SVG element, CSS for finger styling, update `updateKeyboardHighlight()` |
| `CLAUDE.md` | Add brief note to Architecture → "Visual / CSS" section about the hand SVG |

---

## Rollback Plan

If the SVG doesn't work well (performance, visual clutter, etc.):
- Remove the SVG HTML
- Remove CSS for `.finger.active`
- Remove the hand-highlighting code from `updateKeyboardHighlight()`
- Keyboard highlighting remains unchanged

---

## References

- Keyboard layout: `KB_LAYOUT[state.language]` in `typing-game.html`
- Keyboard highlighting: `updateKeyboardHighlight()` function
- Finger colors: CSS custom properties `--neon-pink`, `--neon-purple`, `--neon-cyan`, `--neon-green`
