# Book Textures Explained

This folder contains 3 texture files used by the `scene.gltf` model to give the book a realistic, physically-based appearance. All 3 images are **2048×2048 RGB** and are sampled simultaneously by the GPU every frame.

---

## The 3 Textures

| File | Role | What it does |
|------|------|------|
| `book_baseColor.png` | Albedo / Color | RGB = surface color. Tells the renderer what color the book is at each pixel. |
| `book_metallicRoughness.png` | PBR properties | R channel = **metallic** (0=matte, 1=metal). G channel = **roughness** (0=smooth/shiny, 1=rough). Both packed into one image. |
| `book_normal.png` | Surface detail | RGB encodes a surface normal vector at each pixel, creating fake bump/detail without extra geometry. |

---

## How they connect to the model

The glTF file (`scene.gltf`) is the blueprint that ties everything together. It defines 1 material (`book`) using the PBR (Physically Based Rendering) workflow:

```
Material "book"
├── baseColorTexture          → index 0 → book_baseColor.png
├── metallicRoughnessTexture  → index 1 → book_metallicRoughness.png
└── normalTexture             → index 2 → book_normal.png
```

Three.js's `GLTFLoader` reads the glTF and automatically loads all 3 external images via the `uri` paths. No manual texture loading code is needed — it's all handled by the loader from the glTF definition.

## Sampler settings

All 3 textures share the same sampler:
- **magFilter:** LINEAR
- **minFilter:** LINEAR_MIPMAP_LINEAR
- **wrapS / wrapT:** REPEAT

This means textures tile seamlessly if UV coordinates go beyond the 0–1 range.

## How PBR shading works

Three.js's PBR shader combines all 3 textures when rendering each pixel:

1. **baseColor** provides the base color
2. **metallicRoughness** tells the shader which parts are shiny vs rough (e.g. a glossy cover vs a matte spine)
3. **normal** adds fake surface bump so light catches realistically on edges and details

The result is a book that looks real — not flat — because the renderer simulates how light actually interacts with surfaces in the real world.
