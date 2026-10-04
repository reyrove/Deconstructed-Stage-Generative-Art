# Deconstructed-Stage-Generative-Art

A seed-based generative art catalogue for **Deconstructed Stage**, a computational Suprematist composition system. Built as a single-file HTML page with p5.js, styled as a gallery-ready portfolio and licensing surface.

**Live:** `https://reyrove.github.io/Deconstructed-Stage-Generative-Art/`

---

## Overview

**Deconstructed Stage** renders a grid of sheared planes from numeric seeds. Each composition places a 2–3 × 2–3 lattice of quadrilaterals in a Suprematist palette of red, blue, yellow, and black on a paper-toned ground. Every plane tilts in its own direction and magnitude, and roughly 15% of cells are left empty — creating rhythm and breathing room inside the composition.

The page functions as a **catalogue and licensing surface** for fashion houses, textile studios, and surface designers. Every composition is reproducible by seed, meaning the same number always produces the identical arrangement.

---

## Features

### Generative Engine (p5.js)

- **2–3 × 2–3 grid** — 4 to 9 cells, of which about 85% carry a sheared plane
- **Suprematist palette** — red, blue, yellow, black, from the vocabulary of Kazimir Malevich
- **Per-plane shear** — each quadrilateral shifts its bottom edge by 18%–30% of the cell size, in a random direction
- **Black outline** — every plane carries a thin black outline at `cs × 0.003`
- **Hand-drawn frame** — a heavy border at 4% inset, `cs × 0.006` stroke weight
- **Paper ground** — HSB(40, 10, 95), a warm off-white
- **Empty cells** — 15% of positions stay empty, breaking the grid's regularity

### Same Seed → Many Surfaces

A single seed drives every instance on the page:

| Instance | Purpose | Tiling |
|----------|---------|--------|
| **Cover** | Hero composition | 1 × 1 |
| **Plate** | Main work (master) | 1 × 1 |
| **Frame** | Framed archival print | 1 × 1 |
| **Print** | Archival print surface | 1 × 1 |
| **Scarf** | Silk scarf (3:1) | 3 × 1 |
| **Textile** | Fabric yardage (4:3) | 2 × 2 |
| **Wall** | Wallpaper (2:3) | 1 × 2 |
| **Archive** | 8 unique seeds | 1 × 1 each |

### Controls

- **New Seed** button / `R` key — regenerate with a fresh seed
- **Download** button / `S` key — export the plate canvas as PNG
- **Click any archive plate** — load that seed into the main view

### Performance

- Debounced resize handling (250ms)
- `IntersectionObserver` pauses off-screen sketches
- `visibilitychange` pauses rendering when the tab is hidden
- `pixelDensity(1)` for a lighter canvas footprint
- Two-frame render (`maxFrames: 2`) so tiles draw once and stop

---

## File Structure

```
Deconstructed-Stage-Generative-Art/
├── index.html          # Single-file page (structure + styles + generative engine)
├── README.md
└── images/
    ├── fav.svg         # Favicon
    ├── tote.png        # Mockup: tote bag
    ├── tee.png         # Mockup: t-shirt
    └── cushion.png     # Mockup: cushion
```

No build step. No dependencies to install. p5.js is loaded from a CDN.

---

## Getting Started

1. Clone the repo:
   ```bash
   git clone https://github.com/reyrove/Deconstructed-Stage-Generative-Art.git
   cd Deconstructed-Stage-Generative-Art
   ```
2. Add the three mockup images to `images/` (optional — they appear in the Commission section).
3. Open `index.html` in a browser.

That's it. The page self-initialises on `DOMContentLoaded`.

---

## How the Generative System Works

### Seeded Random

A linear congruential generator replaces p5's native randomness inside each sketch:

```js
class SeededRandom {
  constructor(s) { this.state = (Math.abs(s) % 2147483647) || 1; }
  rand() { this.state = (this.state * 9301 + 49297) % 233280; return this.state / 233280; }
  range(min, max) { return this.rand() * (max - min) + min; }
  int(min, max) { return Math.floor(this.range(min, max + 1)); }
  pick(arr) { return arr[Math.floor(this.rand() * arr.length)]; }
}
```

Every aesthetic decision — grid size, plane colour, shear direction, shear magnitude, empty-cell positions — is drawn from this generator in a fixed order. Same seed, same sequence, same composition.

### Parameter Space (per seed)

| Parameter | Range |
|-----------|-------|
| Grid size | 2 × 2 to 3 × 3 |
| Canvas margin | 8% of canvas |
| Plane size | 65% of available area, then divided by grid |
| Plane origin | 18% from each edge |
| Shear direction | ±1 (two options) |
| Shear magnitude | 18% – 30% of cell |
| Empty-cell probability | 15% |
| Frame stroke weight | `cs × 0.006` |
| Outline stroke weight | `cs × 0.003` |

### The Four Colours

The palette is fixed to four Suprematist anchors:

