# The Artist's AR Toolkit
**WWDC20 · Session 10601** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10601/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
This session walks artists and technical artists through the complete pipeline for creating augmented reality content: from exporting 3D assets out of industry DCCs (Houdini, Blender, KeyShot), through Reality Converter for USDZ generation, into Reality Composer for interactive behavior authoring, and finally onto device via AR Quick Look or Xcode. The talk is deliberately artist-focused, emphasising visual tools and best practices rather than code.

Reality Converter (downloadable from developer.apple.com) is positioned as the successor to Apple's earlier Python-based USDZ tools. It accepts FBX, OBJ, USD, and GLTF geometry along with PNG/JPEG textures, applies PBR materials, lets artists choose IBL lighting environments, set copyright metadata, configure base units (meters or centimeters), and export a standards-compliant USDZ in bulk via "Export All." The key pitfall highlighted is unit mismatch: assets exported in centimeters from a DCC must have the same units declared in Converter or they will appear 100× too large or small in Reality Composer.

Reality Composer's behavior system chains trigger-action cards (Show, Hide, Move To, Orbit, Play USDZ Animation, Change Scene) into sequences evaluated left to right. The session demonstrates how to build ambient scene loops—vehicles driving in, making a turn via an Orbit card around a hidden pivot geometry, then fading out—and how to target individual entities within a USD hierarchy using Hierarchy Select. A new "splash scene" pattern is shown: duplicate the final scene, strip its behaviors, add a single Change Scene action on "Scene Start" so users see a static preview while the AR anchors.

## Key Topics
- **Reality Converter** — convert FBX, OBJ, USD, GLTF to USDZ; set PBR materials/textures, IBL lighting, copyright, base units; bulk export via "Export All" **[NEW tool]**
- **Supported DCC exports** — Houdini (USD with instancing hierarchy), Blender (OBJ), KeyShot (GLTF)
- **Unit configuration** — must match DCC export units (meters vs. centimeters) in Converter to avoid 100× scale error
- **USD hierarchy in Reality Composer** — Hierarchy Select lets artists target individual entities within a USDZ for per-instance behaviors
- **Reality Composer behaviors** — Show / Hide / Move To / Orbit / Play USDZ Animation / Change Scene action cards; evaluated left to right; duration and easing configurable per card
- **Splash scene / loading screen** — duplicate scene with no behaviors; Change Scene on Scene Start transitions to the live scene
- **USDZ export from Reality Composer** — new option (enable in Preferences) for embedding in web / News; Reality export remains the path for Xcode / AR Quick Look
- **iOS Handoff** — Mac ↔ iOS roundtrip for editing and validating AR scenes in context; Edit on iOS, then Done to pass back
- **AR Quick Look distribution** — AirDrop `.reality` file to device; opens directly in AR Quick Look

## APIs & Frameworks

**Reality Converter (macOS app) — tooling, not a code API**
- Accepted geometry formats: FBX, OBJ, USD (`.usd` / `.usda` / `.usdc`), GLTF (`.gltf` / `.glb`)
- Accepted texture formats: PNG, JPEG
- Output: USDZ (`.usdz`)
- IBL environment presets with exposure control
- Grid view (1 m grid lines) for scale reference
- Materials panel: drag-and-drop texture inputs (base color, normal, ambient occlusion, roughness, metallic)
- Properties panel: copyright string, base units (meters / centimeters)
- Animated asset indicator (running man icon)
- File > Export All — batch USDZ export

**Reality Composer (macOS / iOS / iPadOS app)**
- Scene editor with 3D viewport
- Anchor types: horizontal surface, vertical surface, image, face, object
- Hierarchy Select — right-click on entity to reach individual nodes inside a USDZ hierarchy
- Behaviors panel — trigger + action card sequences
  - Triggers: Scene Start, Tap, Proximity, Notification
  - Actions: Show, Hide, Move To, Orbit, Play USDZ Animation, Change Scene, Apply Impulse, Look At Camera, Play Sound, Wait
  - Loop modes: play once, loop endlessly
  - Easing: Ease In, Ease Out, Ease In/Out, None
- Simulate button — live preview in editor
- Edit on iOS / Done — Mac ↔ iOS Handoff for real-world AR validation
- USDZ Export (new, enable in Preferences) — for web / News embedding
- Reality Export — for Xcode / AR Quick Look

**USD / USDZ format concepts**
- USD instancing — instances referencing a prototype geometry, exported with hierarchy intact
- PBR material model — base color, roughness, metallic, normal, ambient occlusion inputs
- USDZ — single-file ZIP archive containing USD + textures, compliant with Apple's AR Quick Look spec

**AR Quick Look**
- Opens `.usdz` and `.reality` files directly from Files app, Mail, Messages, Safari
- Splash/loading scene displayed while AR anchors

**RealityKit** (referenced for Xcode integration)
- `.reality` file loaded at runtime; sample code available in Xcode's RealityKit section

## Code Highlights

No runtime API code samples in this session (artist-focused tooling talk). The key integration point for developers is loading a `.reality` file exported from Reality Composer:

```swift
// In an Xcode project using RealityKit
let anchor = try! Experience.loadScene()  // generated by Reality Composer
arView.scene.anchors.append(anchor)
```

Splash scene pattern in Reality Composer (behavior configuration):
- Scene A (splash): zero behaviors except one Custom behavior: Trigger = "Scene Start" → Action = "Change Scene" → target Scene B
- Scene B (live): full behaviors with animations

## Takeaways
- Reality Converter is the recommended one-stop tool for converting FBX/OBJ/GLTF/USD assets to USDZ; always verify base units in the Properties panel to avoid 100× scale errors.
- Use Hierarchy Select in Reality Composer to target individual USD instances for per-entity behaviors rather than animating the whole group together.
- Chain Show → Move To → Orbit (around a hidden pivot) → Move To → Hide cards to create convincing looping ambient animations without any code.
- Export as `.reality` for Xcode / AR Quick Look integration; use the new USDZ export (enabled in Preferences) when embedding in websites or Apple News.

---
_Source: WWDC20 Session 10601 page (abstract, chapter summaries, code samples, and resource links)._
