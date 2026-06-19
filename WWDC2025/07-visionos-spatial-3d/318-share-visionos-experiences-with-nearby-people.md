# Share visionOS Experiences with Nearby People
**WWDC25 · Session 318** · [Watch](https://developer.apple.com/videos/play/wwdc2025/318/)

_Platforms:_ visionOS 26

## Overview
This session introduces nearby sharing for SharePlay in visionOS 26. When multiple Apple Vision Pro users are physically co-located, GroupActivities sessions can now detect this proximity and share spatial information — including each participant's 3D pose — to enable synchronized, spatially-aware shared experiences. ARKit's `WorldAnchor` gains a new shared mode for nearby participants, allowing all users in proximity to share the same anchored content.

## Key Topics

### Nearby SharePlay Detection
A new `isNearbyWithLocalParticipant` property on SharePlay's participant model allows apps to determine whether a remote participant is physically near the local user. This enables apps to branch their experience — showing full spatial coordination for nearby users while providing a different (possibly 2D) experience for remote users.

### Participant Spatial Pose
`ParticipantState.pose` provides the 3D spatial position and orientation of each nearby participant in the shared coordinate space. Apps can use this to render representations of other participants' avatars, align virtual objects relative to where each person is standing, or coordinate spatial interactions.

### groupActivityAssociation Modifier
A new SwiftUI modifier (`groupActivityAssociation`) links a specific SwiftUI view or window to an active group activity session. This allows different parts of an app's UI to be contextually associated with particular SharePlay sessions when multiple group activities are running simultaneously.

### Shared WorldAnchor (ARKit)
`WorldAnchor` in ARKit gains a `sharedWithNearbyParticipants` parameter. When enabled, an anchor created by one user is visible and consistent across all nearby participants' spaces, enabling true shared spatial anchoring without requiring a separate cloud anchor service.

## APIs & Frameworks

**GroupActivities / SharePlay**
- `isNearbyWithLocalParticipant` **[NEW]** — Bool property on participant state; true when the remote participant is physically co-located with the local user
- `ParticipantState.pose` **[NEW]** — 3D pose (position + orientation) of a nearby participant in the shared spatial coordinate system
- `groupActivityAssociation(_:)` **[NEW]** — SwiftUI view modifier associating a view/window with a specific active group activity

**ARKit**
- `WorldAnchor(sharedWithNearbyParticipants:)` **[NEW]** — creates a world anchor automatically shared with all nearby SharePlay participants; provides a consistent spatial reference across devices

## Code Highlights

```swift
// Check if a participant is nearby and access their pose
for participant in session.activeParticipants {
    if participant.isNearbyWithLocalParticipant {
        let pose = participant.pose // simd_float4x4 position + orientation
        // Position avatar or align content using pose
    }
}
```

```swift
// Create a shared WorldAnchor visible to all nearby participants
let anchor = WorldAnchor(sharedWithNearbyParticipants: true)
try await arSession.run([anchor])
```

```swift
// Associate a SwiftUI view with the current group activity
ContentView()
    .groupActivityAssociation(currentGroupActivity)
```

## Takeaways
- Use `isNearbyWithLocalParticipant` to detect co-located users and enable richer spatial experiences than are possible with remote-only SharePlay.
- Leverage `ParticipantState.pose` to place participant avatars or spatially coordinate virtual objects relative to each person's physical position.
- Use `WorldAnchor(sharedWithNearbyParticipants: true)` to create a shared spatial origin point without a third-party cloud anchor service.
- Apply `groupActivityAssociation` to connect UI elements to specific group activities when your app can host multiple concurrent sessions.

---
_Source: WWDC25 Session 318 page (abstract, chapter summaries, code samples, and resource links)._
