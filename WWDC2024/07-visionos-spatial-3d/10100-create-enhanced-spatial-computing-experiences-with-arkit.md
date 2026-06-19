# Create Enhanced Spatial Computing Experiences with ARKit
**WWDC24 · Session 10100** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10100/)

_Platforms:_ visionOS 2

## Overview
ARKit on visionOS 2 gains several new data providers and API improvements that let developers build richer spatial computing experiences. This session surveys the expanded ARKit surface: new hand-tracking fidelity with skeleton joints, improved plane detection with classification updates, new room tracking for persistent spatial anchors within a room, and a Scene Reconstruction mesh update API for streaming real-world geometry changes.

The talk frames each feature within a practical use case — placing persistent virtual objects in a room, reacting to furniture moved in the user's environment, and building fine-grained hand interaction — then shows the corresponding ARKit provider initialization and data access code.

## Key Topics
- **Hand skeleton tracking** — `HandAnchor` now exposes a full skeleton with named joint transforms, enabling per-finger and per-phalanx pose queries for glove-level fidelity.
- **Plane detection updates** — `PlaneAnchor` gains a richer `classification` enum with new values (`.table`, `.seat`, `.window`, `.door`); plane geometry is returned as a polygon rather than a bounding rectangle.
- **Room tracking** — new `RoomTrackingProvider` delivers persistent room-scoped `RoomAnchor` identifiers so apps can store virtual objects relative to a recognized room and restore them on the next launch.
- **Scene Reconstruction update streaming** — `SceneReconstructionProvider` now emits incremental mesh updates rather than full mesh replacements, reducing processing overhead when the environment changes.
- **World tracking improvements** — `WorldTrackingProvider` has reduced latency anchors for time-critical placement.

## APIs & Frameworks

**ARKit (visionOS)**
- `ARKitSession` — unchanged entry point; run providers with `.run(_:)`
- `HandTrackingProvider` — existing provider
  - **[NEW]** `HandAnchor.handSkeleton` — full skeleton with named joints
  - **[NEW]** `HandSkeleton.joint(_:)` — look up a `HandSkeleton.Joint` by `HandSkeleton.JointName`
  - **[NEW]** `HandSkeleton.JointName` — enum covering wrist, metacarpals, and all three phalanges per finger + thumb
  - `HandSkeleton.Joint.localTransform` / `.rootTransform` — pose of the joint in local or world space
- `PlaneDetectionProvider` — existing provider
  - **[NEW]** `PlaneAnchor.Classification` additions — `.table`, `.seat`, `.window`, `.door`, `.unknown`
  - **[NEW]** `PlaneAnchor.geometry.polygon` — polygon vertices of the detected plane surface (replaces bounding rectangle only)
- **[NEW]** `RoomTrackingProvider` — new provider; delivers `RoomAnchor` values
  - **[NEW]** `RoomAnchor` — persistent anchor tied to a recognized room; stable across sessions
  - `RoomTrackingProvider.requiredAuthorizations` — requires `.worldSensing`
- `SceneReconstructionProvider` — existing provider
  - **[NEW]** `MeshAnchor.geometry` incremental updates — provider now emits diffs; `MeshAnchor.GeometryUpdate.added/updated/removed` distinguish change types
  - `MeshAnchor.geometry.vertices` / `.faces` — unchanged access pattern; now populated incrementally
- `WorldTrackingProvider` — existing provider; latency improvements in visionOS 2 (no new API surface, behavioral change)
- `AnchorUpdate<T>` — existing async sequence type wrapping anchor change events
- `SpatialTrackingSession` — existing; used alongside `ARKitSession` for RealityKit entity anchoring

## Code Highlights
Access individual finger joint positions from the hand skeleton:

```swift
if let indexTip = handAnchor.handSkeleton?.joint(.indexFingerTip) {
    let worldTransform = handAnchor.originFromAnchorTransform * indexTip.rootTransform
    // place virtual object at indexTip world position
}
```

Initialize the new `RoomTrackingProvider` and observe room anchors:

```swift
let roomProvider = RoomTrackingProvider()
let session = ARKitSession()
try await session.run([roomProvider])
for await update in roomProvider.anchorUpdates {
    switch update.event {
    case .added, .updated:
        restoreObjects(for: update.anchor)
    case .removed:
        break
    }
}
```

## Takeaways
- Use `HandSkeleton.JointName` joints (e.g. `.indexFingerTip`, `.thumbTip`) for gesture recognition that goes beyond pinch — per-phalanx tracking enables fully custom hand poses.
- `RoomTrackingProvider` is the right anchor type for persistent placed objects (furniture, sticky notes); it survives app restarts within the same recognized room.
- Subscribe to `SceneReconstructionProvider` incremental updates rather than polling the full mesh on every frame; the new diff API dramatically reduces CPU work when the physical environment is stable.
- Combine `PlaneAnchor.classification` (e.g., `.table`) with plane geometry polygons to anchor virtual content to semantically meaningful surfaces.

---
_Source: WWDC24 Session 10100 page (abstract, chapter summaries, code samples, and resource links)._
