# Create Custom Environments for Your Immersive Apps in visionOS
**WWDC24 · Session 10087** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10087/)

_Platforms:_ visionOS 2

## Overview
visionOS 2 adds environment authoring tools directly to Reality Composer Pro, letting developers build photorealistic or stylized surroundings for their fully immersive apps without leaving Apple's toolchain. This session covers the new environment asset workflow, how to set up ground planes and skyboxes, how to light the scene using Image-Based Lighting (IBL) from an HDRI, and how to configure occlusion and grounding shadows so virtual objects feel anchored to the environment.

The talk also explains how environments interact with passthrough content and dynamic world mapping, and introduces the concept of environment presets — authored environments that apps can switch between at runtime.

## Key Topics
- **Environment authoring in Reality Composer Pro** — new authoring tools for ground planes, skyboxes, and IBL lighting integrated into the existing RCP workflow.
- **HDRI-based Image-Based Lighting** — importing equirectangular HDR images as the scene's light source; IBL drives both diffuse and specular lighting on all RealityKit objects in the scene.
- **Ground plane and occlusion** — configuring a virtual ground plane with a grounding shadow receiver so virtual objects cast shadows, improving the sense of presence.
- **Skybox mesh** — a large sphere or cube mesh mapped with the HDRI environment texture to provide the visual surround.
- **Environment presets** — packaging multiple environments as named presets; switching at runtime via RealityKit API.
- **Passthrough blending** — how fully immersive apps can blend between the authored environment and passthrough video.

## APIs & Frameworks

**RealityKit**
- `ImmersiveSpace` / `RealityView` — unchanged entry points for immersive content
- **[NEW]** `EnvironmentResource` — runtime representation of a baked environment asset authored in Reality Composer Pro
- **[NEW]** `ImageBasedLightComponent` — attach to an entity to apply IBL from an `EnvironmentResource`; replaces manual `DirectionalLight` / `PointLight` setups for environment-lit scenes
- **[NEW]** `ImageBasedLightReceiverComponent` — attach to entities that should receive IBL; controls IBL intensity on a per-entity basis
- `GroundingShadowComponent` — existing component; place on virtual objects to receive grounding shadows cast onto the ground plane
- `OcclusionMaterial` — used on the ground plane mesh to make it invisible but shadow-receiving
- `ModelComponent` / `MeshResource` — unchanged; used for the environment geometry (skybox sphere, ground plane)
- `Entity.load(named:in:)` — load environment assets from the app bundle
- `RealityView` `update` closure — used to swap `EnvironmentResource` at runtime for preset switching

**Reality Composer Pro (editor)**
- New "Environment" template — pre-configured scene with skybox sphere, ground plane, IBL slot
- HDRI import workflow — drag `.hdr` / `.exr` file into the asset browser; assign to IBL slot
- Environment baking — new bake action computes diffuse and specular IBL maps from the HDRI

**SwiftUI / visionOS**
- `ImmersiveSpace(id:)` — unchanged; hosts the `RealityView` for full immersion
- `preferredSurroundingsEffect(_:)` — **[NEW]** modifier that blends the system environment with the app's custom environment; accepts a `SurroundingsEffect`

## Code Highlights
Load and apply a baked environment at runtime:

```swift
RealityView { content in
    if let env = try? await EnvironmentResource(named: "ForestEnvironment") {
        let iblEntity = Entity()
        iblEntity.components.set(ImageBasedLightComponent(source: .single(env)))
        content.add(iblEntity)
    }
}
```

Switch between environment presets in response to a user action:

```swift
@State var currentEnv: String = "Day"

RealityView { content in … } update: { content in
    if let entity = content.entities.first(where: { $0.name == "IBLEntity" }),
       let env = try? await EnvironmentResource(named: currentEnv) {
        entity.components.set(ImageBasedLightComponent(source: .single(env)))
    }
}
```

## Takeaways
- Use the Reality Composer Pro environment template as the starting point — it sets up the skybox sphere, ground plane, and IBL slot with the correct transforms and materials automatically.
- `ImageBasedLightComponent` + `ImageBasedLightReceiverComponent` gives physically correct lighting on all objects in the scene without manual light placement.
- Bake the environment in Reality Composer Pro before shipping; runtime IBL from a raw HDRI is supported but more expensive.
- Add `GroundingShadowComponent` to every significant virtual object so it reads as grounded in the environment even when the ground plane itself is occluded.

---
_Source: WWDC24 Session 10087 page (abstract, chapter summaries, code samples, and resource links)._
