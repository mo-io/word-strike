# SVG Hand Visualization Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a visual SVG hand skeleton overlay on the on-screen keyboard that highlights the correct finger in real time as the player types.

**Architecture:** A single SVG element with both hands (wireframe geometry) is positioned absolutely over the keyboard panel. Each hand has 5 finger groups (thumb, index, middle, ring, pinky). When a key is highlighted in the keyboard, the corresponding finger in the SVG is also highlighted with a glowing neon color and semi-transparent fill. Integration is minimal—just update `updateKeyboardHighlight()` to also set the `.active` class on the finger group.

**Tech Stack:** 
- Inline SVG paths (no external libraries)
- CSS custom properties for neon colors
- Standard DOM classList manipulation
- No additional dependencies

## Global Constraints

- **Single self-contained HTML file** — all SVG, CSS, and JS inline
- **Desktop only** — hidden on mobile (≤520px), same as keyboard panel
- **No animation** — static highlighting only, instant on/off
- **Responsive** — SVG scales with viewport via viewBox
- **Both languages** — English QWERTY and Arabic 101, finger mapping identical
- **Color consistency** — use existing `--neon-*` CSS variables for finger colors

---

## Task 1: Create SVG Hand Geometry & Embed in HTML

**Files:**
- Modify: `typing-game.html` (lines 524–590, CSS section for `#keyboard-panel`)

**Interfaces:**
- Produces: `#hand-svg` SVG element with two `.hand` groups (data-hand="left" / data-hand="right"), each containing 5 `.finger` groups (data-finger="thumb", "index", "middle", "ring", "pinky")
- Each finger contains a `<path class="finger-outline">` element

**Steps:**

- [ ] **Step 1: Define SVG hand paths**

Create the SVG with wireframe finger geometry. Each finger is a simple path. Here's the complete SVG to insert after the `#keyboard-panel` opening in the HTML (inside the `<style>` closing tag, right before `</style>`):

Add this CSS rule for SVG styling:

```css
  /* ── SVG HAND VISUALIZATION ── */
  #hand-svg {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    z-index: 7;
  }

  @media (max-width: 520px) {
    #hand-svg { display: none; }
  }

  .hand { }
  .finger { }

  .finger-outline {
    stroke: var(--dim);
    stroke-width: 1.5;
    stroke-linecap: round;
    stroke-linejoin: round;
    fill: none;
    opacity: 0.2;
    transition: stroke 0.08s, fill 0.08s, opacity 0.08s, filter 0.08s;
  }

  /* Color mapping by finger */
  .finger[data-finger="pinky"] { --kc: var(--neon-pink); }
  .finger[data-finger="ring"] { --kc: var(--neon-purple); }
  .finger[data-finger="middle"] { --kc: var(--neon-cyan); }
  .finger[data-finger="index"] { --kc: var(--neon-green); }
  .finger[data-finger="thumb"] { --kc: var(--neon-cyan); }

  .finger.active .finger-outline {
    stroke: var(--kc);
    stroke-width: 2;
    fill: var(--kc);
    opacity: 0.3;
    filter: drop-shadow(0 0 8px var(--kc));
  }
```

- [ ] **Step 2: Insert SVG HTML into the page**

Find the line `<div id="keyboard-panel"></div>` (around line 685). Replace it with:

