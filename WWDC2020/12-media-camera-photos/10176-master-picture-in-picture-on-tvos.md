# Master Picture in Picture on tvOS
**WWDC20 · Session 10176** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10176/)

_Platforms:_ tvOS 14

## Overview
Picture in Picture (PiP) comes to Apple TV in tvOS 14. Unlike iPadOS PiP, tvOS PiP supports simultaneous video playback — two videos from the same or different apps can play at the same time — and users can swap which video is full-screen and which is in the PiP window. The swipe-up gesture on the Siri Remote now belongs to `AVPlayerViewController` (since tvOS 13) and is required to reveal PiP controls, so apps that previously used swipe-up for custom overlays must migrate to `customOverlayViewController`.

The session covers two implementation paths: using `AVPlayerViewController` (the standard player, which handles PiP automatically) and using `AVPictureInPictureController` directly for custom playback UIs. Both require configuring the Xcode project with the Picture in Picture Background Mode capability and setting the audio session category to `.playback`. Apps using custom UIs must also tie each `AVPlayer` to an `MPNowPlayingSession` for Now Playing info center management.

## Key Topics

### Project Setup
- Add **Background Modes** capability in Xcode 12 and check **Picture in Picture**
- Set audio session category to `.playback` via `AVAudioSession.sharedInstance().setCategory(.playback)`
- Same steps as iPadOS PiP

### Standard Player: AVPlayerViewController
- PiP affordances (start, swap, stop buttons) appear automatically in the transport bar once the capability is configured
- Activate via swipe-up on the Siri Remote; swipe-up is reserved by `AVPlayerViewController` since tvOS 13
- Use `AVPlayerViewControllerDelegate` methods for PiP lifecycle (same API as iPadOS):
  - `playerViewControllerWillStartPictureInPicture(_:)` — PiP about to start
  - `playerViewControllerDidStartPictureInPicture(_:)` — PiP started
  - `playerViewControllerWillStopPictureInPicture(_:)` — PiP about to stop
  - `playerViewControllerDidStopPictureInPicture(_:)` — PiP stopped
  - `playerViewController(_:restoreUserInterfaceForPictureInPictureStopWithCompletionHandler:)` — restore UI quickly; avoid animations; if restoration is too slow, the player is terminated
- Do NOT install a custom swipe-up gesture recognizer when using `AVPlayerViewController`
- Use `customOverlayViewController` property (introduced in tvOS 13) for slide-up interactive overlays; accessed via swipe-up quick gesture or an on-screen button in the transport bar
- Do NOT publish Now Playing info via the shared `MPNowPlayingInfoCenter` in tvOS 14; AVKit handles it; augment metadata via `AVPlayerItem.externalMetadata` only

### Custom Player: AVPictureInPictureController
- `AVPictureInPictureController` is now available on tvOS **[NEW]**
- New tvOS-specific property: `canStopPictureInPicture` **[NEW]** — `Bool`; KVO-observable
  - `false` when full-screen player has no existing PiP → show a **Start PiP** affordance → call `startPictureInPicture()`
  - `true` when full-screen player has an existing PiP → show **Swap** and **Stop** affordances
  - Swap: call `startPictureInPicture()` on the full-screen player's controller (system handles the swap)
  - Stop: call `stopPictureInPicture()` only when user explicitly selects stop
- Observe `canStopPictureInPicture` via KVO; update UI whenever it changes
- Each `AVPlayer` must be tied to an `MPNowPlayingSession` **[NEW requirement for tvOS PiP]**
  - `MPNowPlayingSession(players:)` — creates a session for one or more players
  - `MPNowPlayingSession.nowPlayingInfoCenter.nowPlayingInfo` — set per-player Now Playing info
  - `MPNowPlayingSession.becomeActiveIfPossible()` — makes this session the active Now Playing source
  - The shared `MPNowPlayingInfoCenter` is no longer used in tvOS 14; switch fully to `MPNowPlayingSession`
  - Keep publishing Now Playing info even when your session is not currently displayed; system may show it at any time

### Swap Lifecycle
- **Same-app swap**: `willStart` for player going to PiP → `restoreUserInterface` for returning player → `willStop` for returning player → `didStart` + `didStop` simultaneously
- **Cross-app swap (your app going to PiP)**: `willStart` → your app is backgrounded → `didStart`
- **Cross-app swap (your PiP going full-screen)**: your app foregrounded → `restoreUserInterface` → `willStop` → `didStop`
- Restore UI quickly and without animations to avoid player termination

