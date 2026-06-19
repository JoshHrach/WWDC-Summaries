# Explore media metadata publishing and playback interactions
**WWDC22 · Session 110338** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110338/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
This session explains how to correctly publish Now Playing metadata (title, artwork, elapsed time, playback rate) so it surfaces in Control Center, the Lock Screen, watchOS Now Playing, and tvOS info overlays — and how to respond to remote commands from those surfaces and from devices like HomePod. The central new addition is `MPNowPlayingSession`, which was tvOS-only and is now available on iOS 16.

Three publishing approaches are covered: (1) automatic publishing via `MPNowPlayingSession` with `automaticallyPublishNowPlayingInfo = true` (observes the `AVPlayer` directly); (2) metadata publishing via `AVKit` using `AVPlayerItem.externalMetadata` (best for tvOS); and (3) fully manual publishing via `MPNowPlayingInfoCenter` (for apps that don't use `AVPlayer`).

## Key Topics

**Now Playing eligibility** — An app becomes the active Now Playing app when it registers at least one remote command handler and its `AVAudioSession` is configured with a non-mixable category. Without both conditions, the system will not surface the app's metadata.

**MPNowPlayingSession (iOS 16)** — Represents a discrete playback session tied to one or more `AVPlayer` instances. Multiple sessions can exist simultaneously (e.g., main player + Picture-in-Picture player); call `becomeActiveIfPossible` to promote the appropriate session as the active Now Playing session. Each session has its own `MPRemoteCommandCenter`.

**Remote command handling** — Register handlers on `MPRemoteCommandCenter` properties such as `playCommand`, `pauseCommand`, `skipForwardCommand`, and `skipBackwardCommand`. Commands that don't apply to the current content (e.g., skip forward for a live stream) should be disabled via `isEnabled = false`. `MPSkipIntervalCommandEvent.interval` carries the skip delta.

**Automatic publishing** — Set `MPNowPlayingSession.automaticallyPublishNowPlayingInfo = true`. The session observes `AVPlayer` for duration, elapsed time, playback rate, and play/pause state. Static metadata (title, artwork, ad time ranges) is set on `AVPlayerItem.nowPlayingInfo` using `MPMediaItemProperty` and `MPNowPlayingInfoProperty` keys.

**Ad time ranges** — `MPAdTimeRange(timeRange:)` marks ad segments baked into the asset so the system reports net content elapsed time and duration (excluding ads) in all Now Playing surfaces.

**AVKit metadata (tvOS)** — Set `AVPlayerItem.externalMetadata` with an array of `AVMutableMetadataItem` instances. Use `AVMetadataCommonIdentifierTitle` for the title string, `AVMetadataCommonIdentifierArtwork` for image data. Always set `extendedLanguageTag = "und"` for locale-independent metadata.

**Manual publishing** — Write directly to `MPNowPlayingInfoCenter.default().nowPlayingInfo` using a dictionary with `MPMediaItemPropertyTitle`, `MPMediaItemPropertyArtwork`, `MPMediaItemPropertyPlaybackDuration`, `MPNowPlayingInfoPropertyElapsedPlaybackTime`, and `MPNowPlayingInfoPropertyPlaybackRate`. Update on every non-linear time change, play/pause event, or content change. The system infers correct elapsed time from the last update — do not update periodically.

## APIs & Frameworks

### MediaPlayer
- `MPNowPlayingSession` — represents a playback session; previously tvOS-only, now **[NEW on iOS 16]**
- `MPNowPlayingSession.init(players:)` — accepts one or more `AVPlayer` instances
- `MPNowPlayingSession.automaticallyPublishNowPlayingInfo` — enables automatic metadata observation **[NEW]**
- `MPNowPlayingSession.becomeActiveIfPossible(completionHandler:)` — promotes this session as Now Playing
- `MPNowPlayingSession.remoteCommandCenter` — per-session `MPRemoteCommandCenter`
- `MPRemoteCommandCenter` — registry for remote command handlers
- `MPRemoteCommandCenter.playCommand` — play remote command
- `MPRemoteCommandCenter.pauseCommand` — pause remote command
- `MPRemoteCommandCenter.skipForwardCommand` — skip forward; `preferredIntervals: [Double]`
- `MPRemoteCommandCenter.skipBackwardCommand` — skip backward
- `MPRemoteCommand.addTarget(_:)` — register a command handler closure
- `MPRemoteCommand.isEnabled` — enable/disable a command dynamically
- `MPSkipIntervalCommandEvent` — event type for skip commands; `.interval` carries the skip seconds
- `MPRemoteCommandHandlerStatus` — `.success`, `.commandFailed`, etc.
- `AVPlayerItem.nowPlayingInfo` — dictionary for static metadata in automatic publishing **[NEW]**
- `MPAdTimeRange` — marks an ad segment within the asset timeline
- `MPAdTimeRange.init(timeRange:)` — takes a `CMTimeRange`
- `MPNowPlayingInfoPropertyAdTimeRanges` — key for ad time ranges in `nowPlayingInfo`
- `MPMediaItemArtwork` — wraps a `UIImage` for Now Playing artwork
- `MPMediaItemPropertyTitle` — key for content title string
- `MPMediaItemPropertyArtwork` — key for `MPMediaItemArtwork`
- `MPMediaItemPropertyPlaybackDuration` — key for total duration
- `MPNowPlayingInfoPropertyElapsedPlaybackTime` — key for current position
- `MPNowPlayingInfoPropertyPlaybackRate` — key for playback rate
- `MPNowPlayingInfoCenter.default()` — shared instance for manual publishing
- `MPNowPlayingInfoCenter.nowPlayingInfo` — dictionary written for manual mode

### AVFoundation / AVKit
- `AVPlayerItem.externalMetadata` — array of `AVMetadataItem` for AVKit metadata publishing
- `AVMutableMetadataItem` — mutable metadata item
- `AVMutableMetadataItem.identifier` — e.g., `.commonIdentifierTitle`, `.commonIdentifierArtwork`
- `AVMutableMetadataItem.value` — `NSCopying & NSObjectProtocol` (string for title, `NSData` for artwork)
- `AVMutableMetadataItem.dataType` — `kCMMetadataBaseDataType_JPEG`, `kCMMetadataBaseDataType_PNG`
- `AVMutableMetadataItem.extendedLanguageTag` — set to `"und"` for locale-independent metadata
- `AVMetadataIdentifier.commonIdentifierTitle`
- `AVMetadataIdentifier.commonIdentifierArtwork`
- `AVMetadataIdentifier.commonIdentifierDescription`

## Code Highlights

Automatic publishing with `MPNowPlayingSession`:
```swift
let artwork = MPMediaItemArtwork(image: image)
playerItem.nowPlayingInfo = [
    MPMediaItemPropertyTitle: "Magnificent",
    MPMediaItemPropertyArtwork: artwork
]
session = MPNowPlayingSession(players: [player])
session.automaticallyPublishNowPlayingInfo = true

session.remoteCommandCenter.playCommand.addTarget { _ in
    player.play(); return .success
}
session.remoteCommandCenter.pauseCommand.addTarget { _ in
    player.pause(); return .success
}
```

Skip forward with ad-aware interval:
```swift
session.remoteCommandCenter.skipForwardCommand.preferredIntervals = [15.0]
session.remoteCommandCenter.skipForwardCommand.addTarget { event in
    let skip = event as! MPSkipIntervalCommandEvent
    player.seek(to: CMTimeAdd(player.currentTime(),
                              CMTimeMakeWithSeconds(skip.interval, preferredTimescale: 1)))
    return .success
}
```

## Takeaways
- `MPNowPlayingSession` is now available on iOS 16; use it (with `automaticallyPublishNowPlayingInfo = true`) instead of manual `MPNowPlayingInfoCenter` updates to reduce code and prevent regressions.
- For apps with multiple concurrent playback sessions (PiP, multi-view), call `becomeActiveIfPossible` on the session that should surface in Lock Screen and Control Center.
- Set `extendedLanguageTag = "und"` on `AVMetadataItem` for title and description; using a specific locale code like `"en-us"` hides metadata on devices set to other languages.
- Update `MPNowPlayingInfoCenter.nowPlayingInfo` only on state changes (play/pause, scrub, new content); the system interpolates elapsed time automatically — no periodic polling needed.

---
_Source: WWDC22 Session 110338 page (abstract, chapter summaries, code samples, and resource links)._
