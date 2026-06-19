# Optimize Your 3D Assets for Spatial Computing
**WWDC24 · Session 10186** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10186/)

_Platforms:_ visionOS, visionOS 2

## Overview
Apple Vision Pro's high-resolution displays and demanding frame rates make 3D asset optimization critical. This session, presented by a technical artist on the Apple Vision Pro Apps and Content team, walks through an end-to-end workflow for building a performant immersive scene: from choosing polygon budgets and export formats, to texture packing and color spaces, to material strategies (unlit with baking, PBR, and the hybrid `EnvironmentRadiance` node), sky dome setup, and custom image-based lighting.

The session uses a sample outdoor scene built in Blender 4.1 and assembled in Reality Composer Pro, demonstrating each optimization step in context. A companion sample project is available for download.

## Key Topics

**Before You Begin**
- Determine how the content will be viewed: immersive (full-screen, much higher GPU cost) vs. shared space (other apps may be running). Immersive apps require the most optimization.
- The GPU on Vision Pro only renders pixels your app draws — not passthrough video — so workload scales with how large your app appears. Test early and often.

**Polygon Count**
- Recommended maximum: 500,000 triangles for immersive scenes, 250,000 for shared-space apps — measured by what is visible at any one time
- Conservative safe target: ~100,000 triangles in view at once, leaving headroom
- Split large objects (terrain, environment) into chunks to enable frustum culling of off-camera sections
- Use a camera placed at average eye height (1.5 m) in your DCC tool to judge on-screen size and appropriate polygon density

**Exporting from DCCs (Digital Content Creation tools)**
- USD is the required output format; any tool that exports USD (Blender, Autodesk Maya, SideFX Houdini, etc.) works
- `USDA` — ASCII text format; human-readable, merge-conflict-resolvable; used internally by Reality Composer Pro for scene files
- `USDC` — binary format; compact and efficient for large geometry datasets; preferred for mesh assets
- `USDZ` — zipped package bundling all textures; ideal for distribution and Quick Look; not editable without unzipping
- Coordinate system mismatch: RealityKit uses Y-up / −Z-forward / meters; Blender uses Z-up / Y-forward. Correct on import with a −90° X rotation. New in macOS this year: Preview can remap coordinates and scale.

**Efficient Texture Use**
- Color spaces in Reality Composer Pro: `sRGB – Display P3` for perceptual textures (base color, unlit color); `Linear` for data/HDR
- Vision Pro's native gamut is Display P3; other gamuts are converted at Xcode build time
- Grayscale data textures (roughness, metallic, AO) are not GPU-compressed when standalone — pack them into RGB channels of a single color texture to enable compression and reduce asset size by up to 40%
- In Shader Graph, use the `vector3` type for the packed texture node to suppress color-space transforms; split channels with a `Separate3` node
- Normal maps: RealityKit expects OpenGL format (DirectX has inverted green channel). Remap from [0,1] to [−1,1] range using a multiply-by-2 then subtract-1 pattern, or use the `Normal Map Decode` node (one-step equivalent)
- Scale texture resolution with distance and on-screen size; more than half of texture resolution in a typical immersive scene covers only the 5–10 m immediately around the viewer

**Material Instances**
- Create a "base" material with all shader logic, then promote file references to inputs
- Right-click → `Create Instance` to produce per-asset instances that share the compiled shader graph but swap textures independently
- Benefits: faster asset setup, and the engine loads one shader instead of redundant copies — direct performance win

**Optimizing Materials**
- Prefer unlit materials with baked lighting whenever possible — dynamic realtime lights in RealityKit have significant performance cost
- Bake all shading (diffuse, shadow, AO) into a single texture per object in your DCC tool; use unlit materials in Reality Composer Pro to render it
- Split scenes into chunks so off-camera entities are culled
- Minimize alpha transparency (overdraw): each transparent layer forces the GPU to re-evaluate the same pixels. Use geometry (e.g., individual grass blade meshes, opaque cores for foliage) instead of alpha cards where possible; trading triangles for fewer transparent pixels is almost always worth it within budget

