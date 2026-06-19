# Explore Advanced Rendering with RealityKit 2
**WWDC21 · Session 10075** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10075/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
This session is the rendering-focused companion to "Dive into RealityKit 2" (Session 10074), covering the three most impactful new graphical capabilities added in 2021: custom shaders (geometry modifiers and surface shaders written in Metal Shading Language), post-processing effects (via Core Image, Metal Performance Shaders, SpriteKit, or custom Metal compute kernels), and dynamic mesh creation and updates at runtime.

All four features are demonstrated using the same immersive underwater aquarium demo app. Geometry modifiers are used for the swaying seaweed animation; surface shaders power the octopus color-transition effect; a custom Metal compute kernel implements depth-aware fog blending ARKit scene depth with RealityKit virtual content depth; and the dynamic mesh API creates an animated spiral that contours around the diver's body by inspecting the diver model's vertex data at runtime.

## Key Topics
- **Geometry Modifiers:** Metal Shading Language functions with the `[[visible]]` attribute and `realitykit::geometry_parameters` parameter type. Run in the vertex shader for every vertex, every frame. Can read/write model position offset, normals, UVs, and custom attributes. Attached to a `CustomMaterial` via `CustomMaterial.GeometryModifier`. Access world position, model position, time, material constants, and textures through the `params` object.
- **Surface Shaders:** Metal functions with `realitykit::surface_parameters`. Run in the fragment shader for every visible pixel. Can write base color, normal, roughness, metallic, ambient occlusion, specular, emissive. Attached to `CustomMaterial` via `CustomMaterial.SurfaceShader`. Can combine both geometry modifier and surface shader on the same `CustomMaterial`.
- **Post Processing:** `ARView.renderCallbacks.prepareWithDevice` (called once with `MTLDevice`) and `ARView.renderCallbacks.postProcess` (called every frame with `ARView.PostProcessContext` providing `sourceColorTexture`, `targetColorTexture`, `depth`, `commandBuffer`, `device`, `time`). Usable with Core Image (`CIContext`, `CIFilter`, `CIRenderDestination`), Metal Performance Shaders (`MPSImageThresholdToZero`, `MPSImageGaussianBlur`, `MPSImageAdd`), SpriteKit (`SKRenderer`), or custom Metal compute kernels.
- **Dynamic Meshes:** `MeshResource.contents` exposes `MeshResource.Contents` with `instances`, `models`, and `parts`. Each part contains vertex data: `positions`, `normals`, `textureCoordinates`, `triangleIndices`, etc. `MeshDescriptor` enables creating new meshes from raw geometry data; `MeshResource.generate(from: [MeshDescriptor])` runs the mesh optimizer. `MeshResource.replace(with: MeshResource.Contents)` updates existing mesh contents without re-optimization—used for per-frame updates.

## APIs & Frameworks

**RealityKit**
- `CustomMaterial` **[NEW]** – Material type supporting custom Metal shaders
- `CustomMaterial(from: baseMaterial, geometryModifier:)` **[NEW]** – Inherits base material properties
- `CustomMaterial(from: baseMaterial, surfaceShader:)` **[NEW]** – With surface shader
- `CustomMaterial.GeometryModifier(named:in:)` **[NEW]** – Loads geometry modifier from Metal library
- `CustomMaterial.SurfaceShader(named:in:)` **[NEW]** – Loads surface shader from Metal library
- `ARView.renderCallbacks.prepareWithDevice` **[NEW]** – `(MTLDevice) -> Void`
- `ARView.renderCallbacks.postProcess` **[NEW]** – `(ARView.PostProcessContext) -> Void`
- `ARView.PostProcessContext` **[NEW]** – Provides `sourceColorTexture`, `targetColorTexture`, `depth`, `commandBuffer`, `device`, `time`
- `MeshResource.contents` **[NEW]** – `MeshResource.Contents` accessor
- `MeshResource.Contents` **[NEW]** – Contains `instances`, `models` (keyed), `MeshResource.Part` entries
- `MeshResource.Model` **[NEW]** – Contains `parts: [MeshResource.Part]`
- `MeshResource.Part` **[NEW]** – Contains `positions`, `normals`, `textureCoordinates`, `triangleIndices`
- `MeshResource.Instance` **[NEW]** – Contains `model` key and `transform`
- `MeshDescriptor` **[NEW]** – Describes raw mesh geometry: `positions`, `normals`, `primitives`, `textureCoordinates`
- `MeshDescriptor.primitives` **[NEW]** – `.triangles([UInt32])`, `.quads(...)`, `.polygons(...)`
- `MeshResource.generate(from: [MeshDescriptor])` **[NEW]** – Creates optimized mesh resource
- `MeshResource.replace(with: MeshResource.Contents)` **[NEW]** – Updates mesh contents in place

