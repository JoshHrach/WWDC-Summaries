# Create a Seamless Multiview Playback Experience
**WWDC25 · Session 302** · [Watch](https://developer.apple.com/videos/play/wwdc2025/302/)

_Platforms:_ iOS 26, tvOS 26, iPadOS 26

## Overview
This session introduces APIs for coordinating multi-player media playback — enabling multiple `AVPlayer` instances to stay synchronized across play, pause, seek, stall, and startup events. Two new mechanisms are covered: `AVPlaybackCoordinationMedium` (AVFoundation) for coordinating players within the same process or across a session, and `AVRoutingPlaybackArbiter` for managing routing priority when multiple players compete for audio routes and external displays.

Use cases include picture-in-picture multiview (sports broadcasts), synchronized multi-angle viewing, and coordinated group listening in connected TV experiences.

## Key Topics

### AVPlaybackCoordinationMedium
`AVPlaybackCoordinationMedium` is a new abstraction layer that enables `AVPlaybackCoordinator` to synchronize multiple players using a shared medium. Players call `coordinator.coordinate(using: medium)` to join the synchronized session. The medium handles the logic for who leads and who follows for play/pause, seek, rate changes, and stall recovery.

### AVRoutingPlaybackArbiter
`AVRoutingPlaybackArbiter.shared()` returns the singleton arbiter that mediates conflicts between multiple players competing for external audio routes (AirPlay, Bluetooth) and AirPlay video routes. Two properties control priority:
- `preferredParticipantForExternalPlayback` — which player gets the external display
- `preferredParticipantForNonMixableAudioRoutes` — which player gets the audio route

### Network Resource Priority
`AVPlayer.networkResourcePriority` — new property with `.high` and `.low` values — lets apps express which player's content is most important to buffer first when bandwidth is limited.

### Coordination Events
The medium handles these synchronization events automatically: play/pause, seek to time, rate changes, startup (waiting for content to load), and stall (buffering interruption). All participants in the medium converge to a consistent playback state.

## APIs & Frameworks

**AVFoundation (iOS 26, tvOS 26)**
- **[NEW]** `AVPlaybackCoordinationMedium` — shared coordination medium for multi-player sync
- **[NEW]** `AVPlaybackCoordinator.coordinate(using: AVPlaybackCoordinationMedium)` — join coordination session
- **[NEW]** `AVRoutingPlaybackArbiter` — arbitrate routing conflicts between players
- **[NEW]** `AVRoutingPlaybackArbiter.shared()` — singleton arbiter instance
- **[NEW]** `AVRoutingPlaybackArbiter.preferredParticipantForExternalPlayback` — priority for AirPlay video
- **[NEW]** `AVRoutingPlaybackArbiter.preferredParticipantForNonMixableAudioRoutes` — priority for audio routes
- **[NEW]** `AVPlayer.networkResourcePriority` — `.high`, `.low`
- `AVPlaybackCoordinator` — existing coordinator type, now extended
- Coordinated events: play, pause, seek, rate change, startup, stall

## Code Highlights
Coordinate two players using a shared medium:
```swift
let medium = AVPlaybackCoordinationMedium()

let player1 = AVPlayer(url: mainAngleURL)
try await player1.playbackCoordinator.coordinate(using: medium)

let player2 = AVPlayer(url: alternateAngleURL)
try await player2.playbackCoordinator.coordinate(using: medium)

// Now player1 and player2 stay synchronized
player1.networkResourcePriority = .high
player2.networkResourcePriority = .low
```

Configure routing arbitration:
```swift
let arbiter = AVRoutingPlaybackArbiter.shared()
arbiter.preferredParticipantForExternalPlayback = player1.playbackCoordinator
arbiter.preferredParticipantForNonMixableAudioRoutes = player1.playbackCoordinator
```

## Takeaways
- Use `AVPlaybackCoordinationMedium` to synchronize multiple `AVPlayer` instances for multiview and multi-angle scenarios without manually tracking play/pause state across instances.
- Use `AVRoutingPlaybackArbiter` to control which player "wins" when multiple players compete for AirPlay or Bluetooth audio routes.
- Set `AVPlayer.networkResourcePriority = .high` on the primary player to ensure it buffers preferentially on constrained network connections.

---
_Source: WWDC25 Session 302 page (abstract, chapter summaries, code samples, and resource links)._
