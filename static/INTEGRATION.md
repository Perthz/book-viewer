# Integrating the GLTF Hardcover Model into Biblio

This guide explains how to replace Biblio's procedural `buildHardcover()` geometry
with the rigged GLTF model used in this static viewer, including the canvas texture
pipeline that paints the cover, spine, back, and inner liner.

---

## What you are replacing

Biblio builds hardcover books from plain `BoxGeometry` meshes with solid
`MeshStandardMaterial` colours. The result looks good but cannot carry a real cover
image, title text, or any per-face design.

The static viewer uses a rigged GLTF model (`scene.gltf`) that:
- Has correct curvature, hinge notch, and page-block geometry baked in.
- Is driven by a skeleton so the cover opens/closes with an animation clip.
- Samples a single 2048 × 2048 canvas texture painted at runtime, split into
  per-face regions (front, spine, back, inner liner, page edges, open spreads).

---

## Step 1 — Copy the model assets

Copy the following files from `static/` into your Biblio project folder
(or a subfolder, e.g. `biblio/model/`):

```
scene.gltf
scene.bin
textures/book_baseColor.png
textures/book_metallicRoughness.png
textures/book_normal.png
```

`scene.gltf` references `scene.bin` and the texture paths with relative URIs.
If you move them into a subfolder, open `scene.gltf` in a text editor and update
the `uri` fields under `"buffers"` and `"images"` to match.

---

## Step 2 — Add the GLTFLoader import

Biblio already uses Three.js 0.160.0 via importmap. Add `GLTFLoader` to the
import block at the top of `biblio/index.html`:

```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
  }
}
</script>

<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { GLTFLoader }    from 'three/addons/loaders/GLTFLoader.js';
// ...
</script>
```

---

## Step 3 — Load the model and replace buildHardcover

Replace the call to `buildHardcover()` with a GLTF load. The loader is
asynchronous, so all post-load logic moves into the callback:

```js
// Shared canvas — one per loaded book instance
const TEX_SIZE = 2048;

function createBookCanvas() {
  const canvas = document.createElement('canvas');
  canvas.width = TEX_SIZE;
  canvas.height = TEX_SIZE;
  return canvas;
}

let bookGroup = null;   // THREE.Group returned by GLTF
let bookMesh  = null;   // the SkinnedMesh (cover surface)
let bookMat   = null;   // MeshStandardMaterial on the cover mesh
const texCanvas = createBookCanvas();
const texCtx    = texCanvas.getContext('2d');

const loader = new GLTFLoader();
loader.load('model/scene.gltf', (gltf) => {
  bookGroup = gltf.scene;

  // The cover is the SkinnedMesh with the most vertices.
  bookGroup.traverse(obj => {
    if (obj.isSkinnedMesh && (!bookMesh || obj.geometry.attributes.position.count > bookMesh.geometry.attributes.position.count)) {
      bookMesh = obj;
      bookMat  = obj.material;
    }
  });

  // Default scale: match Biblio's working dimensions (16 cm wide, 24 cm tall, 3 cm thick)
  applySize(16, 24, 3);

  scene.add(bookGroup);

  // Paint an initial cover so the book isn't grey on first render
  paintCover({ color: '#9BB5CE', spineColor: '#6B8CAE', title: 'Book Title', author: 'Author Name' });
});
```

---

## Step 4 — Sizing via skeleton scale

The GLTF model was authored at a fixed physical size. `applySize` scales the
`bookGroup` uniformly so the rendered book matches real-world cm dimensions:

```js
const MODEL_REF_H = 21;   // cm — the height the model was designed for
const MODEL_REF_W = 14;   // cm
const MODEL_REF_T = 2;    // cm

function applySize(widthCm, heightCm, thicknessCm) {
  if (!bookGroup) return;
  const sy = heightCm  / MODEL_REF_H;
  const sx = widthCm   / MODEL_REF_W;
  const sz = thicknessCm / MODEL_REF_T;
  bookGroup.scale.set(sx, sy, sz);
}
```

Call `applySize` whenever width, height, or thickness changes (e.g. from UI
sliders or when a new book is selected from the catalogue).

---

## Step 5 — Canvas texture regions (UV atlas)

The 2048 × 2048 texture is divided into fixed pixel rectangles, one per face.
These coordinates were hand-verified against the model's UV layout:

```js
const REGIONS = {
  //              x0    y0    x1    y1
  back:         { x0:   0, y0: 1224, x1:  611, y1: 2048 },  // back cover face
  spine:        { x0: 611, y0: 1224, x1:  781, y1: 2048 },  // spine
  front:        { x0: 781, y0: 1224, x1: 1403, y1: 2048 },  // front cover face
  innerCover:   { x0:   0, y0:  404, x1: 1403, y1: 1224 },  // inside liner (visible when open)
  paperEdges:   { x0:   0, y0:  150, x1: 1990, y1:  417 },  // stacked page edges
  contentUpper: { x0:1403, y0:  541, x1: 2048, y1: 1291 },  // upper half of open spread
  contentLower: { x0:1403, y0: 1291, x1: 2048, y1: 2048 },  // lower half of open spread
};
```

