# What's New in USD and MaterialX
**WWDC24 · Session 10106** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10106/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2

## Overview
This session covers Apple's 2024 advances in Universal Scene Description (USD) and MaterialX across two rendering engines (RealityKit and Storm) and system tooling on macOS. The three major areas are: MaterialX ShaderGraph node support extended to iOS, iPadOS, and macOS (previously visionOS-only); new USD features in RealityKit (blend shapes, subdivision surfaces, variant API); and a suite of macOS tooling improvements in Preview, Finder, and the command line. Apple also highlights its role in founding the Alliance for OpenUSD (AOUSD) and its open-source contributions to USD, MaterialX, Blender, and Maya USD plugins.

## Key Topics

### MaterialX in RealityKit — Expanded to All Platforms
MaterialX shader support via ShaderGraph (Reality Composer Pro) was introduced for visionOS at WWDC23. It now extends to iOS, iPadOS, and macOS in RealityKit — delivering the same visual quality across all platforms from a single shader definition. Many bespoke ShaderGraph nodes (platform-specific nodes for unique Apple platform capabilities) are being published online as node definitions for integration into third-party DCC applications. Node definitions and introductions are in the ShaderGraph documentation.

### Storm Renderer — MaterialX and Lighting Updates
Storm (Pixar's real-time renderer, included with OpenUSD and used in macOS Preview and DaVinci Resolve) now supports MaterialX, including many ShaderGraph nodes. Storm's lighting model is also updated to more closely match RealityKit, providing a more consistent look when viewing USD assets across macOS applications. Because the two renderers target different use cases, some rendering differences and node support gaps may still exist.

### USD in RealityKit — New Features
**Blend shapes**: USD blend shapes (`UsdSkel`) are now supported in RealityKit, enabling expressive character animation (smiles, frowns, Animoji-style deformations) directly from USD assets without custom rigging.

**Subdivision surfaces**: Subdivision surface meshes can now be loaded and rendered at runtime via RealityKit, producing incredibly smooth objects without high polygon counts in the source asset.

**Variant API**: RealityKit's API supports specifying variants when loading a USD file programmatically, in addition to the variant-switching UI in Quick Look (where variants must be declared on the default prim in the USD scene).

### macOS System Tooling
**Preview app**: A new "Adjust Size" tool lets users scale and re-orient 3D models without launching a 3D editor. Texture compression is available on export to reduce file size. Lighting updated to match RealityKit.

**Finder improvements**: New "Convert to USDZ" Shortcuts action (right-click in Finder). Archive Utility now extracts USDZ archives via right-click. Finder loads pre-rendered preview thumbnails for large USD files, avoiding slow real-time rendering during browsing. Better management of large USD scenes without affecting system performance.

**Command-line tools**: Several USD command-line tools now ship with macOS for technical creators and pipeline automation:
- `usdcat` — convert between USD formats
- `usdchecker` — validate USD files
- `usdzip` — create portable USDZ packages
- `usdcrush` — compress USDZ files

**Unicode prim names**: OpenUSD (via contributions from NVIDIA and Pixar) now supports Unicode names for prims, enabling USD content in Hindi, Chinese, and other non-Latin scripts.

### Alliance for OpenUSD (AOUSD)
Apple co-founded AOUSD with Pixar, Adobe, Autodesk, and NVIDIA in August 2023. The alliance has grown fivefold to include leaders across gaming, e-commerce, and web technologies. AOUSD is developing a formal specification for the core of USD. Apple contributes to USD, MaterialX, Blender, and Maya USD plugins. Pixar's OpenUSD team received an Academy Award for Scientific and Technical Achievement.

## APIs & Frameworks

**RealityKit**
- MaterialX ShaderGraph support on iOS, iPadOS, macOS **[NEW]** (previously visionOS-only)
- USD blend shapes (`UsdSkel`) **[NEW]** — expressive character animation at runtime
- Subdivision surface mesh support **[NEW]** — smooth runtime geometry from low-polygon source
- Variant specification when loading USD files via API **[NEW]**
- USD variant configuration support (complements Quick Look variant UI)

**USD / USDZ (File Format)**
- Blend shapes — `UsdSkel` blend shapes now supported in RealityKit
- Subdivision surfaces — recognized and rendered by RealityKit
- Variants — must be declared on default prim for Quick Look interactive switching
- Unicode prim names **[NEW]** — via OpenUSD community contributions

**MaterialX / ShaderGraph**
- MaterialX in Storm renderer **[NEW]**
- Many ShaderGraph bespoke Apple nodes published as node definitions for DCC tools **[NEW]**
- ShaderGraph node documentation available at developer.apple.com/documentation/ShaderGraph

**macOS System Tooling**
- Preview "Adjust Size" tool for USD/USDZ **[NEW]**
- Preview texture compression on export **[NEW]**
- Storm lighting updated to match RealityKit **[NEW]**
- Shortcuts "Convert to USDZ" action **[NEW]**
- Archive Utility USDZ extraction (right-click) **[NEW]**
- Finder pre-rendered thumbnail support for USD **[NEW]**
- `usdcat` — USD format conversion **[NEW in macOS]**
- `usdchecker` — USD validation **[NEW in macOS]**
- `usdzip` — USDZ packaging **[NEW in macOS]**
- `usdcrush` — USDZ compression **[NEW in macOS]**

**Quick Look**
- USD variant interactive switching **[NEW]** — requires variants on default prim
- RealityKit API for specifying variants on load **[NEW]**

## Code Highlights
No code samples are included in this session. Integration points are:
- Use Reality Composer Pro's ShaderGraph to author MaterialX materials for cross-platform RealityKit apps
- Declare USD variant sets on the `defaultPrim` in USDZ files for Quick Look to surface configuration UI
- Use `usdchecker` in CI pipelines to validate USD assets before shipping
- Use `usdcrush` to compress USDZ for faster App Store downloads and web delivery

## Takeaways
- MaterialX via ShaderGraph is now the recommended shading workflow for all Apple platforms (iOS, iPadOS, macOS, visionOS) — one shader definition produces consistent results everywhere RealityKit runs.
- USD blend shapes and subdivision surfaces in RealityKit enable high-quality character and product visualization directly from standard USD assets, without custom engine work.
- The macOS command-line tools (`usdcat`, `usdchecker`, `usdzip`, `usdcrush`) make USD pipeline automation scriptable — add `usdchecker` to CI to catch asset issues before shipping.
- Declare all USD variants on the `defaultPrim` so Quick Look can discover and surface them as interactive configuration options; RealityKit's API also lets you specify a variant at load time.

---
_Source: WWDC24 Session 10106 page (abstract, chapter summaries, and resource links)._
