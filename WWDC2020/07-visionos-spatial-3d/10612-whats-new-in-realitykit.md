# What's New in RealityKit
**WWDC20 · Session 10612** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10612/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
RealityKit received five major feature areas in iOS 14: video materials (videos as animated textures with spatialized audio), scene understanding via the LiDAR scanner on iPad Pro (occlusion, shadows, physics, and collision with real-world geometry), improved rendering debugging via a new debug model component, expanded face tracking (now works without a TrueDepth camera on A12+ devices), and location anchors (placing AR content at real-world GPS coordinates via ARKit 4's `ARGeoAnchor`).

Scene understanding is the flagship addition, enabling AR experiences where virtual content truly interacts with the physical world. Real-world geometry is represented as automatically managed `HasSceneUnderstanding` entities that can be used for ray-casting, collision event detection, occlusion masking, shadow receiving, and physics simulation.

## Key Topics

### Video Materials
`VideoMaterial` wraps an `AVPlayer` to use any video file (or HLS stream) as an animated texture on any entity. The video's audio automatically becomes a spatialized audio source positioned at the entity, with no manual synchronization required. Supports play/pause/seek via `AVPlayer`, `AVPlayerLooper` for looping, `AVQueuePlayer` for playlists, and remote HLS streaming.

### Scene Understanding (LiDAR Scanner)
`ARView.environment.sceneUnderstanding` is a new option set with four options:
- `.occlusion` — real-world geometry occludes virtual objects
- `.receivesLighting` — virtual objects cast shadows onto real surfaces (auto-enables occlusion)
- `.physics` — virtual objects physically interact with real-world geometry (auto-enables collision)
- `.collision` — generates collision events and enables ray-casting against real-world geometry

Real-world objects are represented as **scene understanding entities** — read-only entities managed by RealityKit containing transform, collision, and physics components plus a unique `SceneUnderstandingComponent`. They conform to `HasSceneUnderstanding` and belong to `CollisionGroup.sceneUnderstanding`. The reconstructed mesh is a continuous approximation, not a crisp model; shadows only cast downward (not on walls).

### Rendering Debugging
A new `DebugModelComponent` can be assigned to any entity to visualize 16 rendering properties grouped as: vertex attributes, material parameters, and PBR outputs (diffuse/specular lighting received, etc.). Visualization applies only to the targeted entity, not its children.

### ARKit 4 Integration: Face Tracking
Face tracking (and thus `AnchorEntity` face anchors) now works on any device with an A12 processor or later, not just devices with a TrueDepth camera. No code changes required for existing face-anchor apps.

### ARKit 4 Integration: Location Anchors
`ARGeoAnchor` (a subclass of `ARAnchor`) lets you create anchors at real-world GPS coordinates. In RealityKit, create an `AnchorEntity` using an `ARGeoAnchor`. Requires `ARGeoTrackingConfiguration` instead of world tracking; scene understanding does not work simultaneously with geo tracking.

## APIs & Frameworks

### RealityKit
- `VideoMaterial` **[NEW]** — video-based material; wraps `AVPlayer`; emits spatialized audio from entity
- `ARView.environment.sceneUnderstanding` **[NEW]** — `SceneUnderstandingOptions` option set
  - `.occlusion` **[NEW]**
  - `.receivesLighting` **[NEW]**
  - `.physics` **[NEW]**
  - `.collision` **[NEW]**
- `SceneUnderstandingComponent` **[NEW]** — component unique to real-world entities
- `HasSceneUnderstanding` **[NEW]** — protocol/trait for scene understanding entities
- `CollisionGroup.sceneUnderstanding` **[NEW]** — collision group for real-world entities
- `Scene.raycast(origin:direction:)` — ray-cast; now works against real-world geometry **[UPDATED]**
- `CollisionEvents.Began` — collision event subscription; now fires against real-world entities **[UPDATED]**
- `DebugModelComponent` **[NEW]** — inspect rendering properties (16 available: vertex attributes, material params, PBR outputs)
- `ARView.debugOptions` — `.showSceneUnderstanding` **[NEW]** — visualize raw LiDAR mesh (color-coded by distance)
- `AnchorEntity(anchor:)` — existing initializer now accepts `ARGeoAnchor` for location anchors

### AVFoundation (used with VideoMaterial)
- `AVPlayer` — controls video playback for `VideoMaterial`
- `AVPlayerItem`, `AVURLAsset` — video asset loading
- `AVPlayerLooper` — looping video playback
- `AVQueuePlayer` — sequential video playback queue
- HLS streaming support via `AVPlayer`

### ARKit 4
- `ARGeoAnchor` **[NEW]** — GPS-coordinate-based world anchor
- `ARGeoTrackingConfiguration` **[NEW]** — configuration for location anchor sessions
- Face tracking extended to A12+ devices without TrueDepth **[NEW]**

## Code Highlights

Loading and playing a video material:
```swift
let asset = AVURLAsset(url: Bundle.main.url(forResource: "glow", withExtension: "mp4")!)
let playerItem = AVPlayerItem(asset: asset)
let player = AVPlayer()
bugEntity.materials = [VideoMaterial(player: player)]
player.replaceCurrentItem(with: playerItem)
player.play()
```

Ray-casting against real-world geometry:
```swift
let bugOrigin = bug.position(relativeTo: nil)
let bugForward = bug.convert(direction: [0, 0, 1], relativeTo: nil)
let collisionResults = arView.scene.raycast(origin: bugOrigin, direction: bugForward)
let filteredResults = collisionResults.filter { $0.entity is HasSceneUnderstanding }
guard let closestCollisionPoint = filteredResults.first?.position else { return }
if length(bugOrigin - closestCollisionPoint) < safeDistance { /* avoid obstacle */ }
```

Handling collision with the real world:
```swift
arView.scene.subscribe(to: CollisionEvents.Began.self) { event in
    guard let sceneUnderstandingEntity = (event.entityA as? HasSceneUnderstanding)
                                      ?? (event.entityB as? HasSceneUnderstanding)
    else { return }
    let bugEntity = (sceneUnderstandingEntity == event.entityA) ? event.entityB : event.entityA
    // disintegrate bugEntity
}
```

Collision filtering with real-world geometry:
```swift
entity.collision?.filter.mask = [.sceneUnderstanding]              // only real world
entity.collision?.filter.mask = CollisionGroup.all.subtracting(.sceneUnderstanding) // never real world
```

## Takeaways

- `VideoMaterial` turns any `AVPlayer` source into an animated texture with automatic spatialized audio — no synchronization code needed.
- Scene understanding enables genuine physical interaction between virtual content and LiDAR-scanned real-world geometry through four configurable options; the entities are read-only and managed by RealityKit.
- The new `DebugModelComponent` exposes 16 rendering properties per entity for iterative material and geometry debugging.
- Face tracking on A12+ devices (including iPhone SE) and location anchors via `ARGeoAnchor` require no changes to existing face-anchor code, but location anchors require `ARGeoTrackingConfiguration` and are incompatible with scene understanding.

---
_Source: WWDC20 Session 10612 page (abstract, transcript, code samples, and resource links)._