> **UV orientation note:** The GPU mirrors and rotates some regions when sampling.
> Paint each region with these compensations applied before drawing text/images:
>
> | Region | flipX | flipY |
> |--------|-------|-------|
> | front | false | true |
> | back | false | true |
> | spine | true | false |
> | innerCover | false | false |
> | paperEdges | false | false |
> | contentUpper | false | true |
> | contentLower | false | true |
>
> Apply the flip by translating to the region centre, calling
> `ctx.scale(flipX ? -1 : 1, flipY ? -1 : 1)`, then translating back before
> drawing. See `paintRegion()` in `static/index.html` for the reference
> implementation (~line 1051).

---

## Step 6 — Minimal paintCover implementation

A minimal cover painter that fills the visible faces with flat colour and draws
the title on front and spine:

```js
function paintCover({ color, spineColor, title, author }) {
  const R = REGIONS;

  // Fill background
  texCtx.clearRect(0, 0, TEX_SIZE, TEX_SIZE);

  const fill = (r, c) => {
    texCtx.fillStyle = c;
    texCtx.fillRect(r.x0, r.y0, r.x1 - r.x0, r.y1 - r.y0);
  };
  fill(R.front,      color);
  fill(R.back,       color);
  fill(R.spine,      spineColor || color);
  fill(R.innerCover, color);
  fill(R.paperEdges, '#F5EEDD');

  // Front title (flipY compensated — draw normally, GPU will flip Y)
  const fr = R.front;
  const fw = fr.x1 - fr.x0, fh = fr.y1 - fr.y0;
  texCtx.save();
  texCtx.translate(fr.x0 + fw / 2, fr.y0 + fh / 2);
  texCtx.scale(1, -1);                       // flipY compensation
  texCtx.fillStyle = '#fff';
  texCtx.font = `bold ${fw * 0.1}px Georgia, serif`;
  texCtx.textAlign = 'center';
  texCtx.textBaseline = 'middle';
  texCtx.fillText(title, 0, -fh * 0.15);
  if (author) {
    texCtx.font = `italic ${fw * 0.055}px Georgia, serif`;
    texCtx.fillText(author, 0, fh * 0.05);
  }
  texCtx.restore();

  // Spine title (flipX compensated, rotated -90° so text reads top-to-bottom)
  const sr = R.spine;
  const sw = sr.x1 - sr.x0, sh = sr.y1 - sr.y0;
  texCtx.save();
  texCtx.translate(sr.x0 + sw / 2, sr.y0 + sh / 2);
  texCtx.scale(-1, 1);                       // flipX compensation
  texCtx.rotate(-Math.PI / 2);
  texCtx.fillStyle = '#fff';
  texCtx.font = `bold ${sw * 0.55}px Georgia, serif`;
  texCtx.textAlign = 'center';
  texCtx.textBaseline = 'middle';
  texCtx.fillText(title, 0, 0);
  texCtx.restore();

  applyCanvasTexture();
}

function applyCanvasTexture() {
  if (!bookMat) return;
  if (!bookMat.map || !(bookMat.map instanceof THREE.CanvasTexture)) {
    bookMat.map = new THREE.CanvasTexture(texCanvas);
    bookMat.map.colorSpace = THREE.SRGBColorSpace;
    bookMat.map.flipY = false;        // glTF default — do not change
    bookMat.map.wrapS = THREE.RepeatWrapping;
    bookMat.map.wrapT = THREE.RepeatWrapping;
    bookMat.needsUpdate = true;
  }
  bookMat.map.needsUpdate = true;
}
```

---

## Step 7 — Reusing the full Cover Designer (optional)

If you want the complete Cover Designer UI (front-cover image, blurb, barcode,
palette auto-detect, EPUB import, etc.) rather than writing a new painter:

1. Copy the entire `static/index.html` `<style>` block for `.cp-*` classes into
   Biblio's stylesheet.
2. Copy the `<div id="coverPanel">` HTML block into Biblio's body.
3. Copy the JS sections marked with `// ── Cover Designer` comments into your
   module, in order: `REGIONS`, `paintRegion`, painter functions
   (`paintFront`, `paintSpine`, `paintBack`, `paintInner`, `paintPaperEdges`,
   `paintContent`), `rebuildTexture`, `applyTextureToBook`, event wiring.
4. Bind the `applySize` function from Step 4 to Biblio's existing
   width/height/thickness controls.

The Cover Designer has no external dependencies beyond Three.js and JSZip
(for EPUB import). JSZip is loaded from CDN:

```html
<script src="https://cdn.jsdelivr.net/npm/jszip@3.10.1/dist/jszip.min.js"></script>
```

---

## Quick-start checklist

- [ ] `scene.gltf`, `scene.bin`, `textures/` copied to Biblio project
- [ ] `GLTFLoader` imported
- [ ] `loader.load()` replaces `buildHardcover()` call
- [ ] `applySize(w, h, t)` wired to dimension controls
- [ ] `paintCover()` / `applyCanvasTexture()` called after load and on design change
- [ ] Existing `buildSoftcover()` left untouched (softcovers keep procedural geometry)

---

## Reference files

| File | Purpose |
|------|---------|
| `static/index.html` | Complete working implementation — single source of truth |
| `static/scene.gltf` | Model definition (references `.bin` and texture URIs) |
| `static/scene.bin` | Geometry + skeleton binary data |
| `static/textures/book_baseColor.png` | Default albedo (overridden by canvas texture at runtime) |
| `static/textures/book_metallicRoughness.png` | PBR roughness/metalness (kept as-is) |
| `static/textures/book_normal.png` | Surface normal map for lighting detail (kept as-is) |
