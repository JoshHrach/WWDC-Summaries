# Iterate your spatial scenes faster with Reality Composer Pro 3
**WWDC26 · Session 280** · [Watch](https://developer.apple.com/videos/play/wwdc2026/280/)

_Platforms:_ visionOS 27, macOS 27

## Overview
This session introduces Reality Composer Pro 3 (RCP3) as a standalone, redesigned tool for building spatial experiences faster through a tighter iteration loop. It is now a free standalone download from developer.apple.com, no longer bundled only with Xcode. The session uses the "Chaparral Village" sample project as the running demo throughout.

The core workflow improvements center on four areas: the entity-component model for scene composition (including the new Compute Graph for GPU simulations), the Prototypes and Instances system for reusable content, direct Live Preview to a connected Apple Vision Pro for in-headset validation without a build cycle, and the Lightmaps baking tool for high-fidelity static lighting. The session closes with a preview of the Reality Composer Pro Assistant — an on-device generative AI model that creates 3D objects and materials from natural language prompts directly inside the editor.

## Key Topics

### Overview and Standalone Download
- RCP3 is now a standalone macOS app, downloadable from developer.apple.com — no Xcode install required.
- Designed for rapid, iterative scene authoring for both developers and artists.
- "Chaparral Village" sample project demonstrates all features covered.

### Entities and Components
- Scenes are built by adding, nesting, and configuring entities and components in a hierarchy panel.
- Components include all standard RealityKit types plus custom types registered via the RCP3 plugin system (Session 281).
- **Compute Graph** component **[NEW]**: attach to an entity for GPU-based particle/physics simulation; configured visually in the Compute Graph editor.

### Prototypes and Instances
- Any entity in the scene can be promoted to a **Prototype** — a reusable template.
- Instances inherit all prototype properties; individual instance overrides can be set without breaking the source.
- Overrides propagate or are reverted per-property; useful for populating large scenes (e.g., many similar buildings) with minimal asset duplication.

### Live Preview
- Target a connected Apple Vision Pro from the Preview toolbar.
- Scene changes made in RCP3 are reflected in the headset in real time — no build, sign, or deploy step required.
- Dramatically shortens the iteration loop for spatial composition and material tuning.

### Lightmaps
- New **Lightmaps** baking tool **[NEW]**: select static geometry, configure quality/resolution, bake indirect lighting, ambient occlusion, and a beauty map.
- Baked results are stored as texture maps applied to scene geometry at runtime.
- Improves visual fidelity of static scenes without the runtime cost of dynamic global illumination.
- Output lightmap textures are compatible with `ShaderGraphMaterial` in RealityKit.

### Reality Composer Pro Assistant
- New AI assistant **[NEW]** embedded in RCP3.
- Accepts natural language prompts to generate 3D objects and materials on demand.
- Uses on-device generative models; generated assets are placed directly into the scene.
- Speeds up prototyping when no source asset exists yet.

## APIs & Frameworks

### Reality Composer Pro 3 Editor Features (NEW)
- **Standalone app**: separate download, not bundled with Xcode
- **Compute Graph component** **[NEW]**: GPU particle simulation authored visually
- **Prototypes and Instances** system **[NEW]**: reusable entity templates with per-instance overrides
- **Live Preview** to connected Apple Vision Pro **[NEW]**: real-time spatial preview without building an app
- **Lightmaps baking** **[NEW]**: indirect lighting, ambient occlusion, beauty map baking for static geometry
- **Reality Composer Pro Assistant** **[NEW]**: generative AI for 3D objects and materials from natural language

### RealityKit (runtime counterparts)
- Lightmap-baked textures consumed via `ShaderGraphMaterial` — existing
- `ModelComponent`, `Entity`, `EntityComponentQuery` — existing
- `NavigationMeshResource`, `NavigationComponent` **[NEW]** — configured via RCP3 navigation mesh component

### Plugin System
- RCP3 loads custom components, systems, and actions from Xcode-compiled plugins (see Session 281)
- `RealityComposerProPlugin`, `RealityComposerProContext` — for registering custom types

## Code Highlights

_This session is primarily editor-workflow focused; no dedicated Swift code samples appear on the session page. Key patterns are in Session 281 (plugins) and Session 393 (visual tools)._

Use baked lightmaps: Simply bake in RCP3's Lightmaps panel; the output textures are automatically applied to the scene USDZ and rendered via `ShaderGraphMaterial` at runtime with no additional Swift code.

Live Preview activation: Connect Apple Vision Pro to Mac, then click the Preview target selector in RCP3's toolbar and choose the headset — the scene streams live.

## Takeaways
- Reality Composer Pro 3 as a standalone download removes friction for artists who do not use Xcode, enabling a broader design team to participate in spatial content creation.
- Live Preview eliminates the most painful part of spatial iteration — the build-deploy-validate cycle — by streaming scene changes directly to the headset.
- The Prototypes and Instances system enables efficient large-scene authoring without asset bloat, similar to prefab systems in traditional game engines.
- The AI assistant lowers the 3D content creation barrier further, letting designers rapidly prototype with generated assets before replacing them with production-quality models.

---
_Source: WWDC26 Session 280 page (abstract, chapter summaries, and resource links)._