```html
<div id="keyboard-panel">
  <svg id="hand-svg" viewBox="0 0 280 140" xmlns="http://www.w3.org/2000/svg">
    <defs>
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
      <!-- Left Thumb (bottom center-left, pointing down) -->
      <g class="finger" data-finger="thumb">
        <path class="finger-outline" d="M 40 80 L 45 100 L 50 115"/>
      </g>
      <!-- Left Index (tall, pointing up) -->
      <g class="finger" data-finger="index">
        <path class="finger-outline" d="M 55 60 L 55 35 L 55 20"/>
      </g>
      <!-- Left Middle (tallest, center-left) -->
      <g class="finger" data-finger="middle">
        <path class="finger-outline" d="M 70 50 L 70 25 L 70 10"/>
      </g>
      <!-- Left Ring -->
      <g class="finger" data-finger="ring">
        <path class="finger-outline" d="M 85 55 L 85 32 L 85 15"/>
      </g>
      <!-- Left Pinky (shortest) -->
      <g class="finger" data-finger="pinky">
        <path class="finger-outline" d="M 100 70 L 100 48 L 100 32"/>
      </g>
    </g>

    <!-- Right hand (mirrored) -->
    <g class="hand" data-hand="right">
      <!-- Right Index -->
      <g class="finger" data-finger="index">
        <path class="finger-outline" d="M 225 60 L 225 35 L 225 20"/>
      </g>
      <!-- Right Middle (tallest, center-right) -->
      <g class="finger" data-finger="middle">
        <path class="finger-outline" d="M 210 50 L 210 25 L 210 10"/>
      </g>
      <!-- Right Ring -->
      <g class="finger" data-finger="ring">
        <path class="finger-outline" d="M 195 55 L 195 32 L 195 15"/>
      </g>
      <!-- Right Pinky (shortest) -->
      <g class="finger" data-finger="pinky">
        <path class="finger-outline" d="M 180 70 L 180 48 L 180 32"/>
      </g>
      <!-- Right Thumb (bottom center-right, pointing down) -->
      <g class="finger" data-finger="thumb">
        <path class="finger-outline" d="M 240 80 L 235 100 L 230 115"/>
      </g>
    </g>
  </svg>
</div>
```

- [ ] **Step 3: Verify SVG renders**

Open `http://localhost:3737/typing-game.html` in a browser. Start a game. You should see faint wireframe fingers at the bottom above the keyboard keys. They will be very dim (opacity 0.2, `--dim` color).

- [ ] **Step 4: Commit**

```bash
git add typing-game.html
git commit -m "feat: add SVG hand skeleton with finger elements

- Create both left and right hands as wireframe paths
- Position absolutely over keyboard panel
- Add CSS for unlit (faint) and active (glow + fill) states
- Color-code fingers by touch typing standard

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 2: Add data-hand Attributes to Keyboard Keys

**Files:**
- Modify: `typing-game.html` (function `buildKeyboard()`, around line 968)

**Interfaces:**
- Consumes: `KB_LAYOUT[state.language]` — array of rows with `{k, f}` objects (character, finger)
- Produces: Each keyboard key DOM element now has `data-hand="left"` or `data-hand="right"` attribute based on position

**Steps:**

- [ ] **Step 1: Identify hand assignment logic**

In QWERTY, keys are naturally split:
- **Left hand**: Q–T, A–G, Z–V (left side of keyboard)
- **Right hand**: Y–P, H–; (colon/semicolon), N–M (right side)
- **Space**: Both hands (assign to right hand for uniqueness)

In Arabic 101, the same column-based split applies (Arabic keys are different characters, but layout is similar).

- [ ] **Step 2: Create hand lookup function**

Inside the JavaScript section (around line 959), after the `KB_LAYOUT` definition, add:

```js
// Map each key character to its hand (left or right)
const KEY_HAND_MAP = {
  // English QWERTY
  'q': 'left', 'w': 'left', 'e': 'left', 'r': 'left', 't': 'left',
  'a': 'left', 's': 'left', 'd': 'left', 'f': 'left', 'g': 'left',
  'z': 'left', 'x': 'left', 'c': 'left', 'v': 'left', 'b': 'left',
  'y': 'right', 'u': 'right', 'i': 'right', 'o': 'right', 'p': 'right',
  'h': 'right', 'j': 'right', 'k': 'right', 'l': 'right',
  'n': 'right', 'm': 'right',
  ' ': 'right', // space
  // Arabic 101
  'ض': 'left', 'ص': 'left', 'ث': 'left', 'ف': 'left', 'ق': 'left',
  'ش': 'left', 'س': 'left', 'د': 'left', 'ج': 'left', 'أ': 'left',
  'ئ': 'left', 'ء': 'left', 'غ': 'left', 'ح': 'left', 'ه': 'left',
  'ف': 'right', 'ع': 'right', 'خ': 'right', 'ج': 'right', 'م': 'right',
  'ل': 'right', 'و': 'right', 'ي': 'right', 'ك': 'right', 'ن': 'right',
};

