# Book Viewer — Design Language

## Overview

Two pages share a unified dark-panel aesthetic: **warm cream app background + frosted-glass control panels** that feel like floating UI overlays. The contrast between the organic paper background and the sleek dark panels creates a "physical object in a digital studio" mood.

| Page | URL | 3D Object |
|------|-----|-----------|
| `static/` | `book-viewer/static/index.html` | Hardcover book (glTF, PBR materials) |
| `note/` | `book-viewer/note/index.html` | Notebook (procedural geometry, flat material) |

---

## Color Palette

### App Background
| Token | Hex | Use |
|-------|-----|-----|
| `--bg` | `#f4f1ec` | App body background (warm cream / paper) |
| `--hint` | `#bbb` | Drag hint text color |

### Panel Surface (shared across all panels)
| Token | Hex | Use |
|-------|-----|-----|
| `--panel-bg` | `rgba(20, 20, 26, 0.88)` | All control panel backgrounds |
| `--panel-border` | `rgba(255,255,255,0.08)` | Panel border (all panels) |
| `--panel-radius` | `12px` | Panel corner radius |
| `--panel-blur` | `blur(12px)` | `backdrop-filter` on panels |

### Text
| Token | Hex | Use |
|-------|-----|-----|
| `--text-primary` | `#e0e0e0` | Panel body text |
| `--text-secondary` | `#aaa` | Labels, secondary info |
| `--text-tertiary` | `#666` | Section headers, uppercase labels |
| `--text-disabled` | `#555` | Disabled / unit labels |
| `--text-value` | `#fff` | Slider values, live numbers |

### Accent Colors (semantic — each feature owns one)
| Token | Hex | Feature |
|-------|-----|---------|
| `--accent-blue` | `#6c8fff` | Dimension sliders, reset button, note page thumb |
| `--accent-orange` | `#ffb060` | Open Book button |
| `--accent-green` | `#50dc82` | Open Book button (active/open state) |
| `--accent-purple` | `#c864ff` | Cover Designer feature |
| `--accent-red` | `#4a2020` | Delete button background |

### Glow Colors (slider thumbs)
| Token | RGBA |
|-------|------|
| Blue glow | `rgba(108,143,255,0.5)` |
| Purple glow | `rgba(200,100,255,0.4)` |

---

## Typography

- **Font stack:** `system-ui, -apple-system, BlinkMacSystemFont, sans-serif` everywhere
- **Panel body:** `0.8rem`
- **Section headers (h3, `.cp-tab`):** `0.7rem`, `text-transform: uppercase`, `letter-spacing: 0.1em`, `font-weight: 500`, color `--text-tertiary`
- **Input / button text:** `0.74–0.78rem`
- **Value labels (slider numbers):** `font-weight: 600`, color `--text-value`
- **Unit labels:** `0.7rem`, color `--text-disabled`

---

## Spatial System

| Token | Value |
|-------|-------|
| Base grid | `4px` |
| Panel padding | `18px 20px` (outer panels), `22px` (Cover Designer modal) |
| Inner spacing | `12–14px` between sections, `8–10px` between rows |
| Panel width | `220px` (side panels), `540px` (Cover Designer modal) |
| Panel radius | `12px` (outer panels), `14px` (modal), `6px` (controls/buttons) |
| Range thumb | `14px × 14px` circle |

---

## Components

### 1. Control Panel (Side Drawer)

Frosted dark glass, fixed top/right, `width: 220px`. Contains dimension sliders, Open Book, rotation, and Cover Designer.

```
#controls
  h3 — "BOOK DIMENSIONS" uppercase header
  .dim-row — label + range slider
  hr.divider — rgba(255,255,255,0.06) separator
  #openBtn — orange → green when book is open
  .anim-row — open amount slider (visible when book is open)
  .rot-row — Flip X/Y/Z sliders
  hr.divider
  #resetBtn — blue ghost button
  .cp-saves — position save/load/delete
  #coverDesignerBtn — purple ghost button
```

### 2. Range Slider

```
input[type="range"]
  track: height:3px, bg: rgba(255,255,255,0.1), border-radius: 2px
  thumb: 14px circle, solid accent color, box-shadow: glow
  :focus — no visible outline (thumb glow is enough)
```

**Thumb colors:**
- Dimension / Flip / Animation sliders → `--accent-blue`
- Cover Designer sliders (font size, etc.) → `--accent-purple`

### 3. Open Book Button

```
#openBtn
  default:   bg rgba(255,160,80,0.18), border rgba(255,160,80,0.35), color #ffb060
  :hover:    bg rgba(255,160,80,0.30)
  .open:     bg rgba(80,220,130,0.15), border rgba(80,220,130,0.35), color #50dc82
```

### 4. Position Save/Load

```
.cp-saves
  select — flex:1, bg #1a1a22, border #333, color #ccc, radius 6px
  Save button — .btn .anim-btn (blue ghost)
  Delete button — bg #4a2020, border none
```

### 5. Cover Designer Modal

Frosted dark glass, centered overlay, `width: 540px`, `border: 1px solid rgba(200,100,255,0.25)`, purple accent.

