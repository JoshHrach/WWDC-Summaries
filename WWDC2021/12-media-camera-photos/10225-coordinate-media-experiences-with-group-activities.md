# Coordinate Media Experiences with Group Activities
**WWDC21 · Session 10225** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10225/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
Group Activities is a new Swift framework that enables synchronized shared media experiences (SharePlay) over FaceTime. This session covers the complete implementation of a coordinated media playback app: defining a `GroupActivity`, using `prepareForActivation()` to detect intent, managing `GroupSession` lifecycle, and connecting an `AVPlayer` to the new `AVPlaybackCoordinator` for automatic synchronized playback across devices.

The session also details Picture in Picture integration for seamless SharePlay start, and deep-dives into `AVPlayerPlaybackCoordinator` internals — including content identity, coordinated playback suspensions, and the alternative `AVDelegatingPlaybackCoordinator` for non-`AVPlayer` playback engines.

## Key Topics

### GroupActivity Protocol
Conform a type to `GroupActivity` (which also requires `Codable`) to define the shared experience. Set `activityIdentifier` (unique string) and implement `metadata` (async, returns `GroupActivityMetadata` with title/preview image). Include any properties needed to load content (e.g., video URL).

### Sharing and Activation
Call `activity.prepareForActivation()` (async) to determine user intent. Returns `.activationDisabled` (not in a call or wants local playback), `.activationPreferred` (in a call, wants to share), or `.cancelled`. Call `activity.activate()` on `.activationPreferred` to create the session.