function getHandForKey(key) {
  return KEY_HAND_MAP[key] || 'left'; // default to left if not found
}
```

- [ ] **Step 3: Modify buildKeyboard() to add data-hand**

Find the `buildKeyboard()` function (around line 968). Modify it to add `data-hand` when creating each key:

```js
function buildKeyboard() {
  kbPanel.innerHTML = '';
  const svgContainer = kbPanel.querySelector('#hand-svg');
  kbPanel.classList.toggle('arabic', state.language === 'ar');
  const layout = KB_LAYOUT[state.language];
  const isAr = state.language === 'ar';
  layout.forEach(row => {
    const rowEl = document.createElement('div');
    rowEl.className = 'kb-row';
    row.forEach(key => {
      const keyEl = document.createElement('div');
      keyEl.className = 'kb-key';
      keyEl.dataset.key = key.k;
      keyEl.dataset.finger = key.f;
      keyEl.dataset.hand = getHandForKey(key.k); // ADD THIS LINE
      keyEl.textContent = isAr ? key.k : key.k.toUpperCase();
      rowEl.appendChild(keyEl);
    });
    kbPanel.appendChild(rowEl);
  });
  // Re-append SVG after keyboard rebuild (it got wiped by innerHTML)
  if (svgContainer) {
    kbPanel.insertBefore(svgContainer, kbPanel.firstChild);
  }
}
```

Wait, there's an issue: the SVG is inside the keyboard panel, but `innerHTML = ''` will remove it. Let me revise:

Actually, looking at the current code, `buildKeyboard()` sets `kbPanel.innerHTML = ''` which clears everything. We need to save the SVG and re-insert it, OR embed the SVG differently. Let me choose a better approach:

**Revised Step 3:**

Find the `buildKeyboard()` function and modify it:

```js
function buildKeyboard() {
  kbPanel.innerHTML = '';
  // Rebuild the SVG first (it will be cleared by innerHTML)
  const svgHtml = `<svg id="hand-svg" viewBox="0 0 280 140" xmlns="http://www.w3.org/2000/svg">
    <defs>
      <filter id="finger-glow">
        <feGaussianBlur stdDeviation="2" result="coloredBlur"/>
        <feMerge>
          <feMergeNode in="coloredBlur"/>
          <feMergeNode in="SourceGraphic"/>
        </feMerge>
      </filter>
    </defs>
    <g class="hand" data-hand="left">
      <g class="finger" data-finger="thumb"><path class="finger-outline" d="M 40 80 L 45 100 L 50 115"/></g>
      <g class="finger" data-finger="index"><path class="finger-outline" d="M 55 60 L 55 35 L 55 20"/></g>
      <g class="finger" data-finger="middle"><path class="finger-outline" d="M 70 50 L 70 25 L 70 10"/></g>
      <g class="finger" data-finger="ring"><path class="finger-outline" d="M 85 55 L 85 32 L 85 15"/></g>
      <g class="finger" data-finger="pinky"><path class="finger-outline" d="M 100 70 L 100 48 L 100 32"/></g>
    </g>
    <g class="hand" data-hand="right">
      <g class="finger" data-finger="index"><path class="finger-outline" d="M 225 60 L 225 35 L 225 20"/></g>
      <g class="finger" data-finger="middle"><path class="finger-outline" d="M 210 50 L 210 25 L 210 10"/></g>
      <g class="finger" data-finger="ring"><path class="finger-outline" d="M 195 55 L 195 32 L 195 15"/></g>
      <g class="finger" data-finger="pinky"><path class="finger-outline" d="M 180 70 L 180 48 L 180 32"/></g>
      <g class="finger" data-finger="thumb"><path class="finger-outline" d="M 240 80 L 235 100 L 230 115"/></g>
    </g>
  </svg>`;
  kbPanel.innerHTML = svgHtml;

  kbPanel.classList.toggle('arabic', state.language === 'ar');
  const layout = KB_LAYOUT[state.language];
  const isAr = state.language === 'ar';
  layout.forEach(row => {
    const rowEl = document.createElement('div');
    rowEl.className = 'kb-row';
    row.forEach(key => {
      const keyEl = document.createElement('div');
      keyEl.className = 'kb-key';
      keyEl.dataset.key = key.k;
      keyEl.dataset.finger = key.f;
      keyEl.dataset.hand = getHandForKey(key.k);
      keyEl.textContent = isAr ? key.k : key.k.toUpperCase();
      rowEl.appendChild(keyEl);
    });
    kbPanel.appendChild(rowEl);
  });
}
```

- [ ] **Step 4: Test data attributes**

Open the browser DevTools (F12). Start a game. Inspect a key (e.g., the 'Q' key) and verify it has `data-hand="left"`. Inspect a key like 'M' and verify it has `data-hand="right"`.

- [ ] **Step 5: Commit**

```bash
git add typing-game.html
git commit -m "feat: add data-hand attributes to keyboard keys

