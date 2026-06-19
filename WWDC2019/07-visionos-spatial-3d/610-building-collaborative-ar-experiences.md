# Building Collaborative AR Experiences
**WWDC19 · Session 610** · [Watch](https://developer.apple.com/videos/play/wwdc2019/610/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
ARKit 3 introduces Collaborative Sessions, a new mode for live multi-user AR that continuously shares world map data and ARAnchor state across all participants without requiring a host device or a pre-baked environment scan. Combined with RealityKit's built-in networking, developers can ship a multiplayer AR experience with dramatically less network synchronization code than was required in iOS 12.

The session covers three topics: how Collaborative Sessions work architecturally (decentralized, peer-to-peer, real-time map sharing), best practices for anchoring virtual content so it remains stable and consistent across devices, and a detailed case study of SwiftStrike — a full-size bowling AR game built with RealityKit and ARKit 3 that demonstrates ownership, custom components, physics tuning, and People Occlusion design.

## Key Topics

**Collaborative Session vs. ARKit 2 Map Save/Load**
- Map Save/Load: pre-bake one world map, load it on all devices; static, no live sharing after start
- Collaborative Session: decentralized, no host device, each user builds their own map while continuously sharing "collaboration data" (ARWorldMap segments) with all peers; any user can add anchors at any time and they appear on all devices instantly

**How Collaborative Sessions Work**
- Each user starts their own coordinate space; maps merge locally once devices observe overlapping areas
- ARKit transmits `ARCollaborationData` (compact ARWorldMap segments) to all peers via the app's network layer
- After localization against another device's map, that device's `ARAnchor` objects appear in the local session
- `ARParticipantAnchor` **[NEW]** — represents another user's current position in local world coordinates; updated at camera frame rate
- `ARAnchor.sessionIdentifier` **[NEW]** — UUID identifying the originating device; use to distinguish self-vs-peer anchors

**Implementation Steps**
1. Connect all devices via `MultipeerConnectivity` (or any network layer with reliable delivery)
2. Set `ARWorldTrackingConfiguration.isCollaborationEnabled = true` **[NEW]**
3. In `ARSessionDelegate.session(_:didOutputCollaborationData:)`: send `ARCollaborationData` to all peers **[NEW]**
4. On receipt: call `arSession.update(with: collaborationData)` **[NEW]**
5. (If using RealityKit) set `ARView.session.run(config)` — RealityKit automatically handles collaborative data transport via `MultipeerConnectivityService`

**Anchor Best Practices**
- Always respond to `ARSessionDelegate.session(_:didUpdate:)` for anchor position changes; ARKit refines anchor positions as the map improves
- Place virtual objects close to their `ARAnchor` — large offsets amplify position jitter when anchors update
- Use multiple independent anchors for unrelated objects; use one shared anchor only for objects that must maintain relative positions

**SwiftStrike Architecture (Case Study)**
- RealityKit networking handles physics sync, entity component sync, People Occlusion — ~1,500 lines of custom sync code from SwiftShot eliminated
- Custom components conform to `Component` + `Codable`; registered before `ARView` init; automatically serialized and synchronized
- `MatchStateComponent` tracks game state transitions as an append-only log so clients see every state even under network delay
- Host device owns the physics simulation; client devices create a `PlayerLocationEntity` (client-owned) containing a `PaddleEntity` (host-owned) — host controls paddle activation while client keeps position updated
- Collision masks: ball ↔ pins and ball ↔ player; pins and player do not collide with each other
- Physics shapes: pins use combination of primitive spheres + convex hulls (keep convex hulls simple for performance)
- World localization: combination of pre-baked ARWorldMap (fast startup) + collaborative data (ongoing refinement)
- Image anchor on floor logo used for precise game board placement
- People Occlusion enabled full-size (not tabletop) gameplay design

## APIs & Frameworks

### ARKit 3 (NEW)
- `ARWorldTrackingConfiguration.isCollaborationEnabled: Bool` **[NEW]** — enables collaborative session
- `ARCollaborationData` **[NEW]** — compact world map segment to transmit to peers
- `ARSession.update(with: ARCollaborationData)` **[NEW]** — inject peer collaboration data
- `ARSessionDelegate.session(_:didOutputCollaborationData:)` **[NEW]** — callback when local session produces data to share
- `ARParticipantAnchor` **[NEW]** — high-frequency anchor representing another user's position
- `ARAnchor.sessionIdentifier: UUID` **[NEW]** — identifies which device created the anchor

### ARKit (existing, referenced)
- `ARWorldMap` / Map Save and Load — predecessor approach; still valid for persistent experiences
- `ARImageAnchor` — used in SwiftStrike for floor logo to place game board
- `ARFrame.worldMappingStatus` — check `.mapped` to confirm another device can localize against your map
- `ARSessionDelegate.session(_:didAdd:)`, `session(_:didUpdate:)`, `session(_:didRemove:)` — anchor lifecycle callbacks

### RealityKit (NEW, referenced)
- `ARView.session` — `ARSession` shared with ARKit; set `isCollaborationEnabled` on its config
- `MultipeerConnectivityService` — automatically handles collaborative data transport when assigned to `Scene.synchronizationService`
- `Component` + `Codable` — custom components synchronize automatically across peers
- `SynchronizationComponent.ownershipTransferMode` — `.autoAccept` / `.manual` for ownership control
- `Entity.requestOwnership(completionHandler:)` — client requests ownership before modifying
- `PhysicsBodyComponent` — rigid body; `ShapeResource` for collision shapes (sphere, box, capsule, convex hull)
- `CollisionComponent` with collision filter masks — configure which objects collide with each other

### MultipeerConnectivity (referenced)
- `MCPeerID`, `MCSession` (encryption required), `MCNearbyServiceAdvertiser`, `MCNearbyServiceBrowser`
- Reliable message delivery required for `ARCollaborationData`

### Combine (referenced)
- Used in SwiftStrike to observe custom component changes and broadcast to game logic

## Code Highlights

Enabling collaborative session:
```swift
let config = ARWorldTrackingConfiguration()
config.isCollaborationEnabled = true
arView.session.run(config)
```

Transmitting collaboration data (without RealityKit auto-handling):
```swift
func session(_ session: ARSession, didOutputCollaborationData data: ARSession.CollaborationData) {
    guard let encodedData = try? NSKeyedArchiver.archivedData(
        withRootObject: data, requiringSecureCoding: true) else { return }
    mcSession.send(encodedData, toPeers: mcSession.connectedPeers, with: .reliable)
}

// On receive:
func session(_ session: MCSession, didReceive data: Data, fromPeer: MCPeerID) {
    if let collabData = try? NSKeyedUnarchiver.unarchivedObject(
        ofClass: ARSession.CollaborationData.self, from: data) {
        arSession.update(with: collabData)
    }
}
```

Custom synchronized component:
```swift
struct MatchStateComponent: Component, Codable {
    enum Transition: Codable { case waitingForPlayers, countdown, playing, finished }
    var transitions: [Transition] = []
}

// Register before ARView init:
MatchStateComponent.registerComponent()
```

Distinguishing own vs peer anchors:
```swift
func session(_ session: ARSession, didAdd anchors: [ARAnchor]) {
    for anchor in anchors {
        if anchor.sessionIdentifier == session.identifier {
            // own anchor
        } else {
            // peer's anchor — show their virtual object
        }
    }
}
```

## Takeaways
- Collaborative Sessions require only `isCollaborationEnabled = true` plus transmitting `ARCollaborationData` to peers — RealityKit + `MultipeerConnectivityService` handles the transport automatically.
- Have one device establish and hold a `.mapped` world map before others approach; side-by-side camera views localize fastest.
- Always respond to anchor updates and keep virtual objects close to their `ARAnchor` to avoid jarring position jumps during map refinement.
- RealityKit's automatic physics and component sync eliminated ~1,500 lines of manual network code vs. the ARKit 2 equivalent (SwiftShot).

---
_Source: WWDC19 Session 610 page (abstract, full transcript, and resource links including "Creating a collaborative session" documentation)._
