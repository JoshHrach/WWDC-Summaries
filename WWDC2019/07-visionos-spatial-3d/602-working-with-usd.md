# Working with USD
**WWDC19 · Session 602** · [Watch](https://developer.apple.com/videos/play/wwdc2019/602/)

_Platforms:_ iOS 13, macOS Catalina 10.15, tvOS 13

## Overview
Universal Scene Description (USD) is a 3D file format and C++ library developed by Pixar for describing, composing, and exchanging 3D scenes at production scale. Apple adopted USD as the foundation for USDZ — the packaging format that powers AR Quick Look — and this session provides a comprehensive introduction to USD's file structure, scene graph model, mesh data, physically-based materials, subdivision surfaces, and composition engine. It also covers two major 2019 additions: the `usdzconvert` command-line tool for converting FBX, glTF, OBJ, and other formats to USDZ, and a new single-API SceneKit export path for writing USDZ programmatically.

## Key Topics

### What USD Is
- USD stands for Universal Scene Description; the library provides reading/writing, a powerful composition engine, and Python bindings.
- Three text/binary variants: `.usda` (human-readable ASCII), `.usdc` (efficient binary), `.usd` (either); all are interconvertible with `usdcat`.
- USDZ is an uncompressed ZIP archive with all files aligned to 64-byte boundaries for memory-mapped access; contains USD scene files plus PNG/JPEG textures; supports nested USDZ archives.

### File Structure and Scene Graph
- USD files are built from **prims** (nested containers) that hold **properties** and **metadata**; every prim is addressable by a path (e.g., `/simpleMesh/cube`).
- Scene graphs define object hierarchies; transforms on a parent propagate to children; materials and animations are separate prims outside the transform hierarchy.

### Mesh Data and Materials
- Mesh attributes (positions, normals, UV coordinates) carry USD interpolation metadata: `uniform` for per-face, `faceVarying` for per-face-per-vertex, etc.
- **UsdPreviewSurface** is the physically-based material subset for real-time rendering; supports metallic-roughness and specular-roughness workflows.
- Shader node graph: `UsdPreviewSurface` shader + `UsdUVTexture` (texture sampler) + `UsdPrimvarReader` (mesh attribute reader) + `UsdTransform2d` (UV transform) nodes connected by object-path wiring.

### Subdivision Surfaces
- USD unifies polygonal and subdivision surface description in a single `UsdGeomMesh` prim; subdivision-specific properties include crease sharpness and corner weights.
- Apple provides Metal shaders built on Pixar's **OpenSubdiv** for GPU evaluation; OpenSubdiv is the basis for subdivision surfaces in SceneKit.

### Composition Engine (References and Overrides)
- USD's composition engine allows virtual overlay instances inside the scene graph, each with a unique path, so per-instance property overrides (e.g., color) are stored as lightweight diffs rather than duplicated geometry.
- Live references: a layout file references asset files authored by other artists; when the asset is updated, the layout automatically picks up the change without modification.

### Workflows: Converting Existing Assets
- `usdzconvert` **[NEW]** — new Python-based CLI tool superseding the Xcode converter; converts OBJ, glTF, FBX, and more to USDZ; performs asset validation after conversion.
- Companion CLI tools from Pixar's USD library (precompiled for macOS): `usdcat` (text dump), `usdtree` (hierarchy view), `usdchecker` (validation), `fixOpacity` (repairs transparent-material issues from iOS 12).
- Material properties (diffuse color, roughness, textures) can be specified as command-line arguments to `usdzconvert`, enabling rich material assignment for simple formats like OBJ.

### Creating USDZ from SceneKit **[NEW]**
- Load or create a `SCNScene` as normal, then call `scene.write(to: url, options: nil, delegate: nil, progressHandler: nil)` with a `.usdz` file extension — a single API exports the entire scene to USDZ.
- Xcode's SceneKit editor also gains a direct USDZ export menu item.

## APIs & Frameworks

### SceneKit
- `SCNScene.write(to:options:delegate:progressHandler:)` **[NEW]** — export scene to USDZ when the destination URL has a `.usdz` extension

### Command-Line Tools (USD Python package)
- `usdzconvert` **[NEW]** — convert OBJ / glTF / FBX → USDZ with material arguments and built-in validation
- `usdcat` — print plain-text representation of any USD/USDZ file
- `usdtree` — print prim hierarchy tree of a USD/USDZ file
- `usdchecker` — validate a USDZ asset against Apple's AR Quick Look requirements
- `fixOpacity` — fix transparent-material opacity regression from iOS 12 → iOS 13

### USD Library Concepts (not Obj-C/Swift APIs, but key to understanding USDZ authoring)
- `UsdGeomMesh` — polygon / subdivision surface prim
- `UsdPreviewSurface` — PBR shader node (diffuseColor, metallic, roughness, normal, occlusion, emissiveColor)
- `UsdUVTexture` — texture sampler shader node
- `UsdPrimvarReader` — mesh attribute reader shader node
- `UsdTransform2d` — UV transform shader node

## Code Highlights

Exporting a SceneKit scene to USDZ in one call:
```swift
let scene = SCNScene(named: "MyModel.scn")!
let outputURL = FileManager.default.temporaryDirectory
    .appendingPathComponent("export.usdz")
scene.write(to: outputURL, options: nil, delegate: nil, progressHandler: nil)
```

Converting a glTF file to USDZ from the terminal:
```bash
usdzconvert myModel.gltf myModel.usdz
```

Assigning a constant metallic-roughness material when converting an OBJ:
```bash
usdzconvert tetrahedron.obj tetrahedron.usdz \
    -diffuseColor 0.8 0.1 0.1 \
    -metallic 0.9 \
    -roughness 0.2
```

Inspecting a USDZ archive's contents (it's a ZIP):
```bash
zipinfo myModel.usdz
usdtree myModel.usdz
usdcat myModel.usdz
```

## Takeaways
- USDZ is an uncompressed ZIP of USD scene files plus textures; understanding its structure (prims, properties, paths) makes authoring and debugging straightforward using the provided CLI tools.
- `usdzconvert` replaces the Xcode converter and adds FBX/glTF support with material flags — integrate it into existing DCC pipelines to automate USDZ production.
- SceneKit gains a one-call USDZ export API (`write(to:)` with a `.usdz` URL) enabling runtime USDZ generation from procedural or user-created scenes.
- USD's composition engine (references + overrides) enables collaborative asset workflows where multiple artists work on the same scene without clobbering each other's data.

---
_Source: WWDC19 Session 602 page (abstract, transcript, and resource links)._
