# Building Apps with RealityKit
**WWDC19 · Session 605** · [Watch](https://developer.apple.com/videos/play/wwdc2019/605/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
This applied session walks through building a complete AR card-matching game ("Memory Cards") using RealityKit, Apple's new high-level AR rendering framework introduced at WWDC19. It covers the four core building blocks of every RealityKit app — `ARView`, `Scene`, `AnchorEntity`, and `Entity` — and demonstrates each major feature: asset loading (synchronous and asynchronous), hit testing, transform animation, occlusion materials, custom components and entities, and built-in multiplayer synchronization via `MultipeerConnectivityService`.

The session is structured as a four-stage development walkthrough: prototype with basic interaction → polish with async loading and occlusion → custom state with the entity-component system → multiplayer with ARKit 3 Collaborative Session. It serves as a companion to "Introducing RealityKit and Reality Composer" (Session 603).

## Key Topics

**Core RealityKit Building Blocks**
- `ARView` — the main view and entry point; placed in the app's view hierarchy
- `Scene` — owned by `ARView`; holds all virtual content
- `AnchorEntity` — connects virtual content to real-world targets (planes, faces, images, objects, body anchors)
- `Entity` — base building block for virtual content

**Anchoring**
- `AnchorEntity(.plane(.horizontal, classification: .any, minimumBounds:))` — anchor to horizontal surfaces
- Supports all ARKit 3 anchor types: planes, faces, images, objects, body anchors **[NEW]**
- `ARView.scene.addAnchor(_:)` — adds anchor; tracking begins automatically

**Asset Loading**
- `Entity.loadModel(named:)` — synchronous load; blocks main thread (suitable only for small assets)
- `Entity.loadModelAsync(named:)` — asynchronous load using Combine; returns `LoadRequest<ModelEntity>`
- Multiple async loads combined with `.collect()` and `.sink` (Combine) — receive callback when all assets finish
- `Entity.clone(recursive:)` — creates an identical copy sharing the same GPU resources; does not reflect later mutations to the original

**Hit Testing**
- `ARView.entity(at:)` — returns entity closest to camera at a screen point
- `ARView.entities(at:)` — returns all entities intersected by the ray
- Entities must have a collision shape to be hit-testable
- `Entity.generateCollisionShapes(recursive:)` — auto-generates box collision shapes from visual bounds; inherited by clones

**Animation**
- `Entity.move(to:relativeTo:duration:timingFunction:)` — transform animation returning `AnimationPlaybackController`
- Timing functions: `.linear`, `.easeIn`, `.easeOut`, `.easeInOut`, cubic Bézier
- `AnimationPlaybackController` — pause, resume, stop, or observe completion

**Occlusion Materials**
- `OcclusionMaterial` — invisible material that masks virtual content behind it, revealing camera passthrough
- Used to simulate real-world occluders (tables, walls)
- `ModelEntity(mesh: .generateBox(size:), materials: [OcclusionMaterial()])` — create occlusion geometry
- Box occluder preferred over plane occluder for viewing from any angle

**Custom Components and Entities**
- `Component` protocol — Swift struct conforming to `Component` (and optionally `Codable`) attached to any entity
- `Entity.components[MyComponent.self]` — get/set components by type
- Custom entity subclasses: declare class inheriting `Entity`, adopt `HasModel`, `HasCollision`, and other `Has*` protocols for built-in component accessors
- Custom entities enable encapsulation of multi-component state changes (e.g., `reveal()` / `hide()` methods)

**Multiplayer**
- `MultipeerConnectivityService` **[NEW]** — RealityKit wrapper around `MCSession`; assigned to `Scene.synchronizationService`
- Automatic scene synchronization: entity hierarchy changes, component changes (if `Codable`), animations propagate automatically
- `ARWorldTrackingConfiguration.isCollaborationEnabled = true` **[NEW ARKit 3]** — shared world map across peers
- Synchronized anchors: create `ARAnchor` → add to `ARSession` → wrap with `AnchorEntity(anchor:)` to share world position
- Ownership model: entity has one owner at a time; owner's changes broadcast to peers; `Entity.requestOwnership(completionHandler:)` requests transfer
- `SynchronizationComponent.ownershipTransferMode` — `.autoAccept` or `.manual` (denies all requests)
- Local-only entities: `Entity.components[SynchronizationComponent.self] = nil` — entity not shared; ideal for per-player UI (selection indicators, private state)
- `ARView.scene.raycast(from:allowing:alignment:)` — raycast against real world for collaborative anchor placement

## APIs & Frameworks

### RealityKit (NEW)
- `ARView` **[NEW]**
- `Scene` **[NEW]**; `Scene.addAnchor(_:)`, `Scene.synchronizationService`
- `AnchorEntity` **[NEW]**; `.plane`, `.face`, `.image`, `.object`, `.body` target types
- `Entity` **[NEW]**; `.components`, `.children`, `.position`, `.clone(recursive:)`
- `ModelEntity` **[NEW]**; `HasModel`, `HasCollision`, `HasPhysicsBody` protocols
- `Entity.loadModel(named:)` **[NEW]** — synchronous
- `Entity.loadModelAsync(named:)` **[NEW]** — async, returns `LoadRequest<ModelEntity>` (Combine Publisher)
- `LoadRequest` **[NEW]** — Combine publisher for async asset loading; composable via `.collect()` + `.sink`
- `Entity.generateCollisionShapes(recursive:)` **[NEW]**
- `Entity.move(to:relativeTo:duration:timingFunction:)` **[NEW]**; `AnimationPlaybackController`
- `AnimationTimingFunction` **[NEW]**: `.linear`, `.easeIn`, `.easeOut`, `.easeInOut`
- `OcclusionMaterial` **[NEW]**
- `Component` protocol **[NEW]** — custom Swift struct components
- `SynchronizationComponent` **[NEW]**; `ownershipTransferMode: .autoAccept | .manual`
- `Entity.requestOwnership(completionHandler:)` **[NEW]**
- `MultipeerConnectivityService` **[NEW]** — wraps `MCSession` for scene sync

### ARKit 3 (referenced)
- `ARWorldTrackingConfiguration.isCollaborationEnabled` **[NEW]** — collaborative world map
- `ARSession.add(anchor:)` — add `ARAnchor` for synchronized world placement
- `ARView.scene.raycast(from:allowing:alignment:)` — real-world raycast for host anchor placement

### Combine (referenced)
- `Publisher.collect()` — gather multiple `LoadRequest` publishers
- `Publisher.sink(receiveCompletion:receiveValue:)` — receive async results

### MultipeerConnectivity (referenced)
- `MCPeerID`, `MCSession` (with `encryptionPreference: .required`)
- `MCNearbyServiceAdvertiser`, `MCNearbyServiceBrowser`

## Code Highlights

Async multi-asset loading with Combine:
```swift
let loads = (0..<8).map { i in Entity.loadModelAsync(named: "card_\(i)") }
loads.collect().sink(
    receiveCompletion: { _ in },
    receiveValue: { [weak self] models in
        self?.cardTemplates = models
    }
).store(in: &cancellables)
```

Custom component and entity:
```swift
struct CardComponent: Component, Codable {
    var revealed: Bool = false
    var kind: String = ""
}

class CardEntity: Entity, HasModel, HasCollision {
    var card: CardComponent {
        get { components[CardComponent.self] ?? CardComponent() }
        set { components[CardComponent.self] = newValue }
    }
    func reveal() {
        card.revealed = true
        components[SynchronizationComponent.self]?.ownershipTransferMode = .manual
        move(to: Transform(rotation: simd_quatf(angle: .pi, axis: [1,0,0])),
             relativeTo: parent, duration: 0.3, timingFunction: .easeInOut)
    }
}
```

Requesting ownership before modifying a shared entity:
```swift
card.requestOwnership { result in
    if result == .granted { card.reveal() }
}
```

## Takeaways
- Always load assets asynchronously with `Entity.loadModelAsync` in AR apps; synchronous loading blocks ARKit's world observation and delays anchor placement.
- Every entity that needs to participate in hit testing must call `generateCollisionShapes(recursive:)`; clones inherit collision shapes automatically.
- Use `OcclusionMaterial` on a box (not a plane) to correctly simulate real-world occlusion from any viewing angle.
- RealityKit's built-in multiplayer requires only setting `scene.synchronizationService` and enabling ARKit's Collaborative Session; ownership transfer and per-component `Codable` sync handle the rest automatically.

---
_Source: WWDC19 Session 605 page (abstract, full transcript, and resource links including RealityKit and ARKit documentation)._
