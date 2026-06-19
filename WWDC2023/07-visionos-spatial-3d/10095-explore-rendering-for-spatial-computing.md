# Explore Rendering for Spatial Computing
**WWDC23 · Session 10095** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10095/)

_Platforms:_ visionOS 1

## Overview
This session covers key rendering considerations for building visionOS apps with RealityKit. It is organized around five topics: customizing image-based lighting (IBL) with `ImageBasedLightComponent`, adding grounding shadows with `GroundingShadowComponent`, understanding the available material types (including the new `ShaderGraphMaterial`), controlling tone mapping per material, and adapting content for two automatic system rendering optimizations — rasterization rate maps and dynamic content scaling.

The session emphasizes practical decisions developers must make to ensure their 3D content and UI look great on the visionOS displays, which use eye-tracking to selectively apply higher rendering resolution at the center of gaze.

## Key Topics

- **Image-based lighting (IBL)** — RealityKit combines an ARKit-provided environment probe texture (room-specific) with a system IBL texture (OS-provided). New in visionOS: override the system IBL by attaching `ImageBasedLightComponent` with a custom `EnvironmentResource`; pair with `ImageBasedLightReceiverComponent` on entities to make them receive the custom IBL.
- **Grounding shadows** — New `GroundingShadowComponent` with `castsShadow: true` adds realistic shadows that appear on 3D model surfaces and physical environment objects, helping users understand object depth and position.
- **Material types** — Four existing materials carry over from iOS/macOS: `PhysicallyBasedMaterial` (PBR, reacts to lighting), `SimpleMaterial` (subset of PBR, for prototyping), `UnlitMaterial` (constant appearance, no lighting), `VideoMaterial` (movie file on surface). New: `ShaderGraphMaterial` (MaterialX-based, authored in Reality Composer Pro — see session 10202).
- **Tone mapping** — RealityKit applies tone mapping to all material color outputs by default; this remaps bright values into the visible range for natural perceived colors. New `applyPostProcessToneMap` parameter on `UnlitMaterial` allows opting out; useful when exact color matching is needed between RealityKit 3D content and SwiftUI/UIKit elements (e.g., UI overlays referencing the same color constants).
- **Rasterization rate maps** — Automatic visionOS optimization driven by eye tracking; renders higher detail near the gaze point and lower detail in the periphery, saving GPU and memory resources. Developers must account for it: use larger mesh triangles and store fine details in opacity textures (mipmapped) rather than in geometry, to avoid flickering artifacts in the periphery.
- **Dynamic content scaling** — Automatic system technique for SwiftUI/UIKit content; rasterizes UI elements at variable resolution based on eye gaze proximity. Core Animation content can opt in via new `CALayer.wantsDynamicContentScaling = true`. Not recommended for primarily bitmap-based content.

## APIs & Frameworks

**RealityKit**
- `ImageBasedLightComponent` **[NEW for visionOS]** — component overriding system IBL; parameter: `source: .single(EnvironmentResource)`
- `ImageBasedLightReceiverComponent` **[NEW for visionOS]** — added to entities that should receive a custom IBL; parameter: `imageBasedLight: Entity`
- `EnvironmentResource` **[NEW]** — loads IBL texture from an asset bundle; `EnvironmentResource(named:)`
- `GroundingShadowComponent` **[NEW]** — casts shadows on surfaces and physical environment; property: `castsShadow: Bool`
- `ShaderGraphMaterial` **[NEW]** — MaterialX-based material, authored in Reality Composer Pro; new for visionOS
- `PhysicallyBasedMaterial` — existing PBR material; works on visionOS
- `SimpleMaterial` — existing simple PBR material; works on visionOS
- `UnlitMaterial` — existing unlit material; new property `applyPostProcessToneMap: Bool` **[NEW]** — set `false` to disable tone mapping; useful for matching UI element colors precisely
- `UnlitMaterial.init(color:applyPostProcessToneMap:)` **[NEW parameter]** — construct with tone mapping disabled
- `VideoMaterial` — existing; works on visionOS
- `ModelComponent` — existing; access via `entity.components[ModelComponent.self]`
- `Entity(named:in:)` — async loading from asset bundle
- `RealityView` — SwiftUI view hosting RealityKit content

**Core Animation**
- `CALayer.wantsDynamicContentScaling` **[NEW]** — Bool property; set to `true` to opt in to dynamic content scaling for Core Animation layers; not recommended for bitmap-heavy content

**Metal / System (referenced)**
- Rasterization rate maps — `MTLRasterizationRateMap` (existing Metal API); automatically applied by RealityKit; reference: "Rendering at Different Rasterization Rates" article on developer.apple.com

## Code Highlights

Custom image-based lighting:
```swift
RealityView { content in
    async let satellite = Entity(named: "Satellite", in: worldAssetsBundle)
    async let environment = EnvironmentResource(named: "Sunlight")

    if let satellite = try? await satellite, let environment = try? await environment {
        content.add(satellite)
        satellite.components.set(ImageBasedLightComponent(source: .single(environment)))
        satellite.components.set(ImageBasedLightReceiverComponent(imageBasedLight: satellite))
    }
}
```

Grounding shadow:
```swift
RealityView { content in
    if let vase = try? await Entity(named: "flower_tulip") {
        content.add(vase)
        vase.components.set(GroundingShadowComponent(castsShadow: true))
    }
}
```

Disable tone mapping for exact color matching:
```swift
RealityView { content in
    if let trafficLight = try? await Entity(named: "traffic_light") {
        content.add(trafficLight)
        if let lamp = trafficLight.findEntity(named: "red_light"),
           var model = lamp.components[ModelComponent.self] {
            let material = UnlitMaterial(color: .init(color), applyPostProcessToneMap: false)
            model.materials = [material]
            lamp.components[ModelComponent.self] = model
        }
    }
}
```

Enable dynamic content scaling for CALayer:
```swift
myLayer.wantsDynamicContentScaling = true
```

## Takeaways

- Use `ImageBasedLightComponent` with a custom `EnvironmentResource` to replace the system IBL and match the lighting of your immersive experience; add `ImageBasedLightReceiverComponent` to each entity that should receive it.
- Add `GroundingShadowComponent(castsShadow: true)` to any RealityKit entity that needs grounding shadows — a single component addition that works on both virtual surfaces and physical environment objects.
- Set `applyPostProcessToneMap: false` on `UnlitMaterial` when your 3D content must precisely match SwiftUI or UIKit element colors; tone mapping is enabled by default and will otherwise cause a visible color discrepancy.
- For 3D content visible in the visual periphery, use larger mesh triangles and store fine details in mip-mapped opacity textures rather than mesh geometry to prevent flickering artifacts caused by the automatic rasterization rate map optimization.

---
_Source: WWDC23 Session 10095 page (abstract, chapter summaries, code samples, and resource links)._