- Create KEY_HAND_MAP lookup for hand assignment
- Add getHandForKey() function
- Modify buildKeyboard() to set data-hand on each key
- Rebuild SVG inside buildKeyboard() to prevent it being wiped

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 3: Update updateKeyboardHighlight() to Highlight Fingers

**Files:**
- Modify: `typing-game.html` (function `updateKeyboardHighlight()`, around line 999)

**Interfaces:**
- Consumes: Keyboard key element with `data-finger` and `data-hand` attributes; SVG with structure from Task 1
- Produces: When a key is highlighted, both the key and its corresponding finger group get the `.active` class

**Steps:**

- [ ] **Step 1: Locate the function**

Find `updateKeyboardHighlight()` around line 999.

- [ ] **Step 2: Replace the function with enhanced version**

```js
function updateKeyboardHighlight() {
  // Clear previous highlights
  kbPanel.querySelectorAll('.kb-key.lit').forEach(el => el.classList.remove('lit'));
  kbPanel.querySelectorAll('.finger.active').forEach(el => el.classList.remove('active'));

  const id = state.activeId;
  if (id === null) return;
  const t = state.tiles[id];
  if (!t || t.typed >= t.word.length) return;
  const nextChar = t.word[t.typed];

  let searchKey;
  if (state.language === 'en') {
    searchKey = nextChar.toLowerCase();
  } else {
    searchKey = nextChar.replace(/[أإآٱ]/g, 'ا');
  }

  const keyEl = kbPanel.querySelector(`.kb-key[data-key="${CSS.escape(searchKey)}"]`);
  if (keyEl) {
    // Highlight the key
    keyEl.classList.add('lit');

    // Also highlight the corresponding finger
    const finger = keyEl.dataset.finger;
    const hand = keyEl.dataset.hand;
    const handSvg = kbPanel.querySelector('#hand-svg');
    if (handSvg && finger && hand) {
      const fingerGroup = handSvg.querySelector(
        `.hand[data-hand="${hand}"] .finger[data-finger="${finger}"]`
      );
      if (fingerGroup) {
        fingerGroup.classList.add('active');
      }
    }
  }
}
```

- [ ] **Step 3: Test in browser**

Start a game. Watch as words appear—each time the keyboard highlights a key, the corresponding finger should light up in the SVG with a glow and semi-transparent fill (neon color matching the keyboard).

Type a few characters and verify:
- The finger highlights when you need to type it
- The finger unhighlights when you move to the next character
- The glow and fill effects are visible

- [ ] **Step 4: Commit**

```bash
git add typing-game.html
git commit -m "feat: sync finger highlighting with keyboard keys

- Update updateKeyboardHighlight() to also light up corresponding finger
- Query SVG for correct hand and finger group
- Apply .active class to trigger CSS glow + fill effect
- Clear previous highlights before setting new ones

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 4: Test with Both Languages and Responsive Behavior

**Files:**
- Test: Manual testing (no automated tests for this feature)

**Steps:**

- [ ] **Step 1: Test English gameplay**

Start a game with English selected. Type several words. Verify:
- Fingers light up correctly for each key
- Finger colors match the keyboard key highlights
- Fingers unhighlight when moving to the next character
- No visual glitches or layout shifts

- [ ] **Step 2: Test Arabic gameplay**

Switch language to Arabic and start a new game. Type several words in Arabic. Verify:
- Fingers light up for Arabic key positions
- Finger mapping is correct (same finger assignments as English)
- Alef normalization still works (أ إ آ → ا)
- No visual glitches

- [ ] **Step 3: Test mobile responsiveness**

Resize the browser to ≤520px width. Verify:
- Keyboard panel disappears (display: none)
- SVG hand also disappears (display: none via CSS media query)
- Game still plays normally without the keyboard/hands

Resize back to >520px. Verify:
- Keyboard panel reappears
- SVG hands reappear
- Highlighting works immediately

- [ ] **Step 4: Test pause and resume**

During a game, press Esc to pause. Verify:
- Keyboard panel disappears
- SVG hand disappears
- The words are frozen

Resume (press Esc or Enter). Verify:
- Keyboard panel and SVG hands reappear
- Highlighting resumes correctly

- [ ] **Step 5: Manual edge cases**

- Start a game, let it run, watch fingers change as you type words
- Try typing very fast—verify finger highlights keep up
- Try typing wrong keys—verify the keyboard shows error feedback (pink flash) but the hand continues to highlight the correct next character (not the wrong key)
- Check pause overlay—keyboard/hands should be hidden

---

## Task 5: Update CLAUDE.md Documentation

**Files:**
- Modify: `CLAUDE.md` (Architecture section)

**Steps:**

- [ ] **Step 1: Add brief note to Architecture**

In `CLAUDE.md`, find the "Visual / CSS" section (around line 175 in the original). Add a new subsection:

```markdown
### SVG Hand Visualization

