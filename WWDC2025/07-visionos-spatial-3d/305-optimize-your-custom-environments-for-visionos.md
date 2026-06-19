# Optimize your custom environments for visionOS
**WWDC25 · Session 305** · [Watch](https://developer.apple.com/videos/play/wwdc2025/305/)

_Platforms:_ visionOS

## Overview
Cinematic-quality immersive environments on Apple Vision Pro demand high visual fidelity, but real-time rendering has strict performance budgets. This session presents a complete, procedural optimization pipeline built in Houdini that takes a high-polygon source environment (100M+ triangles in the example) and produces a real-time-ready asset under 200K triangles and 250MB of texture memory — with no perceptible loss of visual quality.

Apple provides a freely downloadable toolkit of 14 custom Houdini Digital Assets (HDAs) that automate the core optimization steps: adaptive polygon reduction, billboard generation, occlusion culling, screen-space UV projection, texture baking, and USD export with frustum-culling partitions. The session walks through each stage using a Moon environment as the case study.

## Key Topics

### The Immersive Boundary as an Optimization Constraint
Vision Pro's Immersive Boundary — the 3m traversable space around the user — defines the exact viewpoints from which the environment will ever be seen. Every optimization decision keys off this boundary: triangles outside visible silhouettes from boundary viewpoints are removed, texture resolution is allocated proportional to screen-space area from those viewpoints, and UV projection origins are chosen to minimize per-boundary-viewpoint texel waste.

### Adaptive Polygon Reduction (Adaptive Reduce HDA)
Rather than uniform decimation, the HDA performs view-dependent reduction. Triangle density is preserved where silhouette edges are prominent from boundary sample positions and aggressively reduced everywhere else. The heat-map visualization in Houdini shows density proportional to view importance. The Moon example goes from 100M to ~350K triangles in this pass.

### Vista Billboards (Vista Billboard HDA)
Geometry beyond 1km is replaced with flat billboard meshes whose silhouettes are derived from the actual source geometry (no transparency cutouts needed). The billboard HDA raycasts vertices toward a cylindrical surface at the specified distance, retriangulates, and produces a mesh that matches the original silhouette vertex-for-vertex from inside the boundary.

### Occlusion Culling (Remove Backfaces + Occlusion Culling HDAs)
Two sequential culling passes run after polygon reduction:
1. **Backface removal** — dot-product comparison removes polygons that always face away from the boundary. Fast and reliable, removes ~60K triangles in the Moon.
2. **Occlusion culling** — ray-casts millions of visibility rays from boundary sample positions; polygons never hit by any ray are removed. Removes another ~110K triangles. Together, ~50% of post-reduction triangles are culled. Final count: ~180K.

### Screen-Space UV Projection (Mesh Partition + Multi-Projection HDAs)
Instead of surface-area UV mapping (which wastes texels on surfaces seen at glancing angles), the pipeline uses spherical projection from the center of the boundary. For geometry with multiple visible sides, the Mesh Partition HDA splits the mesh into islands, each projected from the viewpoint where it appears largest. The Multi-Projection HDA batch-processes all partitions.

### Texture Baking
Pre-rendered spherical images (captured with ray-traced lighting) are baked onto the optimized UV atlas using composited projection techniques. The entire Moon environment fits in two textures of similar size: one for within-boundary geometry (surface-area mapped) and one for beyond-boundary geometry (screen-space mapped). Total: under 250MB.

### USD Export with Frustum Culling (Boundary Partition + Frustum Partition HDAs)
The USD hierarchy is structured so the renderer can perform per-entity frustum culling at runtime. Inside-boundary geometry uses boundary-aware partitioning; outside-boundary geometry uses progressively larger tiles. Fewer than 100 draw calls per frame in the final Moon scene.

## APIs & Frameworks

- **Houdini Digital Assets (HDAs)** — 14 procedural tools provided by Apple **[NEW, downloadable]**
  - `Boundary Camera HDA` — visualizes Immersive Boundary viewpoints
  - `Boundary Samples HDA` — generates sample positions inside the boundary
  - `Adaptive Reduce HDA` — view-dependent polygon reduction
  - `Vista Billboard HDA` — silhouette-accurate billboard generation for distant geometry
  - `Remove Backfaces HDA` — dot-product backface culling
  - `Occlusion Culling HDA` — ray-cast visibility culling
  - `Mesh Partition HDA` — splits mesh into projection-optimal islands
  - `Multi-Projection HDA` — batch screen-space UV projection
  - `Boundary Partition HDA` — USD partitioning for in-boundary geometry
  - `Frustum Partition HDA` — USD partitioning for distant geometry frustum culling
- **USD / Reality Composer Pro** — target runtime formats for optimized assets
- **RealityKit** / **Unity** — supported runtime engines for the exported USD

## Code Highlights

_This session is a 3D art/pipeline session. No Swift or Xcode code is presented. The workflow is entirely within Houdini using the provided HDA toolkit._

```bash
# Download the toolkit
# https://developer.apple.com/download/files/Immersive-Optimization-Toolkit.zip
# Open optimize.hip in Houdini
# HDAs are in the /HDA folder, pre-referenced in the main file
```

## Takeaways

- The Immersive Boundary is not just a safety feature — it is a rendering budget tool. Every optimization should be keyed to what is visible from inside that boundary.
- View-dependent polygon reduction outperforms uniform decimation by 10–20x for immersive scenes; use the Adaptive Reduce HDA rather than Houdini's native PolyReduce directly.
- Screen-space UV projection compresses the entire environment into texture sizes comparable to just the first few meters of surface-area mapping.
- The 14 Apple-provided HDAs are free to download, modify, and embed in custom pipelines — start with the included `optimize.hip` sample to understand the full chain before customizing.

---
_Source: WWDC25 Session 305 page (abstract, chapter summaries, code samples, and resource links)._
