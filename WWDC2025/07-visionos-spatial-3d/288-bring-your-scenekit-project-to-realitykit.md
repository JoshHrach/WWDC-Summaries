# Bring Your SceneKit Project to RealityKit
**WWDC25 · Session 288** · [Watch](https://developer.apple.com/videos/play/wwdc2025/288/)

_Platforms:_ iOS, macOS, iPadOS, visionOS, tvOS (RealityKit); tvOS (NEW platform support)

## Overview
Apple officially announced that SceneKit is entering soft deprecation — it remains functional and will receive critical bug fixes, but no new features will be added. This session provides a practical migration guide from SceneKit to RealityKit, covering the Entity Component System (ECS) mental model, asset conversion, animation migration, audio replacement, lighting and particle effects, and the new post-processing API. RealityKit now also supports tvOS, expanding its reach to all Apple platforms.

The session is structured as a side-by-side comparison: for each SceneKit concept, it maps the direct RealityKit equivalent and highlights any capability gaps (where the developer may need to re-implement logic).

## Key Topics

### SceneKit Deprecation
SceneKit is in "maintenance mode" — no new features, security and crash fixes only. Developers are encouraged to migrate to RealityKit for future development. New tvOS support in RealityKit eliminates the last platform gap.

### ECS Mental Model
RealityKit uses an Entity Component System. `Entity` replaces `SCNNode`; components replace node properties. Logic moves from subclasses or delegate methods into `System` types that iterate entities per frame.

### View Replacement
`RealityView` replaces `SCNView` and `ARSCNView`. Assets are loaded with `Entity.load(named:)` or Reality Composer Pro, replacing `SCNScene(named:)`.

### Asset Conversion
The recommended pipeline exports SceneKit assets to USD (Universal Scene Description) and imports them into Reality Composer Pro. The CLI tool `xcrun scntool --convert` (new in Xcode 26) handles conversion, including the new `--append-animation` flag for merging baked animation files without losing existing content.

### Animations
SceneKit's animation system maps to `AnimationLibraryComponent`. Named animations stored in the library are accessed by key, then played via `AnimationPlaybackController`. The `--append-animation` flag in `scntool` enables incremental animation file updates.

### Audio
`AudioLibraryComponent` stores named audio resources. `AmbientAudioComponent` replaces ambient audio nodes.

### Lighting
`DirectionalLightComponent` replaces `SCNLight`. Shadow configuration is done via `DirectionalLightComponent.Shadow` with properties for mode, bias, and cascade count.

### Particles
`ParticleEmitterComponent` replaces `SCNParticleSystem`. Particle properties map closely, with configuration done in Reality Composer Pro or in code.

### Post-Processing
The **[NEW]** `PostProcessEffect` protocol enables custom post-processing via Metal Performance Shaders or custom kernels. Conforming types implement a single method receiving a `PostProcessEffectContext` (containing source texture, output texture, command buffer). Built-in MPS operations (`MPSImageThresholdToZero`, `MPSImageGaussianBlur`, `MPSImageAdd`) compose filter chains. Effects are attached via `RealityView.renderingEffects.customPostProcessing`.

## APIs & Frameworks

**RealityKit**
- `Entity` — replaces `SCNNode`
- `Component` protocol — replaces node properties
- `System` protocol — per-frame logic, replaces SCNSceneRendererDelegate
- `RealityView` — replaces `SCNView` / `ARSCNView`
- `Entity.load(named:)` — scene loading
- `AnimationLibraryComponent` — stores named animations
- `AnimationPlaybackController` — play, pause, stop animations
- `AudioLibraryComponent` — named audio resources
- `AmbientAudioComponent` — ambient audio replacement
- `DirectionalLightComponent` — directional light
- `DirectionalLightComponent.Shadow` — shadow configuration
- `ParticleEmitterComponent` — particle effects
- **[NEW]** `PostProcessEffect` protocol — custom post-processing
- **[NEW]** `PostProcessEffectContext` — input/output textures and command buffer
- **[NEW]** `RealityView.renderingEffects.customPostProcessing` — attach effect
- **[NEW]** tvOS platform support for RealityKit

**Metal Performance Shaders (used in post-processing)**
- `MPSImageThresholdToZero` — threshold filter
- `MPSImageGaussianBlur` — blur filter
- `MPSImageAdd` — additive composition

**Xcode CLI**
- **[NEW]** `xcrun scntool --convert` — convert SceneKit scene to USD
- **[NEW]** `xcrun scntool --convert --append-animation` — append animation to existing USD

**Asset Formats**
- USD / USDZ — Universal Scene Description; RealityKit's native asset format
- Reality Composer Pro — visual editing tool for USD assets and component authoring

## Code Highlights
Attach a post-processing effect in RealityKit:
```swift
struct ThresholdEffect: PostProcessEffect {
    func render(context: PostProcessEffectContext) {
        let threshold = MPSImageThresholdToZero(device: context.device, thresholdValue: 0.5, linearGrayColorTransform: nil)
        threshold.encode(commandBuffer: context.commandBuffer, sourceTexture: context.sourceColorTexture, destinationTexture: context.destinationColorTexture)
    }
}
// In RealityView:
.renderingEffects.customPostProcessing(ThresholdEffect())
```

Convert SceneKit scene and append animation:
```sh
xcrun scntool --convert Scene.scn --output Scene.usdz
xcrun scntool --convert Walk.dae --append-animation --output Scene.usdz
```

## Takeaways
- SceneKit is in soft deprecation; start new features in RealityKit and plan a migration roadmap for existing SceneKit code.
- Use `xcrun scntool --convert --append-animation` to batch-migrate baked SceneKit animations into USD assets incrementally.
- Adopt `AnimationLibraryComponent` and `AudioLibraryComponent` to replicate SceneKit's named-resource patterns.
- Use `PostProcessEffect` + Metal Performance Shaders for full-screen effects previously achieved via SCNTechnique or Core Image in SceneKit.

---
_Source: WWDC25 Session 288 page (abstract, chapter summaries, code samples, and resource links)._
