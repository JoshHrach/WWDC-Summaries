# Create 3D Models for Quick Look Spatial Experiences
**WWDC23 · Session 10274** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10274/)

_Platforms:_ visionOS 1, iOS 17, macOS Sonoma 14

## Overview
Quick Look on visionOS presents USDZ 3D models in volumetric windows that float in space, letting users reposition and scale them with familiar gestures. This session provides a complete guide to preparing high-quality 3D assets for Quick Look: how to create USDZ files using various Apple tools, how to inspect and fix visual quality issues in Reality Composer Pro, and how to optimize geometry, textures, materials, physics, animations, and particle systems for smooth performance on the new platform.

The session uses a detailed room model created with RoomPlan, refined in a third-party DCC tool, and previewed in Reality Composer Pro as the central example. It also introduces Reality Composer Pro's Statistics panel and RealityKit Trace as the primary profiling tools for understanding and diagnosing performance bottlenecks.

A key theme throughout is balance: finding the right trade-off between visual fidelity and runtime performance, since Quick Look content runs alongside other apps and must share system resources.

## Key Topics

### How Quick Look Presents 3D Content
Quick Look displays models inside a volumetric window with a right-handed coordinate system. The system automatically adds a ground plane and shadow—developers should not add their own. Quick Look respects `metersPerUnit` from USDZ metadata for real-world scale; content within a reasonable size range is displayed at 100% scale initially, with floor and ceiling limits for tiny or very large objects.

### Ways to Create USDZ Files
- **Existing iOS USDZ files** — compatible directly with visionOS Quick Look.
- **Digital Content Creation (DCC) tools** — create from scratch and export to USDZ.
- **RealityKit Object Capture** — photogrammetry from photos on iOS or macOS; produces models at multiple detail levels.
- **RoomPlan API** — scan real-world spaces and export USDZ floor plans and room models.

### Inspecting Visual Quality in Reality Composer Pro
Reality Composer Pro includes a device preview button that launches the asset directly in Quick Look on device. Key issues to watch for:
- Incorrect orientation (Quick Look faces the content toward the user on launch; fix by rotating the root in Reality Composer Pro or the DCC tool).
- Z-fighting from co-planar geometry — fix by removing overlaps or increasing separation.
- High-frequency normal map aliasing — causes visual artifacts especially in motion or at close range.
- Thin/small geometry flickering under variable rasterization — store fine details in an opacity texture on a larger triangle grid.

### Performance Optimization Guidelines

**File Size**
- Target under 25 MB for a good sharing experience.
- Remove unused audio, textures, and animations before distribution.
- Balance quality vs. file size on textures (lower resolution, lower bit depth where acceptable).

**Geometry**
- Remove hidden/covered geometry.
- Merge small meshes to reduce draw calls (target: under 200 mesh parts, under 100k vertices per model).
- Use LOD-appropriate detail for each object's screen footprint.

**Textures**
- Use grayscale for non-color inputs; pack multiple grayscale maps into individual channels of a single color texture.
- Use constant material values instead of textures where possible.
- Single PBR material: max 2K×2K, 8-bit per channel.
- Spend texture budget on the most visually impactful areas.

**Materials**
- Combine mesh parts that share the same material to reduce shader compilation overhead.
- Use a separate, lightweight material for small transparent areas rather than making the whole mesh transparent.
- Avoid unnecessary overlapping transparency.
- Use MaterialX Unlit surface for baked lighting or non-lit surfaces.

**Physics**
- Minimize the number of active colliders.
- Use static colliders for non-moving objects (e.g., walls).

**Animation**
- Limit weight-per-vertex count for skinned/deformation animations.
- Follow the same geometry guidelines for deformation targets.

**Particles**
- Limit the number of particle emitters and total on-screen particles.
- Experiment with simpler shapes to reduce overdraw.

### Profiling Tools
- **Reality Composer Pro Statistics panel** — triangle count, texture memory usage, and other asset characteristics.
- **RealityKit Trace** (new) — advanced runtime profiling; attach to the Quick Look process in Xcode, captures frame-by-frame rendering pipeline data and provides optimization recommendations.

## APIs & Frameworks

**RealityKit**
- `RealityKit Object Capture` — photogrammetry API for creating USDZ from photos (iOS, macOS)
- `RoomPlan` — room scanning API producing USDZ floor plan exports

**USD / USDZ**
- USDZ format — primary 3D asset format for Quick Look
- `metersPerUnit` metadata field — controls real-world scale in Quick Look
- MaterialX Unlit surface — material type for unlit/baked surfaces

**Developer Tools**
- **Reality Composer Pro** **[NEW]** — asset authoring, preview, and Statistics panel
- **RealityKit Trace** **[NEW]** — Xcode-integrated runtime performance profiler for RealityKit/Quick Look

**Quick Look (visionOS)**
- Volumetric window presentation — automatic ground plane, shadow, real-world scaling
- Pinch and drag gesture — rotate model
- Two-hand pinch gesture — scale model
- Volume window bar — reposition content in space

## Code Highlights
No code samples are included in this session (art/tools-focused content). Key workflow:
1. Create or export USDZ.
2. Open in Reality Composer Pro; check Statistics panel.
3. Click device preview to launch in Quick Look on device.
4. Fix orientation by rotating the root node (e.g., 90° around y-axis).
5. Attach RealityKit Trace in Xcode for performance diagnosis.

## Takeaways
- Existing iOS USDZ files work on visionOS Quick Look without changes; use Reality Composer Pro to preview and tweak orientation and quality.
- Quick Look auto-adds a ground plane and shadow—do not duplicate them in the asset.
- The biggest performance levers are geometry complexity (under 200 mesh parts, 100k vertices), texture size (max 2K, 8-bit), and minimizing overlapping transparency and particle emitters.
- Reality Composer Pro Statistics panel and RealityKit Trace are the go-to tools for diagnosing and resolving performance issues before shipping.

---
_Source: WWDC23 Session 10274 page (abstract, chapter summaries, and resource links)._
