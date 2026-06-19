# Delivering Intuitive Media Playback with AVKit
**WWDC19 · Session 503** · [Watch](https://developer.apple.com/videos/play/wwdc2019/503/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15 (iPad apps on Mac), tvOS 13

## Overview
AVKit is a high-level media playback UI framework built on top of AVFoundation and CoreMedia. This session covers what is new in `AVPlayerViewController` for iOS 13 and tvOS 13, plus best practices for full-screen playback, embedded inline playback, Picture-in-Picture, and the newly supported iPad apps on Mac.

The iOS section focuses on new full-screen presentation state callbacks via `AVPlayerViewControllerDelegate`, first-class support for iPad apps on Mac (Touch Bar, keyboard, PiP — all free with no code changes), external metadata for Now Playing info, and guidelines for correctly using custom controls alongside `AVPlayerViewController`. The tvOS section introduces custom interactive overlays (swipe-up), live-stream channel flipping, and automatic parental content restriction enforcement.

## Key Topics

- **Full-screen presentation callbacks (iOS 13)** — Two new `AVPlayerViewControllerDelegate` methods inform about begin/end of full-screen transitions; `UIViewControllerTransitionCoordinator` reveals whether an interactive dismissal was completed or cancelled. **[NEW]**
- **AVPlayerViewController on iPad apps on Mac** — Zero additional code required; Touch Bar support, keyboard shortcuts, and Picture-in-Picture all work automatically. **[NEW]**
- **External metadata (iOS 13)** — `AVPlayerItem.externalMetadata` supplements or replaces baked-in metadata for Now Playing info, Lock Screen artwork, title, and (on tvOS) parental content ratings. Previously tvOS-only; now on iOS too. **[NEW]**
- **Custom controls with `showsPlaybackControls = false`** — Present `AVPlayerViewController` modally and place custom controls in `contentOverlayView`; interactive dismissals, landscape support, keyboard, Touch Bar, and Now Playing info still work.
- **Full-screen best practices** — Always present modally with `.automatic` style (resolves to `.fullScreen` for AVPlayerViewController); do not override `videoGravity` for full-screen (let AVKit handle zoom/unzoom); observe `AVPlayer.status` / `AVPlayerItem.status` via KVO for error recovery; configure audio session for playback.
- **Embedded inline best practices** — Use `entersFullScreenWhenPlaybackBegins` and `exitsFullScreenWhenPlaybackEnds` for automatic full-screen transitions; observe `isReadyForDisplay` (KVO) to remove poster images when the first video frame is available; adopt `UIViewController` containment APIs.
- **Picture-in-Picture** — Do not pause video when the app enters background (PiP may be starting); wait for `applicationState == .background` or window scene background state if you must pause; use `AVPlayerViewControllerDelegate` to restore UI when user returns from PiP.
- **tvOS: Custom interactive overlays** — Set `customOverlayViewController` on `AVPlayerViewController`; system handles swipe-up/swipe-down presentation and dismissal animation. **[NEW]**
- **tvOS: Live channel flipping** — Implement `skipToNextChannel` / `skipToPreviousChannel` delegate methods; provide a `nextChannelInterstitialViewController` / `previousChannelInterstitialViewController` for the between-channel loading screen. **[NEW]**
- **tvOS: Parental content restrictions** — Provide `iTunesMetadataContentRating` via `externalMetadata`; `AVPlayerViewController` automatically prompts for passcode when content exceeds the user's restriction setting; use `requestPlaybackRestrictionsAuthorization` to prompt earlier. **[NEW]**

## APIs & Frameworks

### AVKit — AVPlayerViewController
- `AVPlayerViewController` — primary media playback UI on iOS, tvOS, macOS (Macs with Catalyst)
- `AVPlayerViewController.player` — assign `AVPlayer` before presenting
- `AVPlayerViewController.showsPlaybackControls` — set `false` for fully custom UI
- `AVPlayerViewController.contentOverlayView` — place custom controls or poster images here
- `AVPlayerViewController.videoGravity` — `AVLayerVideoGravity`; best left unset for automatic behavior in full-screen
- `AVPlayerViewController.externalMetadata` — `[AVMetadataItem]`; supplements baked-in metadata **[NEW on iOS]**
- `AVPlayerViewController.isReadyForDisplay` — KVO-observable; `true` when first video frame is rendered
- `AVPlayerViewController.entersFullScreenWhenPlaybackBegins` — auto full-screen on play **[NEW]**
- `AVPlayerViewController.exitsFullScreenWhenPlaybackEnds` — auto exit full-screen at end **[NEW]**
- `AVPlayerViewController.allowsPictureInPicturePlayback` — enable PiP
- `AVPlayerViewController.customOverlayViewController` — swipe-up overlay on tvOS **[NEW]**

### AVKit — AVPlayerViewControllerDelegate (iOS 13)
- `playerViewController(_:willBeginFullScreenPresentationWithAnimationCoordinator:)` **[NEW]**
- `playerViewController(_:willEndFullScreenPresentationWithAnimationCoordinator:)` **[NEW]**
- `playerViewControllerWillStartPictureInPicture(_:)`
- `playerViewControllerDidStartPictureInPicture(_:)`
- `playerViewController(_:restoreUserInterfaceForPictureInPictureStopWithCompletionHandler:)`

### AVKit — AVPlayerViewControllerDelegate (tvOS 13)
- `playerViewController(_:skipToNextChannel:)` **[NEW]**
- `playerViewController(_:skipToPreviousChannel:)` **[NEW]**
- `playerViewController(_:nextChannelInterstitialViewController:)` **[NEW]**
- `playerViewController(_:previousChannelInterstitialViewController:)` **[NEW]**
- `playerViewController(_:requestPlaybackRestrictionsAuthorization:)` **[NEW]**

### AVFoundation
- `AVPlayer` — manages playback state
- `AVPlayerItem` — represents a single media item; `.externalMetadata: [AVMetadataItem]`
- `AVPlayerItem.status` — `.readyToPlay`, `.failed`; observe via KVO
- `AVPlayer.status` — observe via KVO; check `.error` on failure
- `AVMetadataItem` — `identifier: .iTunesMetadataContentRating`, `value`, `extendedLanguageTag: "und"`
- `AVPlayer.allowsExternalPlayback` — set `false` for splash screen videos
- `AVPlayer.preventsDisplaySleepDuringVideoPlayback` — prevents sleep during mirroring

### AVKit — Other
- `AVPlayerView` (AppKit) — macOS playback view
- `AVRoutePickerView` — wireless route picker for custom playback UIs

### AVAudioSession
- `AVAudioSession.Category.playback`
- `AVAudioSession.silenceSecondaryAudioHintNotification` — mute splash screen audio when another app has primary audio

## Code Highlights

Full-screen presentation state tracking:

```swift
func playerViewController(
    _ pvc: AVPlayerViewController,
    willBeginFullScreenPresentationWithAnimationCoordinator coordinator: UIViewControllerTransitionCoordinator
) {
    coordinator.animate(alongsideTransition: nil) { context in
        if !context.isCancelled {
            // Full-screen presentation confirmed; take strong ref if needed
            self.retainedPlayerVC = pvc
        }
    }
}

func playerViewController(
    _ pvc: AVPlayerViewController,
    willEndFullScreenPresentationWithAnimationCoordinator coordinator: UIViewControllerTransitionCoordinator
) {
    // Re-insert pvc into scroll view before animation ends
    restorePlayerInScrollView(pvc)
}
```

External metadata (title + artwork):

```swift
let titleItem = AVMutableMetadataItem()
titleItem.identifier = .commonIdentifierTitle
titleItem.value = "Episode Title" as NSString
titleItem.extendedLanguageTag = "und"

let artworkItem = AVMutableMetadataItem()
artworkItem.identifier = .commonIdentifierArtwork
artworkItem.value = UIImage(named: "poster")!.pngData()! as NSData
artworkItem.extendedLanguageTag = "und"

playerItem.externalMetadata = [titleItem, artworkItem]
```

tvOS content rating:

```swift
let ratingItem = AVMutableMetadataItem()
ratingItem.identifier = .iTunesMetadataContentRating
ratingItem.value = "PG-13" as NSString
ratingItem.extendedLanguageTag = "und"
playerItem.externalMetadata = [ratingItem]
```

## Takeaways

- Present `AVPlayerViewController` modally with `.automatic` style (never as a child) for full-screen playback; the system handles rotation, zoom, interactive dismissal, and status-bar appearance automatically.
- Use the new iOS 13 full-screen delegate callbacks to maintain strong references and restore the embedded player's position in a scroll view without the user noticing.
- Set `externalMetadata` on `AVPlayerItem` instead of managing `MPNowPlayingInfoCenter` manually — AVKit publishes Now Playing info for you on both iOS and tvOS.
- On tvOS, migrate any custom swipe-up overlays to `customOverlayViewController` for a consistent user experience, and always provide `iTunesMetadataContentRating` metadata to support parental content restrictions.

---
_Source: WWDC19 Session 503 page (abstract, full transcript, and resource links)._
