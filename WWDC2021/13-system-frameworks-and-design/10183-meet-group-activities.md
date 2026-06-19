# Meet Group Activities
**WWDC21 · Session 10183** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10183/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
Group Activities is a new Swift-native framework powering SharePlay, Apple's system for shared FaceTime/Messages experiences. It enables apps to build synchronized, real-time shared experiences — ranging from synchronized media playback to fully custom collaborative activities — on top of an end-to-end encrypted communication channel managed by the system.

SharePlay is built around three core concepts: FaceTime/Messages as the communication layer, a cross-platform session model that works on iOS/iPadOS/macOS/tvOS, and deep AVFoundation integration for time-synchronized media playback. A new playback-sync protocol built into AVFoundation keeps all participants at exactly the same playback position, handles play/pause/seek propagation, and smart volume ducking — all without retransmitting media content, so full-fidelity streams always come from the app's own servers.

This session provides the conceptual overview and architecture introduction; companion sessions go deeper into custom experiences, AVFoundation integration, and web (WebKit) support.

## Key Topics

**SharePlay Sessions**
Before sharing, users join a session (automatically managed by the system). Sessions allow fluid switching between FaceTime audio/video and Messages. Once in a session, users can navigate anywhere in the system and drop into any SharePlay-enabled app.

**GroupActivity Protocol**
The app defines a shared experience by creating a type conforming to `GroupActivity`. This object carries the information needed for the activity (e.g., content URL for media, or state data for custom activities). The app calls `prepareForActivation()` to prompt the user for consent, then `activate()` to initiate sharing.

**GroupSession**
Represents the group participating in the shared experience. Provides access to participants. Used by apps (and by AVFoundation internally) to exchange coordination messages. Not for large data; end-to-end encrypted.

**Observing Sessions via AsyncSequence**
Both the initiating and receiving apps observe incoming sessions through an `AsyncSequence` on `GroupSession`. Both sides use the same code path to receive and join sessions.

**AVFoundation Integration**
For media playback, apps connect their `AVPlayer`'s `AVPlaybackCoordinator` to the `GroupSession`. This handles time synchronization automatically. Apps using custom players can use `AVDelegatingPlaybackCoordinator` for the same benefit.

**Smart Volume and Picture in Picture**
Smart volume automatically ducks media audio when participants speak. Full Picture in Picture support lets users multitask while sharing.

**Platform Reach**
Available on iOS 15, iPadOS 15, macOS Monterey, tvOS 15. WebKit support on macOS for web-based media experiences.

## APIs & Frameworks

### GroupActivities Framework **[NEW]**
- `GroupActivity` protocol — defines a shareable activity **[NEW]**
  - `metadata` — `GroupActivityMetadata` with title, subtitle, type
  - `prepareForActivation() async -> GroupActivityActivationResult` — prompts user for consent **[NEW]**
  - `activate() async throws` — initiates sharing with the group **[NEW]**
- `GroupSession<Activity>` — represents the active shared session **[NEW]**
  - `AsyncSequence` on `GroupSession<Activity>.sessions` — observe incoming sessions **[NEW]**
  - `participants` — set of `Participant` objects in the session
  - `join()` — join the session after setting up (e.g., connecting AVPlaybackCoordinator) **[NEW]**
  - `end()` — end the session **[NEW]**
  - `leave()` — leave the session without ending it for others **[NEW]**
- `GroupActivityMetadata` — title, subtitle, type for system presentation **[NEW]**
- `GroupActivityActivationResult` — `.activationPreferred` / `.activationDisabled` / `.cancelled` **[NEW]**

### AVFoundation Integration **[NEW]**
- `AVPlayer.playbackCoordinator` — `AVPlaybackCoordinator` property for sync **[NEW]**
- `AVPlaybackCoordinator.coordinateWithSession(_:)` — connects player to `GroupSession` **[NEW]**
- `AVDelegatingPlaybackCoordinator` — for custom (non-AVPlayer) media players **[NEW]**

### Messages / FaceTime
- SharePlay automatically integrates with FaceTime and Messages sessions
- System provides UI for inviting participants, switching communication modes, and leaving sessions

## Code Highlights

No full code samples in this overview session. Conceptual flow:

```swift
// 1. Define an activity
struct WatchMovieActivity: GroupActivity {
    var metadata: GroupActivityMetadata {
        var metadata = GroupActivityMetadata()
        metadata.title = "Watch Movie Together"
        metadata.type = .watchTogether
        return metadata
    }
    var contentURL: URL
}

// 2. Initiate sharing
let activity = WatchMovieActivity(contentURL: movieURL)
switch await activity.prepareForActivation() {
case .activationPreferred:
    try await activity.activate()
default:
    break
}

// 3. Observe sessions (both initiator and receiver)
for await session in WatchMovieActivity.sessions() {
    // Configure AVPlayer
    player.playbackCoordinator.coordinateWithSession(session)
    session.join()
}
```

## Takeaways
- Group Activities is a Swift-native framework that makes any FaceTime call a platform for synchronized shared experiences with minimal code — the system handles session management, encryption, and UI.
- AVFoundation's `AVPlaybackCoordinator` provides automatic time-synchronized playback across all participants; media is never retransmitted so full-fidelity streams always come from the app's servers.
- Both the activity initiator and receivers observe sessions through the same `AsyncSequence` on `GroupSession`, reducing the amount of platform-specific logic needed.
- SharePlay is available on iOS, iPadOS, macOS, tvOS, and via WebKit — covering virtually every Apple screen.

---
_Source: WWDC21 Session 10183 page (abstract, chapter summaries, code samples, and resource links)._