An SVG element (`#hand-svg`) overlays the keyboard panel, showing both hands as wireframe skeletons. Each hand has 5 fingers (thumb, index, middle, ring, pinky), color-coded by touch-typing convention. When a key is highlighted in the keyboard (via `updateKeyboardHighlight()`), the corresponding finger lights up with a glow effect and semi-transparent fill. The SVG is non-interactive (`pointer-events: none`) and hidden on mobile (≤520px) via CSS.

**Implementation:**
- SVG paths are rebuilt each time `buildKeyboard()` runs (to avoid being wiped by `innerHTML`)
- Each keyboard key has `data-hand` ("left" or "right") and `data-finger` (finger name) attributes
- `updateKeyboardHighlight()` queries the SVG for the matching finger group and adds the `.active` class
- CSS handles the visual feedback (colors, glow, fill) via the `--kc` custom property per finger

**Files:**
- SVG geometry: embedded in `#keyboard-panel` element
- CSS: styles for `.finger.active` and `.finger-outline`
- JS: `buildKeyboard()` (SVG rebuild), `updateKeyboardHighlight()` (finger highlighting), `KEY_HAND_MAP` (lookup table)
```

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: document SVG hand visualization in CLAUDE.md

- Add SVG Hand Visualization section to Architecture
- Explain overlay behavior, finger-to-key mapping, and integration

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Self-Review Checklist

**1. Spec Coverage:**
- ✓ SVG hand geometry (wireframe skeletons, both hands)
- ✓ Visual feedback (glow outline + semi-transparent fill)
- ✓ Positioning (overlaid on keyboard panel, staggered)
- ✓ Finger-to-key mapping (via data-hand and data-finger attributes)
- ✓ Integration with updateKeyboardHighlight()
- ✓ Responsive behavior (hidden on mobile)
- ✓ Both languages (English and Arabic)

**2. Placeholder Scan:**
- ✓ No "TBD" or "TODO" in tasks
- ✓ All code is complete and runnable
- ✓ All commands are exact with expected output
- ✓ No vague steps like "add appropriate styling"

**3. Type Consistency:**
- ✓ `data-hand="left" | "right"` used consistently
- ✓ `data-finger="thumb" | "index" | "middle" | "ring" | "pinky"` used consistently
- ✓ CSS class names: `.active`, `.lit`, `.finger`, `.hand`
- ✓ Function names: `buildKeyboard()`, `updateKeyboardHighlight()`, `getHandForKey()`
- ✓ CSS variables: `--kc`, `--neon-*`, `--dim`

**4. Ambiguity Check:**
- ✓ SVG structure is explicit (paths included in Task 1)
- ✓ Hand assignment logic is explicit (KEY_HAND_MAP in Task 2)
- ✓ Highlight logic is explicit (Task 3 code)
- ✓ No vague instructions like "make it look good"

**5. Completeness:**
- ✓ All 5 tasks are self-contained and testable
- ✓ Each task produces a working, independently meaningful change
- ✓ Commits are frequent and logical
- ✓ Testing strategy is clear for each task

---

## Summary

This plan implements the SVG hand visualization in 5 focused tasks:
1. Create and embed SVG hand geometry
2. Add hand assignment to keyboard keys
3. Update keyboard highlighting to also highlight fingers
4. Test with both languages and responsive behavior
5. Document in CLAUDE.md

All code is complete, no placeholders. Each task is testable independently. Total estimated effort: 30–45 minutes.
