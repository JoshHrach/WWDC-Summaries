# Create 3D Workflows with USD
**WWDC21 · Session 10077** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10077/)

_Platforms:_ macOS Monterey 12

## Overview
Universal Scene Description (USD), developed by Pixar, is a flexible, extensible file format and set of libraries for assembling, exchanging, and organizing 3D scenes. This session surveys the USD ecosystem updates arriving in 2021 and demonstrates a complete professional 3D pipeline — from real-world Object Capture through sculpting and simulation to AR delivery and offline rendering — built entirely on USD and USDZ.

Key highlights include: Metal-accelerated Hydra Storm renderer shipping in macOS Preview and Quick Look; native USD support in Maya 2022; new Physics simulation schema co-developed by Apple, Nvidia, and Pixar; and a full end-to-end creative workflow using Object Capture, ZBrush, Maya, Houdini, Reality Composer, AR Quick Look, and OTOY Octane — all exchanging data as USD without any import/export steps.

## Key Topics

### USD as Universal 3D Interchange
USD sits above older formats (OBJ, FBX, glTF) by supporting: millions of objects, a scene graph with hierarchical composition, non-destructive editing via "opinions," variants for switching asset configurations, and built-in collaboration through referencing (multiple artists edit separate USD files that are referenced into a single root scene). USDZ is the single-file archive form of USD, optimized for sharing and AR delivery — it bundles all textures and resources.

### USD Ecosystem Updates (2021)
- **Maya 2022**: First version with seamless native USD integration — loads gigabytes of USD data in seconds; USD data appears directly in the Outliner and Attribute Editor; Blendshape and texture support improved in collaboration with Apple and Autodesk.
- **ZBrush**: Reads and writes USD mesh data directly, enabling sculpting inside USD workflows.
- **SideFX Houdini**: Native USD support; procedural fluid simulations (e.g., pouring syrup) authored directly as USD and referenced into the main scene.
- **Multiverse (J-Cube)**: Non-destructive USD editing on Mac powered by Metal Hydra Storm.
- **OTOY Octane**: GPU path-traced rendering directly from USD files in DCC tools.
- **Physics schema**: New USD schema for rigid body simulation co-developed by Apple, Nvidia, and Pixar; supports AR physics through film-level offline simulation.

### Metal-Accelerated Hydra Storm in macOS
Preview and Quick Look in macOS Monterey ship a Metal-accelerated version of Pixar's Hydra Storm renderer. This enables anyone to view production-grade USD assets (tens of thousands of objects, 34 million triangles) interactively without a DCC tool. Features: camera selection from cameras defined in the USD file; an Outliner with object search; click-to-select objects in the viewport; camera lock to an object; export of selected objects as separate USD files; render animation to movie; render current view to image with alpha channel. For AR/USDZ assets, Preview uses the RealityKit rendering engine — the same renderer as AR Quick Look on iOS.

Pixar is making Metal-accelerated Hydra Storm available for integration in third-party apps through the USD Open Source project.

### USDZ Tools on Apple Developer Site
Apple provides command-line tools for batch processing USD/USDZ assets. Reality Converter (stand-alone utility) converts OBJ, Alembic, and other 3D formats to self-contained USDZ. USDZ files can be authored targeting a specific Apple device for tailored AR experiences.

### End-to-End USD Workflow Demonstration
The session demonstrates a full production pipeline: (1) Object Capture on physical pancakes → USDZ mesh with PBR materials; (2) Maya 2022 for scene layout, assembly of props, camera animation — all in USD; (3) ZBrush for sculpting missing geometry and polishing materials on the captured pancake model; (4) Houdini for procedural fluid (syrup) simulation and lighting, contributed back to the USD scene; (5) Reality Composer for final USDZ packaging for AR; (6) AR Quick Look on iPad to view the final scene in augmented reality; (7) OTOY Octane for a path-traced rendered video of the finished scene.

Because all tools natively read and write USD, there are no import/export steps. Any change made in one tool is immediately visible to all others via USD referencing.

## APIs & Frameworks

**USD / USDZ** (Pixar open-source + Apple extensions)
- USD scene graph composition: references, layers, variants, opinions (non-destructive overrides)
- USDZ: single-file USD archive for sharing; bundles geometry, materials, textures, and animations
- Physics schema (new): rigid body simulation properties, co-developed by Apple, Nvidia, and Pixar
- Hydra rendering architecture: pluggable rendering delegate; Hydra Storm = reference renderer
- Metal-accelerated Hydra Storm **[NEW macOS 12]** — available in Preview, Quick Look, and via open-source

**macOS Tools — RealityKit / AR**
- `Reality Converter` — GUI tool: convert OBJ/Alembic/FBX → USDZ with material and texture support
- USDZ command-line tools (developer.apple.com) — batch process and validate USDZ files
- `Reality Composer` — assemble USD scenes for AR; export AR-enabled USDZ
- AR Quick Look — render USDZ on iOS/iPadOS using RealityKit

**Third-Party DCC Integration**
- Maya 2022 USD plugin: native Outliner + Attribute Editor integration; Blendshape and texture export support
- ZBrush: direct USD mesh import/export
- Houdini: native USD scene graph; Karma USD renderer
- OTOY Octane: GPU path-trace rendering from USD via DCC plugin

## Code Highlights

USD referencing pattern (USDA text format — conceptual):
```usda
#usda 1.0
def Xform "Scene" {
    def "Layout" (references = @layout.usd@) {}
    def "Pancakes" (references = @sculpting.usd@) {}
    def "Syrup" (references = @houdini_fx.usd@) {}
    def "Lighting" (references = @lighting.usd@) {}
    def "Camera" (references = @camera_anim.usd@) {}
}
```

Exporting USDZ from command line (usdzip tool):
```bash
# Combine USD files and textures into a single USDZ archive
xcrun usdzip --arkitAsset scene.usd -o output.usdz
```

## Takeaways
- USD's referencing model eliminates import/export steps between tools — any artist's change propagates instantly to all downstream tools and stages.
- Metal-accelerated Hydra Storm in macOS Preview makes production-scale USD assets (millions of triangles) accessible to anyone on a Mac, not just DCC users.
- Object Capture feeds directly into a USD pipeline; ZBrush, Houdini, and Maya can each contribute non-destructively to the same USD scene in parallel.
- The same USD asset can simultaneously target AR Quick Look (via USDZ), interactive Preview, and offline rendering (via Hydra Storm or Octane) with no re-authoring.
- Apple, Pixar, and Nvidia's new Physics schema positions USD as the interchange format for both real-time AR simulation and high-fidelity offline physics.

---
_Source: WWDC21 Session 10077 page (abstract, chapter summaries, code samples, and resource links)._