```js
const PALETTE = [
  [0, 80, 85],    // red
  [220, 80, 80],  // blue
  [50, 90, 90],   // yellow
  [0, 0, 20]      // black
];
```

Each plane picks one at random from this list. The result is a composition that always reads as the same four colours, rearranged in a new rhythm. This is the core logic of Suprematism — a small vocabulary of flat, saturated shapes in infinite arrangements.

### The Shear

Each plane is drawn as a quadrilateral whose bottom edge is shifted horizontally:

```js
const offset = dir * cell * sr.range(0.18, 0.3);

// Quadrilateral:
// (0, 0)                → top-left
// (s, 0)                → top-right
// (s + offset, s)       → bottom-right
// (offset, s)           → bottom-left
```

The result is a plane that leans left or right, but never so far that it escapes its cell. A composition of several leaning planes reads as a stage that is tilting, floating, or on the edge of collapsing — hence the title.

### The Frame

A heavy black border sits at 4% inset from the canvas edge, drawn at 0.6% of canvas for weight. This frames the composition as a self-contained object, in the tradition of Constructivist posters and Suprematist canvases.

### Tiling Logic

Rectangular surfaces (scarf, textile, wall) tile the square composition N×M. Each tile is drawn into its own translated coordinate space, centred within its cell:

```js
const tileW = canvasW / tileCols;
const tileH = canvasH / tileRows;
const tileSize = Math.min(tileW, tileH);
// draw square of side tileSize at (ox, oy) for each cell
```

This preserves the square composition's proportions regardless of surface aspect ratio.

---

## Customisation

### Change the default seed

Edit the init function:

```js
function init() {
  seed = 123456789;  // fixed seed instead of random
  // ...
}
```

### Adjust the grid size

```js
const gridCount = sr.int(2, 3);  // change to sr.int(2, 4) for a bigger grid
```

Note: larger grids make individual planes smaller, so the composition reads more as a mosaic and less as a Suprematist arrangement. The sweet spot is 2–3.

### Adjust the shear

```js
const offset = dir * cell * sr.range(0.18, 0.3);
```

- **Smaller range** (e.g., `0.1, 0.2`) — subtler lean, closer to a regular grid.
- **Larger range** (e.g., `0.3, 0.5`) — dramatic tilt, more dynamic composition.

### Adjust the empty-cell probability

```js
if (sr.rand() < 0.85) {  // change to 0.7 for more empty cells, 0.95 for fewer
```

Lower values leave more empty space, which increases the sense of rhythm and pause. Higher values make the grid denser.

### Change the palette

Edit the `PALETTE` constant:

```js
const PALETTE = [
  [0, 80, 85],    // red
  [220, 80, 80],  // blue
  [50, 90, 90],   // yellow
  [0, 0, 20]      // black
];
```

Add or remove HSB triples. Two colours gives a diptych feel; six colours approaches Constructivist poster territory. Four is the Suprematist minimum.

### Change the ground

```js
p.background(40, 10, 95);
```

Replace with any HSB triple. To make a cooler ground:
```js
p.background(200, 5, 95);   // cool off-white
```

### Change the frame

```js
p.strokeWeight(cs * 0.006);
p.rect(margin * 0.5, margin * 0.5, cs - margin, cs - margin);
```

- Change `0.006` for a thinner or thicker border.
- Change `margin = cs * 0.08` for more or less inset.

### Change surface tiling

Edit `surfaceDefs`:

```js
const surfaceDefs = [
  { id: 'surfacePrint',   wrapId: 'wrap-surfacePrint',   tile: { cols: 1, rows: 1 } },
  { id: 'surfaceScarf',   wrapId: 'wrap-surfaceScarf',   tile: { cols: 3, rows: 1 } },
  // ...
];
```

The `tile` object accepts any `{ cols, rows }` pair.

### Replace mockup images

Swap the files in `images/` or update the `src` attributes in the Commission section:

```html
<img src="images/your-mockup.png" alt="..." loading="lazy" />
```

---

## Design System

| Token | Value | Use |
|-------|-------|-----|
| `--bg` | `#0a0a0a` | Page background |
| `--paper` | `#f4f1ea` | Light surface (Surfaces section) |
| `--gold` | `#b8935a` | Primary accent |
| `--gold-soft` | `#d4b483` | Italic emphasis |
| `--serif` | Cormorant Garamond | Headings, titles |
| `--mono` | DM Mono | Labels, metadata, UI |

Type scale, spacing, and section rhythm match the Skyline, Portrait, Girih, Gothic Grid, Order to Chaos, Wallpaper Groups, Wandering Tiles, Food Grid, and Framed Line templates. Responsive breakpoints at 900px, 720px, 560px, and 400px.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `R` | Regenerate with a new seed |
| `S` | Download plate as PNG |
| `Enter` / `Space` | Activate focused archive item |
| `Esc` | Close legal modal |

---

## Browser Support

Modern evergreen browsers with:

