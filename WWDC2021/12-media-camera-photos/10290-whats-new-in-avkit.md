# What's New in AVKit
**WWDC21 · Session 10290** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10290/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, Mac Catalyst

## Overview
AVKit in 2021 focuses on two themes: Picture in Picture (PiP) improvements and full-screen experience enhancements on macOS and Mac Catalyst. The headline PiP addition is support for `AVSampleBufferDisplayLayer` via a new content source API, which previously only worked with `AVPlayerLayer`. This brings native PiP UI to apps that manage their own media pipelines — video conferencing apps, custom decoders, and live streaming clients — with a new delegate protocol for playback state. A new `canStartPictureInPictureAutomaticallyFromInline` property also lets inline video automatically enter PiP when the user homes out.

On macOS and Mac Catalyst, full-screen video now truly occupies the entire screen rather than just the window, and a new delegate-based lifecycle API gives apps control over persisting the player view controller across navigation events so full-screen playback is not interrupted.

## Key Topics

### Automatic Inline-to-PiP Transition
The new `canStartPictureInPictureAutomaticallyFromInline` property on both `AVPlayerViewController` and `AVPictureInPictureController` enables inline video to automatically enter PiP when the user swipes to the Home screen — mirroring the behavior of YouTube and similar apps. This should only be enabled when the video is the user's primary focus.

### PiP for AVSampleBufferDisplayLayer
The new `AVPictureInPictureController.ContentSource` object accepts either an `AVPlayerLayer` or an `AVSampleBufferDisplayLayer`. For sample buffer-based content, the app implements `AVPictureInPictureSampleBufferPlaybackDelegate` to supply playback state (play/pause, time range, render size, paused status) and respond to user controls (play/pause button, skip buttons) from the PiP window.

### Full-Screen Enhancements on macOS and Mac Catalyst
Mac Catalyst apps automatically get true full-screen behavior in macOS Monterey. Apps using `AVPlayerViewController` should retain a strong reference to the controller during `willBeginFullScreenPresentation` and release it in `willEndFullScreenPresentation` to prevent full-screen playback from being interrupted by navigation. The async `playerViewControllerRestoreUserInterfaceForFullScreenExit` callback allows the app to navigate back to the original page. Equivalent `AVPlayerView` delegate methods handle the same lifecycle on native macOS.

## APIs & Frameworks

**AVKit — Picture in Picture**
- `AVPlayerViewController.canStartPictureInPictureAutomaticallyFromInline: Bool` **[NEW]**
- `AVPictureInPictureController.canStartPictureInPictureAutomaticallyFromInline: Bool` **[NEW]**
- `AVPictureInPictureController.ContentSource` **[NEW]** — init with `playerLayer:` or with `sampleBufferDisplayLayer:playbackDelegate:`
- `AVPictureInPictureController(contentSource:)` **[NEW]**
- `AVPictureInPictureSampleBufferPlaybackDelegate` protocol **[NEW]**
  - `pictureInPictureController(_:setPlaying:)` **[NEW]**
  - `pictureInPictureController(_:skipByInterval:completion:)` **[NEW]**
  - `pictureInPictureControllerTimeRangeForPlayback(_:) -> CMTimeRange` **[NEW]**
  - `pictureInPictureController(_:didTransitionToRenderSize:)` **[NEW]**
  - `pictureInPictureControllerIsPlaybackPaused(_:) -> Bool` **[NEW]**
- `AVPictureInPictureController.isPictureInPictureSupported() -> Bool` — existing
- `AVPictureInPictureController.startPictureInPicture()` — existing
- `AVPictureInPictureController.stopPictureInPicture()` — existing

**AVKit — Full Screen (iOS/Mac Catalyst)**
- `AVPlayerViewControllerDelegate.playerViewController(_:willBeginFullScreenPresentationWithAnimationCoordinator:)` — existing; use to retain a strong reference
- `AVPlayerViewControllerDelegate.playerViewController(_:willEndFullScreenPresentationWithAnimationCoordinator:)` — existing; use to release the retained reference
- `AVPlayerViewControllerDelegate.playerViewControllerRestoreUserInterfaceForFullScreenExit(_:) async -> Bool` **[NEW async variant]**

**AVKit — Full Screen (macOS)**
- `AVPlayerViewDelegate.playerViewWillEnterFullScreen(_:)` **[NEW]** — use to retain `AVPlayerView`
- `AVPlayerViewDelegate.playerViewWillExitFullScreen(_:)` **[NEW]** — use to release retained `AVPlayerView`
- `AVPlayerViewDelegate.playerViewRestoreUserInterfaceForFullScreenExit(_:) async -> Bool` **[NEW]**

## Code Highlights

Enabling automatic PiP from inline:
```swift
var canStartPictureInPictureAutomaticallyFromInline: Bool { get set }
```

Setting up PiP with `AVSampleBufferDisplayLayer`:
```swift
let contentSource = AVPictureInPictureController.ContentSource(
    sampleBufferDisplayLayer: sampleBufferDisplayLayer,
    playbackDelegate: self)
pictureInPictureController = AVPictureInPictureController(contentSource: contentSource)
```

Persisting full-screen player on iOS/Mac Catalyst:
```swift
func playerViewController(_ playerViewController: AVPlayerViewController,
    willBeginFullScreenPresentationWithAnimationCoordinator coordinator: ...) {
    coordinator.animate(alongsideTransition: nil) { _ in
        self.detachedPlayerViewController = playerViewController
    }
}
```

Restoring UI when exiting full screen (async):
```swift
func playerViewControllerRestoreUserInterfaceForFullScreenExit(
    _ playerViewController: AVPlayerViewController) async -> Bool {
    // Custom UI restoration logic
    return true
}
```

## Takeaways
- Set `canStartPictureInPictureAutomaticallyFromInline = true` to let inline video transition to PiP automatically when users home out — only when the video is the primary focus.
- `AVPictureInPictureController.ContentSource` with `AVSampleBufferDisplayLayer` brings native PiP to custom media pipelines; implement `AVPictureInPictureSampleBufferPlaybackDelegate` to drive the PiP UI.
- Mac Catalyst apps automatically get true full-screen video in macOS Monterey with no code changes required.
- Retain a strong reference to `AVPlayerViewController`/`AVPlayerView` inside `willBeginFullScreen` to allow users to navigate away without ending full-screen playback.

---
_Source: WWDC21 Session 10290 page (abstract, chapter summaries, code samples, and resource links)._
