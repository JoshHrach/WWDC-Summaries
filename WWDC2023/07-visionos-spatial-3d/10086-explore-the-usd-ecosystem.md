# Explore the USD Ecosystem
**WWDC23 · Session 10086** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10086/)

_Platforms:_ visionOS 1, iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
This session surveys the state of Universal Scene Description (USD) and MaterialX on Apple platforms in 2023, with a particular focus on visionOS (then called xrOS). USD — originally created by Pixar and now known as OpenUSD — is the foundational 3D interchange format across Apple's ecosystem. With the arrival of visionOS, USD and the newly supported MaterialX shader standard become the primary authoring formats for immersive spatial content.

MaterialX, an open-source shading standard originally from Industrial Light & Magic, is introduced to visionOS via RealityKit's Shader Graph in Reality Composer Pro. It allows artists to build custom physically-based and unlit shaders through a node graph, with the resulting MaterialX graph embedded directly inside USD files. Apple has also contributed Metal shader code generation to the MaterialX 1.38.7 open-source release.

Color management improvements in RealityKit now support Display P3 textures, enabling wider gamut 3D content that accurately represents the full color range of Apple displays. Third-party application support for USD is also expanding: Blender 3.5 adds native USDZ import/export and Metal rendering, and Metal support has been added to Hydra Storm (Pixar's real-time USD renderer), enabling feature-film-scale scenes on Apple Silicon hardware.

## Key Topics

### USD on visionOS
- USD (OpenUSD) is the core 3D content format for visionOS; USDZ is the delivery format.
- Quick Look on visionOS displays USDZ assets spatially.
- Freeform app now embeds USDZ content across macOS, iOS, iPadOS, and visionOS.
- Safari adds a `<model>` HTML element for embedding USDZ in web pages (enabled via Developer menu / Settings).

### MaterialX in RealityKit and Reality Composer Pro
- MaterialX shader graphs embedded in USD files are now supported on visionOS via RealityKit.
- Reality Composer Pro introduces a visual Shader Graph editor for authoring MaterialX nodes.
- Key MaterialX node types: RealityKit PBR, RealityKit Unlit, Geometry Modifier, and custom utility nodes.
- Apple contributed Metal backend to MaterialX 1.38.7 for GPU shader code generation.
- Third-party support in Houdini (SideFX) and Maya (Autodesk LookdevX).

### Color Management in RealityKit
- RealityKit now supports Display P3 color-space-tagged textures for wider gamut rendering.
- Textures tagged with Display P3 express up to 25% more colors than sRGB.
- Proper color space tagging prevents accidental color shifts when authoring content.

### RealityKit Custom USD Schemas
- RealityKit introduces new ECS Component schemas for visionOS: `RealityKitComponent` (built-in) and `RealityKitCustomComponent` (custom Swift components).
- Custom Swift struct components map to equivalent USD prim representations.
- Spatial audio extended with `RealityKit Audio File`, `Audio File Group`, and `MixGroup` schemas.

### USD Ecosystem Updates
- Apple platforms updated to a newer, more efficient USD version; affects Motion, Quick Look, and Preview.
- Storm renderer in macOS Preview gains support for additional USD schemas.
- Metal support added to Hydra Storm: enables interactive rendering of film-scale USD assets (e.g., Animal Logic ALab at 26+ GB) on MacBook Pro via Apple Silicon unified memory.
- Maya USD plugin: contributions to geometry/material export and animation import.
- Blender 3.5: native USDZ import/export, Metal Eevee/Cycles rendering (up to 4x faster viewport, up to 2x faster final renders vs CPU).
- OpenUSD.org now lists compatible DCC applications.
- USD library dependency reduction for easier integration into iOS and other platforms.

## APIs & Frameworks
- `RealityKit` — Apple's real-time 3D rendering and ECS framework for visionOS/iOS/macOS **[NEW visionOS support]**
- `USD` / `OpenUSD` — Universal Scene Description interchange format (Pixar open source)
- `USDZ` — USD zip container format for delivery on Apple platforms
- `MaterialX` — open-source shading node graph standard; version 1.38.7 adds Metal code generation **[NEW]**
- `RealityKit PBR` MaterialX node — physically based rendering shader **[NEW]**
- `RealityKit Unlit` MaterialX node — unlit/stylized shader **[NEW]**
- `Geometry Modifier` MaterialX node — surface deformation shader **[NEW]**
- `RealityKitComponent` USD schema — built-in ECS component stored in USD **[NEW]**
- `RealityKitCustomComponent` USD schema — custom Swift ECS component stored in USD **[NEW]**
- `RealityKit Audio File` USD schema — spatial audio asset reference **[NEW]**
- `RealityKit Audio File Group` USD schema — grouped spatial audio assets **[NEW]**
- `RealityKit MixGroup` USD schema — audio mix group **[NEW]**
- `spatialAudio` USD schema — base USD spatial audio representation (extended by RealityKit)
- `Hydra` — USD renderer abstraction framework; now has Metal backend via Apple/Pixar collaboration **[NEW Metal support]**
- `Storm` — real-time Hydra renderer; Metal support added **[NEW Metal support]**
- `<model>` HTML element — Safari element for embedding USDZ in web pages **[NEW]**
- Reality Composer Pro — macOS app for preparing USD/USDZ content for visionOS **[NEW]**
- Quick Look — USDZ viewer; now available on visionOS **[NEW visionOS]**

## Code Highlights
The session shows a USD file snippet with a custom RealityKit component prim alongside geometry prims, and a corresponding Swift `Component` struct that reads the USD data (e.g., `EngagementPoint` values). No full code listings are provided; the core pattern is:

```usda
def RealityKitCustomComponent "EngagementComponent" {
    # Swift struct fields represented as USD attributes
    float engagementRadius = 0.5
}
```

Mapped to a Swift struct conforming to `RealityKit.Component` that reads these USD-authored values at runtime.

## Takeaways
- visionOS makes USD and USDZ first-class citizens; all spatial content should be authored as OpenUSD with MaterialX shaders for custom appearances.
- MaterialX 1.38.7 with Apple's Metal backend opens a path to portable, GPU-accelerated shader authoring on Apple platforms.
- Display P3 color tagging in textures is critical for achieving visually accurate wide-gamut content on Apple displays.
- Apple Silicon unified memory and Metal Hydra Storm enable interactive workflows with feature-film-scale USD scenes previously requiring desktop workstations.

---
_Source: WWDC23 Session 10086 page (abstract, chapter summaries, code samples, and resource links)._
