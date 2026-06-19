# Create a Great Video Playback Experience
**WWDC22 · Session 10147** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10147/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
This session presents a completely redesigned iOS/iPadOS system media player in AVKit that adopts the tvOS player's look and feel while adding touch-first interactions. The session walks through the design principles behind the new player, new visual intelligence (Live Text) capabilities for paused video frames, and new API for interstitials and playback speed controls that was previously tvOS-only.

The redesigned player strips away visual chrome to keep focus on content, introduces gesture-based timeline scrubbing, tap-to-play/pause, and pinch-to-zoom for aspect fill. The session frames these design decisions around three principles: intuitive interactions, tight system integration, and content-forward design.

The second half covers concrete new APIs: `allowsVideoFrameAnalysis` for Live Text in video frames, `AVInterstitialTimeRange` and related delegate methods ported from tvOS to iOS, and `AVPlaybackSpeed` for customizable playback rate controls across iOS, macOS, and tvOS.

## Key Topics

### Redesigned iOS/iPadOS System Player
- Removed slider knob; timeline is draggable from anywhere
- Pinch gesture replaces aspect fill control button
- Tap center to play/pause without showing controls
- Scroll anywhere over video to scrub timeline with momentum physics
- Portrait content support improved
- iPadOS adds keyboard, trackpad, mouse, and game controller support

### Content Metadata in the Player UI
Content title, subtitle, and description can be surfaced in the fullscreen UI by setting `AVMetadataItem` objects on `AVPlayerItem.externalMetadata`. These appear as an info panel accessible by tapping the title area.

### Live Text (Visual Analysis) in Video
`AVPlayerViewController.allowsVideoFrameAnalysis` enables Live Text on paused video frames — text can be selected and copied from within the player. Enabled by default for apps linking against the new SDKs; can be disabled for performance-critical contexts.

### Interstitials on iOS
`AVInterstitialTimeRange` and `AVPlayerItem.interstitialTimeRanges` are ported from tvOS to iOS. New delegate callbacks (`willPresentInterstitial`, `didPresentInterstitial`) notify the app when an interstitial begins and ends. Custom interstitial UI (e.g., skip buttons) should be added to `contentOverlayView`.

### Playback Speed Controls
New `AVPlaybackSpeed` class and `speeds` / `selectedSpeed` / `selectSpeed(_:)` API on both `AVPlayerViewController` and `AVPlayerView` allow customizing the playback rate menu shown in the transport controls. Enabled by default across all platforms.

## APIs & Frameworks

### AVKit — AVPlayerViewController / AVPlayerView
- `AVPlayerViewController.allowsVideoFrameAnalysis` **[NEW]** — toggles Live Text analysis on paused frames (iOS/macOS)
- `AVPlayerViewController.speeds` **[NEW]** — array of `AVPlaybackSpeed` defining the speed menu; set to `[]` to hide
- `AVPlayerViewController.selectedSpeed` **[NEW]** — the currently active `AVPlaybackSpeed`
- `AVPlayerViewController.selectSpeed(_:)` **[NEW]** — programmatically sets the active playback speed
- `AVPlayerView.allowsVideoFrameAnalysis` **[NEW]** — macOS equivalent
- `AVPlayerView.speeds` **[NEW]** — macOS equivalent
- `contentOverlayView` — existing property; custom UI (e.g., skip buttons) must be added here

### AVKit — AVPlaybackSpeed
- `AVPlaybackSpeed` **[NEW]** — represents a user-selectable playback speed option
- `AVPlaybackSpeed.rate: Float` **[NEW]** — the `AVPlayer` rate to set when selected
- `AVPlaybackSpeed.localizedName: String` **[NEW]** — accessibility name (e.g., "Two and a half times speed")
- `AVPlaybackSpeed.localizedNumericName: String` **[NEW]** — string to display in the speed menu UI
- `AVPlaybackSpeed.systemDefaultSpeeds: [AVPlaybackSpeed]` **[NEW]** — default list of system-provided speeds

### AVKit — Interstitials (iOS, ported from tvOS)
- `AVInterstitialTimeRange` **[NEW on iOS]** — represents a time range occupied by an interstitial
- `AVPlayerItem.interstitialTimeRanges: [AVInterstitialTimeRange]` **[NEW on iOS]** — read-only; auto-populated from HLS or `AVPlayerInterstitialEvent`
- `AVPlayerViewControllerDelegate.playerViewController(_:willPresent:)` **[NEW on iOS]** — called when an interstitial is about to play
- `AVPlayerViewControllerDelegate.playerViewController(_:didPresent:)` **[NEW on iOS]** — called after an interstitial ends

### AVFoundation — Metadata
- `AVMutableMetadataItem` — creates metadata items
- `AVMetadataIdentifier.commonIdentifierTitle` — title string
- `AVMetadataIdentifier.iTunesMetadataTrackSubTitle` — subtitle string
- `AVMetadataIdentifier.commonIdentifierDescription` — description/info paragraph
- `AVPlayerItem.externalMetadata` — array for overriding in-stream metadata

### AVFoundation — Interstitials
- `AVPlayerInterstitialEventController` — manages custom interstitial events
- `AVPlayerInterstitialEvent` — defines a single interstitial event
- `AVPlayerInterstitialEvent.restrictions` — e.g., `.requiresPlaybackAtPreferredRateForAdvancement`, `.constrainsSeekingForwardInPrimaryContent`
- `AVPlayerInterstitialEventController.cancelCurrentEvent(withResumptionOffset:)` — programmatically ends an interstitial

## Code Highlights

Setting content metadata for the fullscreen info panel:
```swift
let titleItem = AVMutableMetadataItem()
titleItem.identifier = .commonIdentifierTitle
titleItem.value = "My Movie"

let subtitleItem = AVMutableMetadataItem()
subtitleItem.identifier = .iTunesMetadataTrackSubTitle
subtitleItem.value = "Episode 1"

playerItem.externalMetadata = [titleItem, subtitleItem]
```

Adding a custom skip button for a pre-roll interstitial:
```swift
let eventController = AVPlayerInterstitialEventController(primaryPlayer: mediaPlayer)
let event = AVPlayerInterstitialEvent(primaryItem: interstitialItem, time: .zero)
event.restrictions = [.requiresPlaybackAtPreferredRateForAdvancement, .constrainsSeekingForwardInPrimaryContent]
eventController.events.append(event)

func playerViewController(_ vc: AVPlayerViewController, willPresent interstitial: AVInterstitialTimeRange) {
    showSkipButton(afterTime: 5.0) {
        eventController.cancelCurrentEvent(withResumptionOffset: .zero)
    }
}
```

Customizing playback speeds:
```swift
let newSpeed = AVPlaybackSpeed(rate: 2.5, localizedName: "Two and a half times speed")
playerViewController.speeds.append(newSpeed)

// Hide the speed menu entirely:
playerViewController.speeds = []
```

## Takeaways
- Adopting `AVPlayerViewController` gives apps the fully redesigned player, Picture in Picture, SharePlay, Live Text in video, and hardware input support with minimal code.
- `allowsVideoFrameAnalysis` enables Live Text on paused frames by default; disable it only for performance-critical contexts like scrolling video collections.
- `AVInterstitialTimeRange` and related delegate callbacks are now available on iOS, enabling proper interstitial handling parity with tvOS.
- `AVPlaybackSpeed` and the new speed menu APIs are cross-platform (iOS, macOS, tvOS) and enabled by default — customize or disable only when your use case requires it.

---
_Source: WWDC22 Session 10147 page (abstract, chapter summaries, code samples, and resource links)._