### GroupSession Lifecycle
Receive sessions from `GroupActivity.sessions()` AsyncSequence (for all session types produced by the activity). The sequence delivers sessions to both the activating device and remote devices. After receiving a session:
1. Configure content (get the activity's payload, load the right video).
2. Attach the group session to the playback coordinator.
3. Call `session.join()` to start the real-time connection.
Use `session.leave()` to disconnect locally or `session.end()` to end for the entire group.

### AVPlayerPlaybackCoordinator
Access via `AVPlayer.playbackCoordinator`. Call `coordinator.coordinateWithSession(_:)` to link the player to the group session. The coordinator intercepts all transport control API (rate changes, seeks) and:
- Negotiates with remote coordinators before applying commands.
- Keeps all `AVPlayer` instances playing the same item in sync.
- Only applies state when both players have items with matching identity (same URL, or same custom identifier).
- On join: automatically applies the existing group state to the new participant's player.

### Content Identity
By default, two items are "the same" if they share the same URL. Override this by implementing `AVPlayerPlaybackCoordinatorDelegate.playbackCoordinator(_:identifierFor:)` to return a custom string identifier — useful for cached content or `AVComposition`/`AVMutableMovie`.

### Personalized Interstitials (Ads)
Play ads in a separate `AVPlayer`, not in the coordinated player. This keeps the main asset's timeline clean and allows the coordinator to resync after the ad.

### Coordinated Playback Suspensions
Use `AVCoordinatedPlaybackSuspension` to temporarily detach one participant from the group (e.g., during audio session interruption, manual catch-up scrubbing). Begin with `coordinator.beginSuspension(for:)`, then safely seek/rate-change locally. End with `suspension.end()` or `suspension.end(proposingNewTime:)` to rejoin at a specific time. Automatic suspensions are added by `AVPlayerPlaybackCoordinator` for audio interruptions, network stalls, and AVKit scrubbing.

### AVDelegatingPlaybackCoordinator
Alternative for non-`AVPlayer` playback (e.g., custom audio engines). The delegate receives `AVPlaybackCommand` objects (play, pause, seek, buffer) and is responsible for applying them to the custom playback object and updating the coordinator with current item identity.

## APIs & Frameworks

**Group Activities** (`import GroupActivities`) — **[NEW]**
- `GroupActivity` protocol — define a shareable activity **[NEW]**
  - `activityIdentifier: String` — unique type identifier
  - `metadata: GroupActivityMetadata` — async property with title/preview
  - `activate()` — activate and create a group session
  - `prepareForActivation()` — async; returns `GroupActivityActivationResult`
  - `GroupActivityActivationResult` — `.activationDisabled`, `.activationPreferred`, `.cancelled`
  - `sessions()` — static `AsyncSequence` delivering `GroupSession<Self>` objects
- `GroupSession<Activity>` **[NEW]** — real-time session between participants
  - `join()` — connect to the group
  - `leave()` — local disconnect
  - `end()` — end for all participants
  - `activity` publisher (`PassthroughSubject`) — current activity (Combine)
  - `state` — `.waiting`, `.joined`, `.invalidated`
  - `activeParticipants` — set of `Participant`
- `GroupActivityMetadata` **[NEW]** — title, subtitle, previewImage for system UI
- `GroupSessionMessenger` **[NEW]** — send arbitrary `Codable` messages to group participants

**AVFoundation** (`import AVFoundation`)
- `AVPlaybackCoordinator` **[NEW]** — abstract base for coordinated playback
  - `beginSuspension(for:)` — start a coordinated playback suspension
  - `suspensionReasonsThatTriggerWaiting` — configures which suspensions cause others to wait
  - `otherParticipants` — array of `AVCoordinatedPlaybackParticipant`
- `AVPlayerPlaybackCoordinator: AVPlaybackCoordinator` **[NEW]** — ties to `AVPlayer`
  - `coordinateWithSession(_:)` — attach group session to player
  - `delegate` — `AVPlayerPlaybackCoordinatorDelegate`
- `AVPlayerPlaybackCoordinatorDelegate` **[NEW]**
  - `playbackCoordinator(_:identifierFor:)` — return custom string identity for an `AVPlayerItem`
- `AVPlayer.playbackCoordinator` **[NEW]** — access the player's coordinator
- `AVPlayer.playImmediately(atRate:)` — override waiting and start immediately
- `AVPlayer.rateDidChangeNotification` **[NEW]** — includes info on whether change originated from another participant
- `AVPlayerItem.timeJumpedNotification` — **[enhanced]** includes whether jump was from another participant
- `AVPlayer.reasonForWaitingToPlay` — `.waitingForCoordinatedPlayback` **[NEW]** reason
- `AVCoordinatedPlaybackSuspension` **[NEW]**
  - `end()` — rejoin group at current group time
  - `end(proposingNewTime:)` — rejoin group and propose a new time
- `AVCoordinatedPlaybackSuspension.Reason` — extensible struct for suspension reasons
- `AVDelegatingPlaybackCoordinator: AVPlaybackCoordinator` **[NEW]** — for custom playback engines
  - `coordinateWithSession(_:)` — attach group session
  - `playerItem(didFinishPlayingWithError:)` — inform coordinator of item completion
  - Delegate receives `AVPlaybackCommand` subclasses: `AVPlayCommand`, `AVPauseCommand`, `AVSeekCommand`, `AVBufferingCommand`
- `AVPlayerInterstitialEvent` **[NEW]** — plays interstitials in a separate player to preserve main asset timing

**Picture in Picture** (AVKit)
- `GroupSession.isEligibleForGroupImmersiveSpace` / system PiP integration — GroupActivities framework delivers sessions to background app for PiP setup
- `GroupSession` requestForeground APIs — for cases where PiP cannot be used

## Code Highlights

Defining and activating a GroupActivity:
```swift
struct MovieWatchingActivity: GroupActivity {
    var movie: Movie
    static let activityIdentifier = "com.example.MovieWatchingActivity"
    var metadata: GroupActivityMetadata {
        get async {
            var metadata = GroupActivityMetadata()
            metadata.title = movie.title
            return metadata
        }
    }
}

func playButtonTapped() {
    let activity = MovieWatchingActivity(movie: movie)
    Task {
        switch await activity.prepareForActivation() {
        case .activationDisabled: enqueuedMovie = movie
        case .activationPreferred: activity.activate()
        case .cancelled: break
        default: break
        }
    }
}
```

Receiving a GroupSession and connecting to AVPlayer:
```swift
func listenForGroupSession() {
    Task {
        for await groupSession in MovieWatchingActivity.sessions() {
            player.playbackCoordinator.coordinateWithSession(groupSession)
            groupSession.activity.sink { activity in
                enqueue(movie: activity.movie)
            }.store(in: &subscriptions)
            groupSession.join()
        }
    }
}
```

Custom playback suspension:
```swift
extension AVCoordinatedPlaybackSuspension.Reason {
    static let whatHappened = Self("com.example.what-happened")
}

let suspension = coordinator.beginSuspension(for: .whatHappened)
player.seek(to: catchUpTime)
player.rate = 2.0
// When caught up:
suspension.end()
```

## Takeaways
- Group Activities + `AVPlayerPlaybackCoordinator` handle all real-time sync networking; calling `coordinateWithSession(_:)` is the only step needed to enable synchronized playback.
- `prepareForActivation()` elegantly handles the local-vs-shared intent question without requiring apps to check FaceTime state manually.
- Coordinated playback suspensions isolate one participant from the group during interruptions or manual catch-up, automatically rejoining on `suspension.end()`.
- Content identity defaults to URL matching; use `AVPlayerPlaybackCoordinatorDelegate` for cached or URL-less assets to ensure sync correctness.

---
_Source: WWDC21 Session 10225 page (abstract, chapter summaries, code samples, and resource links)._
