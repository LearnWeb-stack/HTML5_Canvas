# HTML5_Canvas
# JavaScript Animation & Collision Tutorial

A complete beginner-friendly tutorial covering four core vanilla JavaScript concepts for building DOM-based animations and collision-driven interactions — no canvas, no libraries, no frameworks required.

---

## What's Inside

The tutorial is structured into four sections, each with a syntax reference, plain-English explanation, and a simple working code example.

| # | Topic | Covers |
|---|-------|--------|
| 01 | `requestAnimationFrame` | Smooth animation loops, start/stop, FPS |
| 02 | `getBoundingClientRect` | Element position, size, DOMRect properties |
| 03 | Collision Detection | AABB formula, `isColliding()` function |
| 04 | Jump on Collision | Gravity, velocity, `hasJumped` lock |

---

## Section Breakdown

### 01 — requestAnimationFrame

**What it is:** A browser API that runs your function just before the next screen repaint (~60fps), perfectly in sync with the display.

**Syntax:**
```js
let frameId = requestAnimationFrame(callback);
cancelAnimationFrame(frameId);
```

**Key points:**
- Always call `requestAnimationFrame(itself)` inside the callback to keep the loop running
- Returns a numeric ID — pass it to `cancelAnimationFrame()` to stop
- Automatically pauses when the browser tab is hidden (unlike `setInterval`)
- Callback receives a timestamp you can use to calculate delta time

---

### 02 — getBoundingClientRect

**What it is:** Returns a `DOMRect` object with the exact pixel position and size of any HTML element, measured relative to the viewport.

**Syntax:**
```js
const rect = element.getBoundingClientRect();
```

**DOMRect properties:**

| Property | Description |
|----------|-------------|
| `rect.top` | Pixels from viewport top to element top edge |
| `rect.left` | Pixels from viewport left to element left edge |
| `rect.right` | Pixels from viewport left to element right edge |
| `rect.bottom` | Pixels from viewport top to element bottom edge |
| `rect.width` | Rendered width in pixels |
| `rect.height` | Rendered height in pixels |

**Key points:**
- Call it fresh every frame — values change as the element moves
- Works after CSS transforms, scrolling, and layout changes
- Works on any visible element: `div`, `img`, `button`, `canvas`, etc.

---

### 03 — Collision Detection

**What it is:** AABB (Axis-Aligned Bounding Box) — checks whether two elements overlap by comparing their edges every animation frame.

**The `isColliding()` function:**
```js
function isColliding(a, b) {
  const r1 = a.getBoundingClientRect();
  const r2 = b.getBoundingClientRect();

  return !(
    r1.right  < r2.left   ||   // A is left of B
    r1.left   > r2.right  ||   // A is right of B
    r1.bottom < r2.top    ||   // A is above B
    r1.top    > r2.bottom      // A is below B
  );
}
```

**Logic:** Two boxes are colliding when none of the four "no-overlap" conditions are true. Negate them with `!` and you get the collision check.

**Key points:**
- Never cache rect values — always call `getBoundingClientRect()` each frame
- AABB only works on axis-aligned (non-rotated) rectangles
- Add a small padding value (e.g. 5px) for more forgiving hit detection

---

### 04 — Jump on Collision

**What it is:** When `isColliding()` returns `true`, apply an upward velocity to the element. Gravity pulls it back down each frame.

**The three new variables:**
```js
let y         = 0;       // height above ground (px)
let velY      = 0;       // current vertical speed
let hasJumped = false;   // one-shot lock — prevents re-triggering

const GRAVITY    = 0.6;  // added to velY every frame
const JUMP_FORCE = -12;  // negative = upward direction
```

**The complete loop:**
```js
function loop() {
  // 1. Horizontal movement
  x += velX;
  if (x >= maxX || x <= 0) velX *= -1;

  // 2. Apply gravity every frame
  velY += GRAVITY;
  y    -= velY;
  if (y <= 0) { y = 0; velY = 0; hasJumped = false; }

  // 3. Apply to DOM
  player.style.left   = x + 'px';
  player.style.bottom = (groundOffset + y) + 'px';

  // 4. Collision → jump (fires only once per contact)
  if (isColliding(player, wall) && !hasJumped) {
    velY      = JUMP_FORCE;
    hasJumped = true;
  }

  requestAnimationFrame(loop);
}
```

**Why `hasJumped` matters:** Without it, `isColliding()` stays `true` for several overlapping frames. Each frame would reset `velY` back to `JUMP_FORCE`, cancelling the jump before it goes anywhere. `hasJumped = true` locks the trigger after the first hit and only unlocks it when the element lands (`y <= 0`).

**Visual state guide:**

| State | Condition | Action |
|-------|-----------|--------|
| Moving normally | `isColliding = false`, `y = 0` | Walk animation |
| Collision hit | `isColliding = true`, `hasJumped = false` | Set `velY = JUMP_FORCE`, lock |
| In the air | `y > 0`, `hasJumped = true` | Apply gravity |
| Landing | `y <= 0` | Reset `y`, `velY`, unlock |

---

## Quick Reference

| API / Variable | What it does |
|----------------|--------------|
| `requestAnimationFrame(fn)` | Schedules `fn` before the next browser paint (~60fps) |
| `cancelAnimationFrame(id)` | Cancels a scheduled frame using the returned ID |
| `el.getBoundingClientRect()` | Returns position and size of `el` relative to viewport |
| `velY += GRAVITY` | Increases downward speed each frame (simulates gravity) |
| `y -= velY` | Moves element up (negative `velY`) or down (positive `velY`) |
| `hasJumped = true` | Locks jump so it only fires once per collision contact |
| `hasJumped = false` | Unlocks jump when element lands back on the ground |

---

## Files

| File | Description |
|------|-------------|
| `JS-Animation-Collision-Tutorial.docx` | Full tutorial document with code examples and explanations |
| `README.md` | This file |

---

## Prerequisites

- Basic knowledge of HTML and JavaScript
- A browser (Chrome, Firefox, Edge, Safari — all supported)
- No build tools, no npm, no frameworks needed

---

## How to Run the Examples

Every code example in the tutorial runs in a plain HTML file. Create a file like this and open it in your browser:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Example</title>
  <style>
    #scene {
      position: relative;
      width: 100%;
      height: 200px;
      background: #f5f5f5;
      overflow: hidden;
    }
    #player {
      position: absolute;
      width: 44px;
      height: 44px;
      background: steelblue;
      border-radius: 6px;
      bottom: 0;
      left: 20px;
    }
  </style>
</head>
<body>
  <div id="scene">
    <div id="player"></div>
  </div>
  <button onclick="startLoop()">Start</button>

  <script>
    // Paste any example from the tutorial here
  </script>
</body>
</html>
```

---

## License

Free to use for personal learning and educational purposes.
