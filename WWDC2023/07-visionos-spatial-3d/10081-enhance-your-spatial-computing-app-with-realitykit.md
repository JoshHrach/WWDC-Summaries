# Enhance Your Spatial Computing App with RealityKit
**WWDC23 · Session 10081** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10081/)

_Platforms:_ visionOS 1

## Overview
This session builds on the foundational RealityKit session ("Build spatial experiences with RealityKit") to explore advanced features that make spatial apps more immersive and dynamic. It covers five new capabilities: RealityView attachments for embedding SwiftUI views in entity hierarchies, `VideoPlayerComponent` for in-scene video playback, portals for rendering alternate world content through mesh surfaces, `ParticleEmitterComponent` for visual effects, and anchors for attaching 3D content to real-world surfaces.

The session uses a solar system/tides example app to demonstrate each feature in progressive layers. Attachments bridge SwiftUI and RealityKit so labels and panels can live alongside 3D models. Portals create a "magic window" effect using `WorldComponent` and `PortalComponent` together. Particle emitters, authored in Reality Composer Pro, can be driven at runtime via custom `System` implementations. Anchors placed in an `ImmersiveSpace` enable content to adhere to physical walls, floors, or head/hand locations.

## Key Topics

- **RealityView attachments** — Two-part setup: `attachments` view builder declaring tagged SwiftUI views; `attachments.entity(for:)` in the `make` closure to retrieve views as `ViewAttachmentEntity`; positioning and parenting attachment entities in the scene; update closure responds to SwiftUI state changes.
- **Video playback** — `VideoPlayerComponent` wraps `AVPlayer`; automatically generates aspect-ratio-correct rectangular mesh; supports 2D and MV-HEVC 3D video; auto-displays AVPlayer captions; `isPassthroughTintingEnabled` for TV-app-style ambient tinting; `VideoPlayerEvents` subscription for property change callbacks.
- **Portals** — `WorldComponent` marks an entity subtree as a separate world; `PortalMaterial` applied to a mesh entity; `PortalComponent(target:)` links a mesh to a world; entities inside the world are only visible through the portal mesh; `ImageBasedLightComponent` applies separate lighting inside the world.
- **Particle emitters** — `ParticleEmitterComponent` authored in Reality Composer Pro or created at runtime; custom `System` subclass with `EntityQuery` for batch updates; runtime-adjustable properties: `lifeSpan`, `vortexStrength`, and more.
- **Anchors** — `AnchorEntity` with surface specification (`.plane(.vertical, classification: .wall, minimumBounds:)`); tracking modes `.continuous` and `.once`; `AnchoredStateChanged` event; anchor transforms are private — use ARKit for explicit transform access.

## APIs & Frameworks

**RealityKit**
- `RealityView` — primary SwiftUI/RealityKit bridge view
- `RealityView` `attachments:` view builder parameter **[NEW]** — declares `Attachment(id:content:)` views
- `Attachment(id:content:)` **[NEW]** — wraps a SwiftUI view with a tag for use as an entity
- `RealityViewAttachments.entity(for:)` **[NEW]** — retrieves a `ViewAttachmentEntity` by tag
- `ViewAttachmentEntity` **[NEW]** — entity type representing an embedded SwiftUI view
- `VideoPlayerComponent` **[NEW]** — RealityKit component embedding `AVPlayer` in a 3D scene; auto-generates mesh
- `VideoPlayerComponent.isPassthroughTintingEnabled` **[NEW]** — enables ambient passthrough color matching
- `VideoPlayerEvents.VideoSizeDidChange` **[NEW]** — event type for video property changes
- `WorldComponent` **[NEW]** — marks an entity subtree as a separate rendering world
- `PortalComponent` **[NEW]** — links a mesh entity to a world; `init(target: Entity)`
- `PortalMaterial` **[NEW]** — material type that makes a mesh act as a portal window
- `ParticleEmitterComponent` **[NEW]** — component for particle visual effects (snow, sparks, etc.)
- `ParticleEmitterComponent.mainEmitter.lifeSpan` **[NEW]** — particle lifetime
- `ParticleEmitterComponent.mainEmitter.vortexStrength` **[NEW]** — swirl intensity
- `AnchorEntity` **[NEW on visionOS]** — anchors entity subtrees to real-world surfaces or body parts
- `AnchorEntity(.plane(.vertical, classification:minimumBounds:))` — wall/floor anchor specification
- `AnchoredStateChanged` event — notification when entity becomes anchored
- `AnchoringComponent.TrackingMode.continuous` / `.once` — anchor update frequency
- `EntityQuery` — queries scene for entities matching component criteria
- `System` protocol — custom per-frame update logic
- `SceneUpdateContext` — context passed to `System.update(context:)`
- `ImageBasedLightComponent` — applies IBL environment to an entity subtree
- `ImageBasedLightReceiverComponent` — marks entities to receive IBL
- `EnvironmentResource` — loadable environment/IBL asset
- `ModelComponent` — mesh + materials component
- `MeshResource.generatePlane(width:height:cornerRadius:)` — generates flat mesh
- `Entity.load(named:)` / `Entity(named:)` — async entity loading
- `ImmersiveSpace` **[NEW]** — SwiftUI scene type for full-room spatial content
- `.immersionStyle(selection:in:)` modifier **[NEW]** — e.g. `.mixed`

**AVFoundation**
- `AVPlayer` — drives `VideoPlayerComponent`
- `AVPlayerItem` — wraps video URL asset
- `AVURLAsset` — URL-based video asset

## Code Highlights

RealityView with attachments:
```swift
RealityView { content, attachments in
    let earth = try! await Entity(named: "Earth")
    content.add(earth)
    if let label = attachments.entity(for: "earth_label") {
        label.position = [0, -0.15, 0]
        earth.addChild(label)
    }
} attachments: {
    Attachment(id: "earth_label") { Text("Earth") }
}
```

VideoPlayerComponent:
```swift
let player = AVPlayer()
entity.components[VideoPlayerComponent.self] = .init(avPlayer: player)
entity.scale *= 0.4
player.replaceCurrentItem(with: playerItem)
player.play()
```

Portal setup:
```swift
let world = Entity()
world.components[WorldComponent.self] = .init()

let portal = Entity()
portal.components[ModelComponent.self] = .init(
    mesh: .generatePlane(width: 1, height: 1, cornerRadius: 0.5),
    materials: [PortalMaterial()])
portal.components[PortalComponent.self] = .init(target: world)
```

Wall anchor in ImmersiveSpace:
```swift
ImmersiveSpace {
    RealityView { content in
        let anchor = AnchorEntity(.plane(.vertical, classification: .wall, minimumBounds: [1, 1]))
        anchor.addChild(makePortal())
        content.add(anchor)
    }
}
.immersionStyle(selection: $style, in: .mixed)
```

## Takeaways

- Use `RealityView` attachments to embed SwiftUI labels and panels alongside 3D content without leaving the RealityKit entity hierarchy.
- `VideoPlayerComponent` is the correct way to embed video in a RealityKit scene — it handles mesh generation, captions, and passthrough tinting automatically.
- Portals require pairing `WorldComponent` (marks the world subtree), `PortalMaterial` (on the mesh), and `PortalComponent(target:)` (links them); entities in the world are only visible through the mesh.
- Anchors work only inside `ImmersiveSpace`; use ARKit if you need the actual anchor transform — RealityKit hides it for privacy.

---
_Source: WWDC23 Session 10081 page (abstract, chapter summaries, code samples, and resource links)._