### Architecture Considerations
- The `AVPlayerViewControllerDelegate` must persist during PiP — it must NOT be part of the view hierarchy; use a separate delegate object
- Design the navigational hierarchy after a swap carefully: what screen should users see after dismissing a video?
- Do NOT pause playback when the app is deactivated if the video is in PiP (no need when using `AVPlayerViewController`)

## APIs & Frameworks

- **AVKit**
  - `AVPlayerViewController` — standard player; PiP UI is automatic with capability configured
  - `AVPlayerViewController.customOverlayViewController` **[introduced tvOS 13]** — slide-up overlay; replaces custom swipe-up gesture handlers
  - `AVPlayerViewControllerDelegate` — PiP lifecycle delegate (now on tvOS) **[NEW]**
  - `AVPlayerViewControllerDelegate.playerViewControllerWillStartPictureInPicture(_:)` **[NEW on tvOS]**
  - `AVPlayerViewControllerDelegate.playerViewControllerDidStartPictureInPicture(_:)` **[NEW on tvOS]**
  - `AVPlayerViewControllerDelegate.playerViewControllerWillStopPictureInPicture(_:)` **[NEW on tvOS]**
  - `AVPlayerViewControllerDelegate.playerViewControllerDidStopPictureInPicture(_:)` **[NEW on tvOS]**
  - `AVPlayerViewControllerDelegate.playerViewController(_:restoreUserInterfaceForPictureInPictureStopWithCompletionHandler:)` **[NEW on tvOS]**
  - `AVPictureInPictureController` — now available on tvOS **[NEW]**
  - `AVPictureInPictureController.canStopPictureInPicture` **[NEW, tvOS only]** — KVO-observable Bool
  - `AVPictureInPictureController.startPictureInPicture()` — starts PiP or initiates a swap
  - `AVPictureInPictureController.stopPictureInPicture()` — stops PiP
- **AVFoundation**
  - `AVAudioSession.sharedInstance().setCategory(.playback)` — required for PiP
  - `AVPlayerLayer(player:)` — used to create `AVPictureInPictureController` for custom UIs
  - `AVPlayerItem.externalMetadata` — array of `AVMetadataItem`; use to augment Now Playing metadata in standard player
- **MediaPlayer**
  - `MPNowPlayingSession(players:)` **[NEW]** — per-player Now Playing session; required for custom UI PiP on tvOS
  - `MPNowPlayingSession.nowPlayingInfoCenter` — per-session info center
  - `MPNowPlayingSession.becomeActiveIfPossible()` — activates this session as the system's Now Playing source
  - `MPNowPlayingInfoCenter.nowPlayingInfo` — dictionary of Now Playing metadata

## Code Highlights

Setting up the audio session:
```swift
let audioSession = AVAudioSession.sharedInstance()
try? audioSession.setCategory(.playback)
```

Observing `canStopPictureInPicture` for custom UI (KVO):
```swift
_ = pipController.observe(\.canStopPictureInPicture) { controller, _ in
    if controller.canStopPictureInPicture {
        // Show Swap and Stop affordances
    } else {
        // Show Start PiP affordance
    }
}
```

Tying an `AVPlayer` to an `MPNowPlayingSession` for custom UI:
```swift
final class CustomPlayerViewController: UIViewController {
    init(player: AVPlayer) {
        let playerLayer = AVPlayerLayer(player: player)
        pictureInPictureController = AVPictureInPictureController(playerLayer: playerLayer)
        nowPlayingSession = MPNowPlayingSession(players: [player])
    }

    func publishNowPlayingMetadata() {
        nowPlayingSession.nowPlayingInfoCenter.nowPlayingInfo = // your metadata dict
        nowPlayingSession.becomeActiveIfPossible()
    }
}
```

## Takeaways
- tvOS 14 PiP uniquely supports simultaneous dual-video playback and cross-app swapping — design your app's navigational hierarchy and delegate lifecycle with this in mind.
- The swipe-up gesture is reserved by `AVPlayerViewController` since tvOS 13; migrate custom swipe-up overlays to `customOverlayViewController` immediately when targeting tvOS 14.
- For custom player UIs, `AVPictureInPictureController.canStopPictureInPicture` (new, tvOS-only, KVO-observable) drives which PiP affordances to display; calling `startPictureInPicture()` handles both starting PiP and initiating a swap.
- The shared `MPNowPlayingInfoCenter` is no longer used in tvOS 14 — switch all Now Playing metadata to `MPNowPlayingSession` and keep publishing even when your session is not currently shown.

---
_Source: WWDC20 Session 10176 page (abstract, transcript, code samples, and resource links)._
