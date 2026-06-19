# Make a Great SharePlay Experience
**WWDC22 · Session 10139** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10139/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
This session covers design and implementation best practices for building great SharePlay experiences using the Group Activities framework. It introduces the concept of "presence" — making people feel as if they are together in the same space even when apart — as the guiding design principle for SharePlay activities, and walks through the full lifecycle of a SharePlay session: starting, during, and ending.

A key new capability in iOS 16 is the ability to start a SharePlay session directly from within an app (or from Messages), without requiring an active FaceTime call. This expands the range of contexts in which SharePlay can be surfaced to users.

## Key Topics
- **SharePlay as a presence technology** — the "portal exercise": imagine your app in physical space; design controls and interactions as if everyone were physically present together
- **Identifying group activities** — look for activities people enjoy doing together in person (cooking, music, fitness, games, watching video); not every personalized experience translates directly to a group experience
- **Two types of coordinated experiences:**
  - *Single-view* — all participants see and experience the same content in sync (e.g., video playback with `AVPlaybackCoordinator`)
  - *Multi-view* — participants have different views of the same coordinated activity (e.g., Heads Up! — one person guesses, others give clues); use `GroupSessionMessenger.send(_:to:)` to send messages to specific participant subsets
- **Starting a SharePlay session** — new in iOS 16: start directly from the app UI using `GroupActivitySharingController`; register the activity with `GroupActivity.registerGroupActivity(_:)` to surface in the Share Sheet; start from Messages without a FaceTime call
- **Group activity metadata** — implement `GroupActivity.metadata` returning `GroupActivityMetadata` with descriptive title, subtitle, image, and a `webPageURL` for non-app users
- **Lobby / late-joiner handling** — support a lobby state where participants wait for all members to join before beginning (important for turn-based experiences)
- **Contextual UI during activity** — communicate why the UI changes (e.g., who paused the video, who drew a stroke); show participant attribution in shared canvases or playback controls
- **Status bar and controls accessibility** — avoid permanently hiding the status bar; provide a gesture to reveal it (e.g., single tap); the SharePlay pill in the status bar provides access to key controls
- **Picture in Picture** — support PiP for coordinated media so users can multitask (respond to Messages, switch apps) without leaving the shared experience
- **Shared playback controls** — do not restrict who can interact with playback controls; everyone should feel able to "walk up" and interact

## APIs & Frameworks
**Group Activities framework**
- `GroupActivity` protocol — defines a SharePlay activity; requires `metadata: GroupActivityMetadata`
- `GroupActivityMetadata` — `title`, `subtitle`, `previewImage`, `webPageURL`
- `GroupActivity.registerGroupActivity(_:)` — registers the activity to appear in the Share Sheet **[existing, highlighted]**
- `GroupActivitySharingController` **[NEW]** — UIKit/AppKit controller for starting a SharePlay session directly from the app UI (without requiring an active FaceTime call); presents UI for selecting friends and choosing FaceTime or Messages
- `GroupSessionMessenger` — sends messages to all or specific participants
- `GroupSessionMessenger.send(_:to:)` — sends a message to a subset of participants (for multi-view experiences)
- `GroupSession` — represents the active SharePlay session
- `GroupSession.activeParticipants` — set of currently active participants

**AVFoundation**
- `AVPlaybackCoordinator` — coordinates playback state (play/pause/seek) across all participants for single-view media experiences
- `AVPlayer.coordinateWithSession(_:)` / `AVQueuePlayer.coordinateWithSession(_:)` — connects an AVPlayer to a group session for synchronized playback

**UIKit / System**
- Picture in Picture (`AVPictureInPictureController`) — support for multitasking during SharePlay media sessions
- Status bar management — ensure the SharePlay pill remains accessible; use single-tap gesture to reveal hidden status bar

## Code Highlights
No new code samples in this session (design-focused). Key API touchpoints:

Register group activity for Share Sheet:
```swift
// In the activity type definition
struct WatchTogetherActivity: GroupActivity {
    var metadata: GroupActivityMetadata {
        var metadata = GroupActivityMetadata()
        metadata.title = "Watch Together"
        metadata.webPageURL = URL(string: "https://example.com/join")
        return metadata
    }
}
// Register it to appear in Share Sheet
WatchTogetherActivity.registerGroupActivity()
```

Start SharePlay from app UI (new in iOS 16):
```swift
// Present GroupActivitySharingController from within your app
let controller = GroupActivitySharingController(WatchTogetherActivity())
present(controller, animated: true)
```

Send a message to a subset of participants (multi-view):
```swift
let messenger = GroupSessionMessenger(session: groupSession)
try await messenger.send(myMessage, to: .only(subsetOfParticipants))
```

## Takeaways
- Design SharePlay activities around what people enjoy doing together in person — use the "portal exercise" to think through what shared controls and presence cues are needed.
- iOS 16 allows SharePlay sessions to start directly from the app or from Messages, without a FaceTime call — register your activity and adopt `GroupActivitySharingController` to surface these entry points.
- Provide meaningful `GroupActivityMetadata` (title, subtitle, image, web URL) so participants know what to expect when joining.
- Support PiP, accessible status bar/controls, and contextual attribution UI to maintain presence and usability during the shared experience.

---
_Source: WWDC22 Session 10139 page (abstract, chapter summaries, code samples, and resource links)._
