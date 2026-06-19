# Managing 3D Assets with Model I/O
**WWDC15 · Session 602** · [Watch](https://developer.apple.com/videos/play/wwdc2015/602/)

_Platforms:_ iOS 9, OS X El Capitan 10.11

## Overview
This session introduces Model I/O, a brand-new framework in iOS 9 and OS X El Capitan for importing, processing, and exporting 3D assets. Model I/O provides a unified set of physically based data structures for meshes, materials, lights, cameras, environments, and voxels — filling the gap between content creation tools (Maya, Modo, etc.) and Apple's rendering frameworks (SceneKit, Metal, OpenGL via GLKit/MetalKit).

The framework handles the traditionally tedious work of parsing common 3D file formats, generating hardware-ready buffers, and performing offline baking operations (ambient occlusion, light maps) that produce high-quality visuals at minimal runtime cost. It is integrated into Xcode 7's SceneKit editor, Finder/Quick Look preview, Swift Playgrounds, and the GameKit APIs.

Presenters cover geometry features (normal smoothing, subdivision surfaces), volumetric representations (voxels and CSG Boolean operations), physically based materials and IES photometric lights, physically accurate camera models, and advanced global illumination baking.

## Key Topics

### File Import/Export
- Supported import formats: Alembic (`.abc`), Polygon (`.ply`), Wavefront (`.obj`), STL (triangles/CAD), USD
- Export: OBJ, STL, and others
- Single-call import: `MDLAsset(url:)` / `exportAssetToURL(_:)`

### Physical Realism — Lights
- **IES photometric profiles**: Load a manufacturer's real-world light measurement file to get physically accurate irradiance distribution in shaders.
- **Color temperature**: Specify lights in Kelvin (e.g., 4000K) instead of RGB.
- **Image-based lighting (IBL)**: Derive irradiance from panoramic photographs (iPhone HDR panorama supported).
- **Spherical harmonics**: Compact 27-float representation of an irradiance environment map.
- Light subclasses: `MDLAreaLight` (procedural), `MDLPhotometricLight` (IES), light probes with cube maps.

### Physical Realism — Materials
- Physically based BRDF with 10 artist-friendly parameters (metalness, roughness, etc.)
- Dielectric (clay-like) vs. metallic materials via a single parameter
- Clear-coat and satin finish achievable through parameter combinations
- Supports Lambert/Blinn-Phong for legacy compatibility

### Physical Realism — Cameras
- Full lens-to-sensor model: field of view, barrel distortion, chromatic aberration, aperture (f-stop), lens barrel length, sensor size, and exposure characteristics
- Utility functions agree with textbook lens equations (e.g., 50mm f/1.8 bokeh calculations)

### Procedural Skies
- Physics-based procedural sky generator (time of day, atmospheric conditions, back scatter)
- Photography-based: load iPhone/DSLR spherical panorama → cube map for reflection and irradiance

### Geometry Features
- **Normal smoothing** (`addNormalsWithAttributeNamed(_:creaseThreshold:)`) — interpolated vertex normals for smooth shading with crease threshold control
- **Subdivision surfaces** (`newSubdividedMesh(_:submeshIndex:subdivisionLevels:)`) — LOD generation from a low-poly control cage
- **Voxels** (`MDLVoxelArray`) — sparse volumetric grid with shell-level values; supports union, intersection, difference Boolean CSG operations; convert back to polygon mesh

### Framework Integration
- **SceneKit**: 1:1 mapping — `MDLAsset` ↔ SCNNode (root), `MDLMesh` ↔ SCNGeometry, `MDLLight` ↔ SCNLight, `MDLCamera` ↔ SCNCamera, `MDLMaterial` ↔ SCNMaterial
- **MetalKit**: `MDLMesh` → `MTKMesh` array with Metal-ready vertex buffers; developer writes own shaders
- **GLKit**: Similar Metal-buffer preparation path
- **Xcode 7 SceneKit Editor**: Drag-and-drop ambient occlusion bake button, vertex or texture output
- **Finder/Quick Look**: Preview `.abc` and `.obj` files by pressing Space Bar
- **Swift Playgrounds**: Model I/O works directly in Playgrounds

### Baking — Ambient Occlusion
- Offline ray-traced ambient occlusion — measures how much sky/environment light reaches each surface point
- Store as per-vertex floats (cheap, works well for high-poly meshes) or UV-unwrapped texture (needed for low-poly)
- Built-in UV mapper for texture baking: `generateAmbientOcclusionTexture(withQuality:attenuationFactor:objectsToConsider:vertexAttributeNamed:materialPropertyNamed:)`
- Quality parameter controls ray count; attenuation controls shadow strength

### Baking — Light Maps
- Precompute diffuse lighting from many lights into a single texture (one texture fetch at runtime, regardless of light count)
- Supports area lights, IES photometric lights, spotlights with accurate soft shadows
- Eliminates per-frame lighting computation cost

## APIs & Frameworks

- `ModelIO` framework **[NEW]**
- `MDLAsset` **[NEW]** — top-level container; `init(url:)`, `exportAssetToURL(_:)`
- `MDLObject` **[NEW]** — base scene graph node with transform component and custom components
- `MDLTransformComponent` **[NEW]**
- `MDLComponent` **[NEW]** — protocol for custom components (e.g., trigger volumes)
- `MDLMesh` **[NEW]**
  - `addNormalsWithAttributeNamed(_:creaseThreshold:)` **[NEW]**
  - `newSubdividedMesh(_:submeshIndex:subdivisionLevels:)` **[NEW]**
  - `generateAmbientOcclusionVertexColors(withQuality:attenuationFactor:objectsToConsider:vertexAttributeNamed:)` **[NEW]**
  - `generateAmbientOcclusionTexture(withQuality:attenuationFactor:objectsToConsider:vertexAttributeNamed:materialPropertyNamed:)` **[NEW]**
  - Mesh generators: `newBox(withDimensions:segments:geometryType:inwardNormals:allocator:)`, `newSphere(...)`, etc. **[NEW]**
- `MDLSubmesh` **[NEW]** — index buffer + material assignment within a mesh
- `MDLVertexDescriptor` **[NEW]** — describes vertex buffer layout (attribute, format, stride)
- `MDLMeshBuffer` / `MDLMeshBufferAllocator` **[NEW]** — custom memory management
- `MDLMaterial` **[NEW]** — physically based BRDF; `scatteringFunction`, singly-inherited base materials
- `MDLMaterialProperty` **[NEW]** — name, semantic, type, value
- `MDLLight` / `MDLAreaLight` / `MDLPhotometricLight` **[NEW]**
  - `lightType`, `color`, `colorTemperature` (Kelvin)
  - `IESProfile` loading **[NEW]**
- `MDLCamera` **[NEW]** — full lens/sensor model; `fieldOfView`, `focalLength`, `fStop`, `apertureBladeCount`, `sensorVerticalAperture`, `exposure`
- `MDLTexture` **[NEW]** — `init(named:)`, `init(url:)`, cube map creation
- `MDLSkyCubeTexture` / `MDLCheckerboardTexture` **[NEW]** — procedural environment textures
- `MDLVoxelArray` **[NEW]**
  - `init(asset:divisions:interiorShells:exteriorShells:patchRadius:)` **[NEW]**
  - `voxelIndices(insideBox:)` / `setVoxelsForMesh(_:divisions:interiorShells:exteriorShells:patchRadius:)` **[NEW]**
  - `union(with:)`, `intersect(with:)`, `difference(with:)` — CSG Boolean ops **[NEW]**
  - `mesh(using:)` — convert voxels back to polygon mesh **[NEW]**
- `MTKMesh` / `MTKMeshBuffer` (MetalKit integration) **[NEW]**
- `SCNScene(mdlAsset:)` / `SCNNode(mdlObject:)` / `SCNGeometry(mdlMesh:)` (SceneKit integration) **[NEW]**

## Code Highlights

Import and export:
```swift
let asset = MDLAsset(url: url)
asset.exportAssetToURL(exportURL)
```

Generate ambient occlusion per-vertex (one liner):
```swift
shipMesh.generateAmbientOcclusionVertexColors(
    withQuality: 1.0, attenuationFactor: 0.98,
    objectsToConsider: [shipMesh], vertexAttributeNamed: MDLVertexAttributeOcclusionValue)
```

Voxelize a mesh and perform CSG:
```swift
let voxels = MDLVoxelArray(asset: asset, divisions: 64,
    interiorShells: 1, exteriorShells: 1, patchRadius: 0)
let result = voxels.union(with: otherVoxels)
let outMesh = result.mesh(using: allocator)
```

SceneKit integration from MDLAsset:
```swift
let scene = SCNScene(mdlAsset: asset)
```

## Takeaways
- Model I/O eliminates the custom file-format parser problem for 3D content — one API covers Alembic, OBJ, PLY, STL, and more.
- Physically based lights (IES profiles, color temperature), materials (BRDF), cameras (lens model), and skies are first-class citizens, enabling production-quality visuals without implementing the underlying physics math.
- Offline baking (ambient occlusion, light maps) via the Xcode editor or the API produces visuals that are computationally prohibitive at runtime, at zero runtime performance cost.
- SceneKit, MetalKit, and GLKit all receive hardware-ready buffers directly from MDLAsset/MDLMesh — no intermediate conversion code needed.

---
_Source: WWDC15 Session 602 page (abstract, chapter summaries, code samples, and resource links)._