- Canvas 2D (universal)
- `IntersectionObserver` (universal in modern browsers)
- `backdrop-filter` (Safari 9+, Chrome 76+, Firefox 103+)
- CSS custom properties (universal)

Fallback: if `IntersectionObserver` is unavailable, sketches render continuously.

---

## Accessibility

- Archive items are keyboard-focusable with `role="button"` and `aria-label`
- Legal modal traps focus, closes on `Esc` and overlay click
- `prefers-reduced-motion` disables transitions and smooth scroll
- All interactive elements have visible focus states (via browser default)

---

## Section Map

| Section | ID | Purpose |
|---------|-----|---------|
| Cover | — | Hero title + live composition |
| Statement | `#statement` | Artist statement |
| Plate | `#work` | Main work + metadata + controls |
| Surfaces | `#application` | Framed print + 4 tiled surfaces |
| Process | `#process` | Three process cards |
| Archive | `#archive` | 8 selectable seeds |
| Commission | `#commission` | Services + mockups + CTA |
| Colophon | — | Footer, links, legal |
| Legal modal | `#legalModal` | Licensing / Terms / Credits |

---

## Deployment (GitHub Pages)

1. Push the repo to GitHub.
2. Go to **Settings → Pages**.
3. Under **Source**, select `Deploy from a branch`.
4. Choose branch `main` and folder `/ (root)`.
5. Save — the site goes live at `https://reyrove.github.io/Deconstructed-Stage-Generative-Art/`.

---

## Notes on the Source Sketch

The generative engine is adapted almost verbatim from the original p5 sketch. The adaptations that turn it into a catalogue page:

1. **Seeded random replaces p5's `random()`** — a local `SeededRandom` class drives every choice. The original sketch calls `setSeeds()` for fxhash compatibility; that's replaced with the seeded generator directly.
2. **Sketch factory** — `makeSketch({ slotId, seedValue, ... })` returns a p5 sketch that can be instantiated into any DOM slot, enabling cover / plate / frame / surfaces / archive from one seed.
3. **DOM-driven canvas sizing** — `cs = min(windowHeight, windowWidth)` becomes slot-rect dimensions.
4. **Tiling support** — the `tile: { cols, rows }` parameter repeats the square composition across rectangular surfaces.
5. **Drawing logic takes `p` explicitly** — the original used p5's global mode; here `p` is passed so the factory closure works.
6. **`color(...)` replaced with `[h, s, b]` arrays** — the original sketch uses p5's `color()` to build the palette. Inside the factory, this is replaced with plain arrays passed to `p.fill()`. Same visual result, faster execution.
7. **`random([-1, 1])` replaced with `sr.pick([-1, 1])`** — the shear direction is drawn from the seeded generator.
8. **`windowResized` replaced by a debounced `handleResize`** — all sketches are destroyed and recreated on resize (250ms debounce), since their canvases are locked to slot dimensions.
9. **`triggerPreview` removed** — the original calls it for fxhash thumbnail generation; not needed in the plain-browser catalogue.

---

## Related Repositories

| Repo | Theme |
|------|-------|
| [`Skyline-Series`](https://github.com/reyrove/Skyline-Series) | Themed cityscapes |
| [`Portrait-Series-Generative-Art`](https://github.com/reyrove/Portrait-Series-Generative-Art) | Generative faces |
| [`Girih-2-Generative-Art`](https://github.com/reyrove/Girih-2-Generative-Art) | Islamic geometric ornament |
| [`Gothic-Grid-Generative-Art`](https://github.com/reyrove/Gothic-Grid-Generative-Art) | Mosaic grid composition |
| [`Order-to-Chaos-Generative-Art`](https://github.com/reyrove/Order-to-Chaos-Generative-Art) | Gradient grid composition |
| [`Wallpaper-Groups-Generative-Art`](https://github.com/reyrove/Wallpaper-Groups-Generative-Art) | Crystallographic symmetry |
| [`Wandering-Tiles-Generative-Art`](https://github.com/reyrove/Wandering-Tiles-Generative-Art) | Line-drawing composition |
| [`Food-Grid-Generative-Art`](https://github.com/reyrove/Food-Grid-Generative-Art) | Illustrated food grid |
| [`Framed-Line-Generative-Art`](https://github.com/reyrove/Framed-Line-Generative-Art) | Gesture line composition |
| **`Deconstructed-Stage-Generative-Art`** | Suprematist composition *(this repo)* |

All ten share the same catalogue template, design tokens, and seed-based reproducibility model.

---

## Credits

- **Design & Generative System** — Reyhaneh Daneshdoost
- **Typefaces** — Cormorant Garamond, DM Mono
- **Engine** — p5.js 1.9.0
- **Palette Reference** — Suprematism, Kazimir Malevich
- **Edition** — Deconstructed Stage, Autumn 2026

---

## License

All artwork, code, and generative systems are the intellectual property of the artist. Compositions may not be reproduced, redistributed, resold, or adapted without a written license.

For licensing, commissions, or private systems: **reyhanehdaneshdoost@gmail.com**