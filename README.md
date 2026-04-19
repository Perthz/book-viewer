# 📖 3D Book Viewer

Interactive 3D book model viewer built with Three.js.

**Live:** [https://perthz.github.io/book-viewer](https://perthz.github.io/book-viewer)

## Features

- [x] 3D glTF book model with orbit controls (drag / scroll / right-drag)
- [x] Warm cozy room lighting — Biblio-style hemisphere + key/fill/rim rig
- [x] Dimension sliders: width (5–50 cm), height (5–40 cm), thickness (0.3–10 cm)
- [x] Cover image import — apply your own cover art
- [x] Shadow casting with PCFSoftShadowMap
- [x] ACES filmic tone mapping + depth fog

## Setup

```bash
# Serve locally (any static server works)
cd book-viewer
python3 -m http.server 7890
# → Open http://localhost:7890
```

Or use VS Code's Live Server, Caddy, nginx, etc.

## Project Structure

```
book-viewer/
├── index.html     ← Single-file app (Three.js via CDN importmap)
├── scene.gltf     ← glTF book model
├── scene.bin      ← Model geometry buffer
└── license.txt    ← CC-BY-4.0 (Eric Fan / Sketchfab)
```

## TODO

- [ ] Cover image actually applies texture to the front face (mesh identification by AABB Z-max is wired but texture not visibly applied — likely UV mapping issue in source model)
- [ ] Shadow acne / breeding persists (bias tuning in progress)
- [ ] Procedurally-built hardcover geometry option (like Biblio's HardcoverBookModel)
- [ ] Title / author text rendering on cover
- [ ] Spine texture support
- [ ] Export as PNG / screenshot button

## Tech Stack

- **Three.js r160** — 3D rendering
- **OrbitControls** — camera interaction
- **GLTFLoader** — model loading
- **PCFSoftShadowMap** — soft shadows

## Credit

Book model by [Eric Fan](https://sketchfab.com/ericfan1992) on Sketchfab — [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/)