**Metal Shading Language (RealityKit headers)**
- `#include <RealityKit/RealityKit.h>` – RealityKit MSL header
- `[[visible]]` attribute – Makes function linkable from RealityKit
- `realitykit::geometry_parameters` – Geometry modifier parameter type
  - `params.geometry().world_position()` – Current vertex world position
  - `params.geometry().model_position()` – Current vertex model position
  - `params.geometry().set_model_position_offset(float3)` – Write vertex offset
  - `params.uniforms().time()` – Current frame time
  - `params.textures().custom()` – Custom texture slot
  - `params.material_constants().base_color_tint()` – Material tint
- `realitykit::surface_parameters` – Surface shader parameter type
  - `params.surface().set_base_color(half3)` – Write base color
  - `params.surface().set_normal(float3)` – Write surface normal
  - `params.surface().set_roughness(half)` – Write roughness
  - `params.surface().set_metallic(half)` – Write metallic
  - `params.surface().set_ambient_occlusion(half)` – Write AO
  - `params.surface().set_specular(half)` – Write specular
  - `params.geometry().uv0()` – Texture coordinate
  - `params.textures().base_color()`, `.normal()`, `.roughness()`, `.metallic()`, `.ambient_occlusion()`, `.emissive_color()`, `.custom()` – Texture slots
  - `realitykit::unpack_normal(half3)` – Unpack normal map value
- `ARKit.ARFrame.displayTransform(for:viewportSize:)` – Used to orient ARKit depth texture

**Core Image, Metal Performance Shaders, SpriteKit** – All usable within `postProcess` callback (no new APIs, existing frameworks integrated with new `renderCallbacks`)

## Code Highlights
Geometry modifier for wave animation (MSL):
```metal
[[visible]]
void seaweedGeometry(realitykit::geometry_parameters params) {
    float3 worldPos = params.geometry().world_position();
    float3 modelPos = params.geometry().model_position();
    float time = 0.1 * params.uniforms().time();
    float3 maxOffset = float3(sin(8.0 * (worldPos.x + time)),
                              sin(8.0 * (worldPos.y + time)),
                              sin(8.0 * (worldPos.z + time)));
    float3 offset = maxOffset * 0.05 * max(0.0, modelPos.y);
    params.geometry().set_model_position_offset(offset);
}
```

Applying a custom material with geometry modifier (Swift):
```swift
let library = MTLCreateSystemDefaultDevice()!.makeDefaultLibrary()!
let geometryModifier = CustomMaterial.GeometryModifier(named: "seaweedGeometry", in: library)
seaweed.model!.materials = seaweed.model!.materials.map { baseMaterial in
    try! CustomMaterial(from: baseMaterial, geometryModifier: geometryModifier)
}
```

Post-process using MPS bloom (Swift):
```swift
arView.renderCallbacks.postProcess = { context in
    let brightness = MPSImageThresholdToZero(device: context.device, thresholdValue: 0.2, linearGrayColorTransform: nil)
    brightness.encode(commandBuffer: context.commandBuffer, sourceTexture: context.sourceColorTexture, destinationTexture: bloomTexture)
    let blur = MPSImageGaussianBlur(device: context.device, sigma: 9.0)
    blur.encode(commandBuffer: context.commandBuffer, inPlaceTexture: &bloomTexture)
    let add = MPSImageAdd(device: context.device)
    add.encode(commandBuffer: context.commandBuffer, primaryTexture: context.sourceColorTexture, secondaryTexture: bloomTexture, destinationTexture: context.targetColorTexture)
}
```

## Takeaways
- Geometry modifiers and surface shaders unlock the ability to express any visual effect in Metal while still benefiting from RealityKit's PBR lighting, shadow casting, and AR integration.
- The `renderCallbacks` post-processing pipeline integrates cleanly with Core Image, MPS, and SpriteKit, making it practical to add sophisticated fullscreen effects without writing all infrastructure from scratch.
- Dynamic meshes via `MeshResource.Contents` and `MeshDescriptor` enable procedural geometry that responds to scene data (e.g., contouring a spiral to a character's silhouette by inspecting its vertex positions at runtime).
- For per-frame mesh updates, use `MeshResource.replace(with:)` on existing `Contents` rather than regenerating through `MeshDescriptor`, which re-runs the mesh optimizer and is too expensive for real-time use.

---
_Source: WWDC21 Session 10075 page (abstract, chapter summaries, code samples, and resource links)._
