# Discover USDKit and what's new in OpenUSD
**WWDC26 · Session 285** · [Watch](https://developer.apple.com/videos/play/wwdc2026/285/)

_Platforms:_ macOS 27, visionOS 27, iOS 27

## Overview
This session introduces USDKit, a new first-party Swift framework that brings native USD support to Apple platform apps with built-in integration for RealityKit, Spatial Preview, and the spatial web. It also covers major OpenUSD standard advances — the Particle Field USD primitive type for Gaussian Splats, accessibility metadata standardization, and state-of-the-art mesh and texture compression — plus expanded 3D editing tools in Preview for Mac.

The OpenUSD section highlights Apple's active role in the Alliance for OpenUSD: co-developing the new Particle Fields primitive (with NVIDIA, Adobe, and Pixar), driving the first formal USD core specification, and contributing to MaterialX and OpenVDB updates across all Apple platforms. The Preview for Mac section showcases new 3D scene manipulation, property/lighting editing, and a choice of three renderers: RealityKit, Storm, and a new high-fidelity Raytracer, all supporting OpenPBR materials.

The USDKit API walkthrough demonstrates core USD concepts in Swift: opening/creating stages, traversing the prim hierarchy, adding referenced assets via composition, repositioning with transform operations, applying the new AccessibilityAPI schema, and exporting compressed USDZ packages. Integration paths are outlined for three audiences: USDKit for Apple-first Swift developers, SwiftUSD via SPM for open-source workflows, and the full OpenUSD C++ framework for cross-platform codebases.

## Key Topics

### OpenUSD Standard Advances
- **First formal USD core specification** released through Alliance for OpenUSD.
- **Particle Fields** primitive **[NEW]**: new USD type for representing Gaussian Splats and point clouds natively in USD; co-developed with NVIDIA, Adobe, and Pixar; composites alongside traditional 3D data.
- **AccessibilityAPI schema** **[NEW]**: standardized multi-apply API schema for adding accessible labels and descriptions to 3D objects; supported in Blender and Maya.
- **Mesh compression**: new algorithm (co-developed with Alliance for Open Media) achieves up to 90% size reduction.
- **Texture compression**: AVIF-based; combined with mesh compression yields ~7× smaller assets on average.
- MaterialX and OpenVDB updates across all Apple platforms.

### USDKit (New Framework)
- First-party Swift framework; ships with Apple platforms.
- Three integration paths:
  - **USDKit**: Apple SDK, fully Swift, ships with the OS.
  - **SwiftUSD** (via SPM): open-source Swift wrapper for existing USD C++ codebases.
  - **OpenUSD**: embeddable C++ framework for cross-platform apps.
- All share the same USD file format — full interoperability.

### 3D Editing in Preview for Mac
- Direct scene manipulation: translate, rotate, scale prims.
- Property and lighting editing panel.
- Hierarchy browser.
- Asset conversion between USD variants.
- Three renderers: **RealityKit** (default), **Storm** (OpenUSD renderer), **Raytracer** (new high-fidelity path tracer) — all support OpenPBR materials.
- Spatial Preview integration: one-click live link to Quick Look on paired Apple Vision Pro.

### Spatial Web — Safari Model Tag
- `<model>` HTML element now renders interactive USD/USDZ content on macOS and iOS.
- Spatial breakout on visionOS: tapping the model element enters a full immersive environment.
- USD assets are as native to web pages as images and video.

### USDKit Key Concepts and API
- Layers, Composition, Stages, Prims, Schemas, Attributes, Metadata — standard USD concepts wrapped in Swift.
- `USDStage()` for new in-memory stage; `USDStage.open(_:)` for file on disk.
- Traverse: `stage.descendants` (sequence of all prims).
- Define and reference: `stage.definePrim(at:type:)`, `prim.references.add(_:)`.
- Transform: `prim.addTransformOperation(type: .translate)`, `prim["xformOp:translate", as: USDValue.Vec3d.self] = [x, y, z]`.
- AccessibilityAPI: `prim.applyAPISchema("AccessibilityAPI", instanceName:)`, `prim.makeAttribute(named:as:)`, `prim[name, as: String.self] = value`.
- Export: `stage.exportPackage(to:options:)` with `.preferSmallTextureFiles(quality:)` and `.preferSmallMeshFiles`.

## APIs & Frameworks

### USDKit (NEW framework)
- `USDStage()` **[NEW]**: in-memory stage
- `USDStage.open(_ url: URL) throws` **[NEW]**: open from file
- `stage.descendants: Sequence<USDPrim>` **[NEW]**: traverse all prims
- `stage.definePrim(at: String, type: String) -> USDPrim` **[NEW]**
- `prim.references.add(_ path: String)` **[NEW]**: USD reference composition arc
- `prim.addTransformOperation(type: .translate)` **[NEW]**
- `prim[key, as: T.Type]` subscript **[NEW]**: attribute read/write
- `USDValue.Vec3d` **[NEW]**
- `prim.applyAPISchema(_ name: String, instanceName: String)` **[NEW]**
- `prim.makeAttribute(named: String, as: USDAttributeType)` **[NEW]**
- `stage.exportPackage(to: URL, options: [USDExportOption])` **[NEW]**
  - `.preferSmallTextureFiles(quality: .standard)` **[NEW]**
  - `.preferSmallMeshFiles` **[NEW]**
- `prim.name`, `prim.isValid`, `prim.isAnnotation` **[NEW]**
- `SdfPath` **[NEW]**: USD path type

### SpatialPreview (companion to USDKit)
- `USDPreviewSession(stage: USDStage)` **[NEW]**: live-preview a USDKit stage on visionOS
- See Session 282 for full API

### Safari / WebKit
- `<model src="..." environmentmap="...">` HTML element **[NEW]**: inline 3D model with spatial breakout on visionOS

### Preview for Mac (new 3D editing capabilities)
- Three renderers: RealityKit, Storm, new Raytracer **[NEW]**
- All support OpenPBR materials **[NEW]**

### Command Line Tools
- `usdcrush` **[NEW]**: command-line mesh and texture compression
  - `usdcrush model.usdz -o optimized.usdz`

## Code Highlights

Open a stage and traverse for a prim:
```swift
let stage = try USDStage.open(url)
for prim in stage.descendants {
    if prim.name == "scope" { /* found */ }
}
```

Add a reference and reposition:
```swift
let scope = stage.definePrim(at: "/World/scope", type: "Xform")
try scope.references.add("/ALab/assets/scope.usda")
scope.addTransformOperation(type: .translate)
scope["xformOp:translate", as: USDValue.Vec3d.self] = [2.5, 0.0, -1.0]
```

Apply accessibility metadata:
```swift
try scope.applyAPISchema("AccessibilityAPI", instanceName: "default")
scope["accessibility:default:label", as: String.self] = "Oscilloscope"
scope["accessibility:default:description", as: String.self] = "Vintage signal analyzer..."
```

Export with compression:
```swift
try stage.exportPackage(to: output, options: [
    .preferSmallTextureFiles(quality: .standard),
    .preferSmallMeshFiles
])
```

## Takeaways
- USDKit is the recommended path for Apple-platform Swift apps that need USD read/write/edit capabilities — it eliminates the need to embed the full C++ OpenUSD library.
- The new Particle Fields USD primitive standardizes Gaussian Splats representation in the broader USD ecosystem, enabling interoperability between DCC tools and Apple's rendering pipeline.
- Accessibility metadata in USD (`AccessibilityAPI` schema) is a standards-level win — 3D objects can now carry accessible descriptions that flow through Quick Look, Preview, and any renderer that respects the spec.
- Mesh + texture compression (~7× average) makes USD assets significantly more practical for web delivery and over-the-air distribution, with no quality loss for standard use cases.

---
_Source: WWDC26 Session 285 page (abstract, chapter summaries, code samples, and resource links)._