```
#coverPanel
  h2 — "Cover Designer", color --accent-purple
  .cp-tabs — tab bar with bottom-border active indicator
  .cp-page — content area per tab
  #cpPreviewWrap — canvas wrapper
    #cpPreview — 540×540 canvas, border rgba(255,255,255,0.1), radius 8px
    #cpPreviewLabel — "UV LAYOUT · BACK | SPINE | FRONT", 0.65rem monospace
  .cp-row — label + input/textarea/color picker
  .cp-actions
    .cp-apply — primary: purple bg/border
    .cp-secondary — ghost: rgba(255,255,255,0.06) bg
```

### 6. Color Picker Row

```
.cp-color-wrap
  input[type="color"] — 36×32px, border none, bg none, radius 4px, padding 0
  span — hex value in monospace, color --text-secondary
```

### 7. Ghost Buttons (shared pattern)

```
<button> (no class)
  bg: rgba(255,255,255,0.06), border: rgba(255,255,255,0.1), color: #aaa
  :hover: bg rgba(255,255,255,0.1), color: #ddd
  radius: 6px, padding: 7–9px, font-size: 0.74–0.78rem
```

---

## CSS Rules (copy-paste patterns)

### Frosted panel
```css
background: rgba(20, 20, 26, 0.88);
border: 1px solid rgba(255,255,255,0.08);
border-radius: 12px;
backdrop-filter: blur(12px);
```

### Range slider base
```css
input[type="range"] {
  -webkit-appearance: none;
  width: 100%;
  height: 3px;
  background: rgba(255,255,255,0.1);
  border-radius: 2px;
  outline: none;
}
input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 14px;
  height: 14px;
  background: #6c8fff;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 0 8px rgba(108,143,255,0.5);
}
```

### Ghost button
```css
background: rgba(255,255,255,0.06);
border: 1px solid rgba(255,255,255,0.1);
border-radius: 6px;
color: #aaa;
cursor: pointer;
```
---

## Interaction Behaviors

| Element | Hover | Active / Focus |
|---------|-------|----------------|
| Range slider | (none — cursor is already pointer on thumb) | thumb glow intensifies |
| Ghost button | `bg rgba(255,255,255,0.1)`, color `#ddd` | slight scale press |
| `#openBtn` | bg opacity +0.12 | text color shifts to green when open |
| `.cp-tab` | color `#ccc` | color `--accent-purple`, bottom border |
| Form inputs | — | `border-color: rgba(200,100,255,0.5)` |
| Color swatch | `cursor: pointer` | — |
| Position select | border-color `#555` | — |

---

## States

### Loading
```
#loading — fixed fullscreen, bg --bg (#f4f1ec), centered "Loading model…" text
```

### Drag Hint
```
#hint — fixed bottom-center, color --hint (#bbb), 0.75rem
  "Drag to rotate · Scroll to zoom · Right-drag to pan"
```

### Open Book Slider (appears when book is open)
```
.anim-row
  .anim-label — "Open Amount" left, "X%" right (value in --text-value)
  input[type="range"] — blue thumb
```

---

## 3D Scene Defaults (static/index.html)

| Property | Default Value |
|----------|--------------|
| Camera position | `(0, 2, 8)` |
| Camera target | `(0, 0, 0)` |
| Book dimensions | W: 14cm, H: 21cm, T: 2cm |
| Open amount | 0% |
| Flip X | 0° |
| Flip Y | 0° |
| Flip Z | 0° |
| Paper color | `#f5f0e8` |
| Cover color | `#8b2020` (deep red) |
| Position (default) | `front` (user saved, set as default) |

### OrbitControls
- Left drag → orbit
- Scroll → zoom
- Right drag → pan
- Damping factor: `0.05`

---

## Cover Designer Canvas

- Canvas size: `540 × 540px`
- Shows: scaled representation of the full `2048 × 2048` texture
- Label: `"UV LAYOUT · BACK | SPINE | FRONT"` — this label describes the texture zones conceptually; the actual texture is a single painted artwork covering the full UV region `u=[0.0023, 0.6839], v=[0.1083, 1.0]` mapped to pixel `x=[4,1400], y=[0,1826]`
- Image rendering: `image-rendering: auto` (smooth scaling)

---

## Naming Conventions (CSS classes)

| Pattern | Meaning |
|---------|---------|
| `.dim-*` | Dimension controls (width/height/thickness) |
| `.anim-*` | Open-book animation controls |
| `.rot-*` | Flip/rotation controls |
| `.cp-*` | Cover Designer panel (`.cp-` prefix) |
| `.btn` | Base button class |
| `.anim-btn` | Animation/secondary button (blue ghost) |
| `hr.divider` | Panel section separator |

---

## File Structure

```
book-viewer/
├── design.md              ← This file
├── static/
│   ├── index.html         ← Hardcover book viewer
│   ├── scene.gltf         ← Book 3D model
│   ├── scene.bin          ← Binary geometry
│   └── textures/
│       ├── book_baseColor.png       ← PBR color texture (2048×2048)
│       ├── book_normal.png           ← Normal map (2048×2048)
│       ├── book_metallicRoughness.png
│       ├── book_uv_region.png       ← Cropped UV region (full capture)
│       ├── book_uv_region_small.png ← Cropped UV region (preview)
│       └── README.md                 ← Texture explanation
└── note/
    └── index.html         ← Notebook viewer (procedural geometry)
```
