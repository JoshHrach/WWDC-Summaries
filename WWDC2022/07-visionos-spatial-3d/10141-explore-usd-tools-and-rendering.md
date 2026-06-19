# Explore USD tools and rendering
**WWDC22 · Session 10141** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10141/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This session covers two major areas of the USD ecosystem on Apple platforms: updates to USD authoring and conversion tools (USDZ Tools, Reality Converter) and a deep dive into USD Hydra rendering — what Hydra is, how it decouples scene graphs from renderers, and how to integrate it into a macOS app using the open-source Metal-accelerated Storm renderer.

New capabilities include Python 3 / Apple Silicon support in USDZ Tools, automatic USD asset repair in Reality Converter, a new `preferredIblVersion` metadata key for controlling AR Quick Look lighting, and the public release of Metal-accelerated Storm (USD 22.05). A complete open-source sample project (HydraPlayer) demonstrates building a full USD viewport renderer with Hydra on macOS.

## Key Topics

**USDZ Tools updates** — Command-line Python tools for generating, validating, and inspecting USDZ files. New: Python 3 and Apple Silicon support, upgraded USD version (22.05), new `usdzconvert` flags: `--useObjMtl` (OBJ materials), GLTF points/lines support, FBX scene-time animations, `--preferrediblversion` flag.

**Reality Converter updates** — macOS GUI tool wrapping USDZ Tools. New: streamlined texture inspection and material editing UI, JPEG texture compression on export (up to 80% file size reduction), automatic USD asset repair (mismatched attributes, missing default prim, deprecated attributes, missing stage metadata), Apple Silicon native binary.

**AR Quick Look new lighting (iOS 16)** — New `preferredIblVersion` metadata key in USDZ stage layer metadata. Value `0` = system default; `1` = classic lighting; `2` = new improved lighting (brighter, higher contrast, better shape definition). Assets created after July 1, 2022 default to new lighting; older assets keep classic lighting. Set explicitly via `usdzconvert --preferrediblversion 2` or Reality Converter property panel.

**USD Hydra architecture** — Hydra is an abstract rendering transport layer from Pixar's open-source USD project. It decouples scene graphs (connected via Scene Delegates) from renderers (connected via Render Delegates), allowing the same scene to be rendered by different backends (Storm for fast preview, RenderMan for final render, RealityKit for AR). Metal-accelerated Storm is now open-source as of USD 22.05.

**HydraPlayer sample app** — Demonstrates loading a USD stage with `UsdStage::Open`, setting up a camera and ambient light, initializing `UsdImagingGLEngine` (Storm) with a root path pointing at the stage, and issuing render calls. Available as a public sample project.

## APIs & Frameworks

### USD C++ / Objective-C++ API
- `UsdStage::Open(filePath)` — load a USD stage from a file path
- `UsdStage::GetPseudoRoot()` — returns the root prim of the stage
- `SdfPath` — USD scene path type
- `SdfPath::AbsoluteRootPath()` — the absolute root path `/`
- `UsdImagingGLEngine` — Storm renderer class (Hydra render delegate for Metal-accelerated preview)
- `UsdImagingGLEngine::SetCameraState(modelViewMatrix, projMatrix)` — set camera matrices
- `UsdImagingGLEngine::SetLightingState(lights, material, sceneAmbient)` — set scene lighting
- `UsdImagingGLRenderParams` — render configuration struct
- `UsdImagingGLRenderParams.clearColor` — `GfVec4f` background/clear color
- `UsdImagingGLRenderParams.frame` — `UsdTimeCode` for animated scenes
- `UsdImagingGLEngine::Render(stage, params)` — execute a render pass
- `GlfSimpleLight` — simple scene light description
- `GfMatrix4d` — 4×4 double-precision matrix
- `GfVec4f` — 4-component float vector

### USDZ Tools (Python command-line)
- `usdzconvert` — convert OBJ / GLTF / FBX to USDZ
- `--useObjMtl` **[NEW]** — include OBJ material definitions
- `--preferrediblversion <0|1|2>` **[NEW]** — set lighting version metadata
- `usdzip`, `usdcheck`, `usdcat`, `usddiff` — USD inspection and validation utilities

### USD Stage Metadata
- `customLayerData["Apple"]["preferredIblVersion"]` **[NEW]** — int metadata key: `0` system default, `1` classic lighting, `2` new lighting
- Set in `.usda` file or via `usdzconvert --preferrediblversion`

### RealityKit (reference)
- AR Quick Look — system USD viewer; respects `preferredIblVersion` for lighting
- Object Capture API — photogrammetry-based USDZ creation (macOS)
- RoomPlan API — parametric room scan to USDZ (new in iOS 16)

## Code Highlights

Setting new lighting in USDA metadata:
```usda
#usda 1.0
(
    customLayerData = {
        dictionary Apple = {
            int preferredIblVersion = 2
        }
    }
)
```

Loading a USD stage and initializing Storm in Objective-C++:
```objc
// Load stage
_stage = UsdStage::Open([filePath UTF8String]);

// Initialize Storm engine
_engine.reset(new UsdImagingGLEngine(
    _stage->GetPseudoRoot().GetPath(),
    excludedPaths,
    SdfPathVector(),
    SdfPath::AbsoluteRootPath(),
    driver));

// Render
_engine->SetCameraState(modelViewMatrix, projMatrix);
_engine->SetLightingState(lights, _material, _sceneAmbient);
UsdImagingGLRenderParams params;
params.clearColor = GfVec4f(0.0f, 0.0f, 0.0f, 0.0f);
params.frame = timeCode;
```

Building USD + Hydra from source (macOS):
```bash
arch -x86_64 /bin/zsh   # Apple Silicon (Rosetta while arm64 support transitions)
git clone https://github.com/PixarAnimationStudios/USD.git
python3 USD/build_scripts/build_usd.py --generator Xcode --no-python USDInstall
```

## Takeaways
- Set `preferredIblVersion = 2` in all new USDZ assets to opt into the new AR Quick Look lighting; Reality Converter applies it by default for new exports.
- Metal-accelerated Storm is now open-source (USD 22.05) — use the HydraPlayer sample as a starting point for building USD viewport tools on macOS.
- Hydra's Scene Delegate / Render Delegate architecture lets you swap renderers (Storm for preview, RenderMan for final, custom renderer for your pipeline) without changing your scene graph code.
- Reality Converter's automatic USD repair feature can fix common authoring issues (missing default prim, mismatched attributes, deprecated metadata) without manual USD editing.

---
_Source: WWDC22 Session 10141 page (abstract, chapter summaries, code samples, and resource links)._
