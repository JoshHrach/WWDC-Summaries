# Dive into RealityKit 2
**WWDC21 · Session 10074** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10074/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
RealityKit 2 introduces a sweeping set of enhancements to Apple's AR authoring framework, focusing on developer control over appearance and behavior. The session uses an immersive underwater demo app to illustrate five major feature areas: a more complete Entity Component System with custom user-defined systems, new material types including PhysicallyBasedMaterial and CustomMaterial, an expanded animation pipeline, a new character controller, and runtime resource generation for meshes and audio.

The ECS improvements are the architectural centerpiece. Developers can now define their own `System` subclasses with explicit dependency ordering, perform efficient `EntityQuery` operations, and use `TransientComponent` to prevent certain state from being cloned or inherited. The session shows how this shift moves logic out of Game Manager classes and monolithic entity subclasses and into cleanly separated, ordered system update functions.

The session pairs with "Explore advanced rendering with RealityKit 2" (Session 10075), which covers the custom shader, geometry modifier, and post-processing capabilities used in the same demo.

## Key Topics
- **Custom ECS Systems:** `RealityKit.System` protocol with `init(scene:)`, `update(context:)`, `dependencies` (`.before`, `.after`), and `EntityQuery` for efficient per-frame entity selection. Replaces `SceneEvents.update` closure-based patterns.
- **Entity & Component Architecture:** Components as pure state structs; entities as identifiers; `TransientComponent` for non-inherited state; `storeWhileEntityActive` for automatic subscription lifecycle management.
- **Material Enhancements:** `PhysicallyBasedMaterial` (superset of SimpleMaterial, matches USD PBR schema) with properties for normal map, opacity/threshold, ambient occlusion, clearcoat. `CustomMaterial` for Metal-backed shaders. `VideoMaterial` gains transparency support.
- **Animation Pipeline:** Blend layers with individual blend factors and playback speeds; `FromToByAnimation` for procedurally created transform animations; `AnimationView` for slicing a single USD timeline into named clips; transition duration blending between clips.
- **Character Controller:** Capsule-based physics character that automatically interacts with AR mesh colliders (LiDAR mesh). `move(to:)` for obstacle-aware movement; `teleport` for obstacle-ignoring placement.
- **Runtime Resource Generation:** `SceneUnderstandingComponent.entityType` (.face / .meshChunk) for face mesh entities; `TextureResource.generate(from:CGImage)` for runtime texture creation; `AudioBufferResource` from `AVAudioBuffer` (e.g., `AVSpeechSynthesizer`) with `.spatial`, `.nonSpatial`, `.ambient` input modes.

## APIs & Frameworks

**RealityKit**
- `RealityKit.System` protocol **[NEW]** – Custom per-frame system with `init(scene:)` and `update(context: SceneUpdateContext)`
- `SystemDependency` **[NEW]** – `.before(SystemType.self)`, `.after(SystemType.self)`
- `SceneUpdateContext` **[NEW]** – Provides `deltaTime` and `scene` reference in system `update`
- `EntityQuery` **[NEW]** – `EntityQuery(where: .has(ComponentType.self) && .has(OtherComponent.self))`
- `scene.performQuery(_:)` **[NEW]** – Executes entity query, returns iterable result
- `TransientComponent` protocol **[NEW]** – Marks components not inherited on entity clone
- `Cancellable.storeWhileEntityActive(_:)` **[NEW]** – Auto-unsubscribes event when entity deactivates
- `PhysicallyBasedMaterial` **[NEW]** – Full PBR material: `baseColor`, `roughness`, `metallic`, `normal`, `opacity`, `opacityThreshold`, `ambientOcclusion`, `clearcoat`, `blending`
- `CustomMaterial` **[NEW]** – Metal-backed surface shader and geometry modifier support
- `VideoMaterial` – Gains transparency support **[NEW]**
- `PhysicallyBasedMaterial.Texture` **[NEW]**
- `AnimationResource` – Existing type; now loadable per-clip or via `AnimationView`
- `AnimationView` **[NEW]** – Slices a USD timeline into named clips by timecode
- `FromToByAnimation` **[NEW]** – Procedurally create transform/position/rotation animations
- Blend layers API **[NEW]** – Per-layer blend factor and playback speed control
- `CharacterControllerComponent` **[NEW]** – Capsule-based character physics with `height` and `radius`
- `Entity.move(to:)` **[NEW]** – Obstacle-aware character movement
- `Entity.teleport(to:)` **[NEW]** – Obstacle-ignoring placement
- `SceneUnderstandingComponent.entityType` **[NEW]** – `.face` or `.meshChunk`
- `TextureResource.generate(from: CGImage, options:)` **[NEW]** – Runtime texture from CGImage
- `AudioBufferResource(buffer: AVAudioBuffer, inputMode:, shouldLoop:)` **[NEW]** – Runtime audio resource
- `AudioBufferResource.InputMode` **[NEW]** – `.spatial`, `.nonSpatial`, `.ambient`
- `Entity.playAudio(_:)` – Plays `AudioBufferResource` with 3D positional audio
- `ModelComponent` – Holds mesh and materials; accessible on face entities
- `Entity.components[ComponentType.self]` – Component get/set

**ARKit**
- LiDAR mesh / scene reconstruction (used by `CharacterControllerComponent`)

**AVFoundation**
- `AVSpeechSynthesizer`, `AVSpeechUtterance`, `AVSpeechSynthesisVoice`
- `AVAudioBuffer`

**PencilKit**
- `PKCanvasView` – Used to draw textures applied to face mesh

## Code Highlights
Custom System with dependency ordering and entity query:
```swift
class FlockingSystem: RealityKit.System {
    required init(scene: RealityKit.Scene) { }
    static var dependencies: [SystemDependency] { [.before(MotionSystem.self)] }
    private static let query = EntityQuery(where: .has(FlockingComponent.self)
                                                && .has(MotionComponent.self))
    func update(context: SceneUpdateContext) {
        context.scene.performQuery(Self.query).forEach { entity in
            guard var motion = entity.components[MotionComponent.self] else { return }
            motion.forces.append(/* separation, cohesion, alignment */)
            entity.components[MotionComponent.self] = motion
        }
    }
}
```

Runtime audio from speech synthesis:
```swift
synthesizer.write(utterance) { audioBuffer in
    guard let audioResource = try? AudioBufferResource(
        buffer: audioBuffer, inputMode: .spatial, shouldLoop: true)
    else { return }
    entity.playAudio(audioResource)
}
```

## Takeaways
- Custom `System` types with `EntityQuery` and explicit dependency ordering replace closure-based `SceneEvents.update` handlers, enabling clean separation of concerns and predictable execution order.
- `PhysicallyBasedMaterial` aligns RealityKit materials with USD PBR, making it straightforward to load USD assets and tweak individual material properties at runtime.
- `CharacterControllerComponent` removes the need for manual physics character implementation by automatically resolving collisions with the LiDAR-reconstructed AR mesh.
- `AudioBufferResource` and `TextureResource.generate(from:CGImage)` open the door to fully procedural, generative AR experiences without pre-baked asset files.

---
_Source: WWDC21 Session 10074 page (abstract, chapter summaries, code samples, and resource links)._