**Sky Dome Setup**
- Use an inverted sphere (or any large encompassing geometry) as a sky dome; ensure diameter is large enough (e.g., 500 m) that the user never reaches its edge
- Sky dome textures need high resolution — at least 8K horizontal for scenes with fine detail; crop below the horizon if that area won't be seen
- Always use an unlit material on sky domes; they are among the highest-priority assets to optimize

**Image-Based Lighting (IBL)**
- A sky dome mesh does not contribute to PBR lighting — a separate IBL setup is required
- In Reality Composer Pro: add an `ImageBasedLight` component to an entity (the IBL source); attach an `ImageBasedLightReceiver` component to the parent of PBR objects and link it to the IBL entity
- IBL texture format: HDR equirectangular (Lat/Long); keep resolution low (e.g., 512 px wide) unless the scene has highly reflective/mirror surfaces
- `EnvironmentRadiance` Shader Graph node — a performance-efficient middle ground between unlit and full PBR: feeds IBL specular (and optionally diffuse) radiance into an unlit material graph, adding view-dependent reflections without the full PBR cost

**Profiling**
- Reality Composer Pro Statistics Panel — check triangle count, mesh/texture/material totals at build time (textures shown uncompressed; Xcode build compresses them, typically reducing ~1.2 GB → ~300 MB)
- RealityKit Trace — profile GPU and rendering performance
- RealityKit Debugger (new) — inspect entity hierarchy and detect code-level performance issues

## APIs & Frameworks

**Reality Composer Pro**
- `Shader Graph` — node-based material editor; builds MaterialX-compatible shader graphs
- `Normal Map Decode` node — remap normal map data from [0,1] to [−1,1] in one step
- `Separate3` node — split a packed RGB texture into individual R/G/B channel outputs
- `EnvironmentRadiance` node — sample IBL specular/diffuse radiance inside an unlit material graph
- Material Instances (`Create Instance`) — share a compiled shader graph across multiple assets with per-instance texture overrides
- `Promote to Input` — expose material file references as per-instance overridable parameters
- Statistics Panel — view triangle count, draw calls, and texture memory usage

**RealityKit**
- `ImageBasedLight` component — define a custom IBL source using an HDR equirectangular texture
- `ImageBasedLightReceiver` component — apply a specified IBL source to an entity and its descendants
- RealityKit coordinate system: Y-up, −Z-forward, meters
- RealityKit Trace — Instruments template for GPU and render performance profiling
- RealityKit Debugger **[NEW]** — entity-level scene debugging tool new in visionOS 2 / Xcode 16

**USD Formats**
- `USDA` — ASCII USD; editable, diffable, merge-friendly
- `USDC` — binary USD; compact for geometry-heavy assets
- `USDZ` — zipped USD package; bundles textures for distribution

## Code Highlights

No Swift/code samples were presented — this session is a DCC tool and Reality Composer Pro workflow walkthrough. Key numeric targets:

- Triangle budget: ≤500K (immersive), ≤250K (shared space), ~100K (safe conservative target)
- Texture packing: combine roughness/metallic/AO into one RGB texture → up to 40% size reduction + GPU compression enabled
- Sky dome texture: ≥8K horizontal resolution
- IBL texture: ~512 px wide (sufficient for non-mirror surfaces)
- Camera placement for DCC preview: 1.5 m eye height at scene center

## Takeaways
- Bake lighting into unlit materials wherever possible — dynamic lights and full PBR are expensive; reserve PBR for hero assets that must react to real-world environment changes.
- Pack grayscale textures (roughness, metallic, AO) into a single RGB texture to unlock GPU compression and reduce asset size by up to 40%.
- Use Material Instances in Reality Composer Pro to share a single compiled shader graph across multiple assets — faster iteration and better runtime performance.
- Use the `EnvironmentRadiance` Shader Graph node when an unlit material is not enough but a full PBR shader is overkill — it adds view-dependent specular reflections at lower cost than PBR.
- Always measure in the Statistics Panel and test on device early; immersive apps consume significantly more GPU than shared-space apps.

---
_Source: WWDC24 Session 10186 page (abstract, chapter list, and full transcript)._
