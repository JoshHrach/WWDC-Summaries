# Now Playing and Remote Commands on tvOS
**WWDC17 · Session 251** · [Watch](https://developer.apple.com/videos/play/wwdc2017/251/)

_Platforms:_ tvOS 11

## Overview
Accurate, complete Now Playing metadata and proper Remote Command handling are essential for a polished tvOS media experience. This session covers three integration paths for providing Now Playing information — TVML's `MediaItem` JavaScript object, AVFoundation's `AVMutableMetadataItem` for apps using `AVPlayerViewController`, and `MPNowPlayingInfoCenter` for apps using custom players or audio-only content — and shows how to handle external playback commands from Siri, the Siri Remote, and the iOS TV Remote app using `MPRemoteCommandCenter`.

Now Playing metadata appears in four places on tvOS: inline in `AVPlayerViewController` overlaid on video, in the iOS TV Remote app with playback controls, as an on-screen notification badge when audio metadata changes, and as a full-screen idle display that prominently shows artwork. Getting the metadata right across all of these surfaces requires understanding which identifiers map to which display slots in each interface.

The session explains the correct `extendedLanguageTag` value (`"und"` for undefined/universal), how to provide artwork via `MPMediaItemArtwork`'s size-based image callback, and when to update Now Playing info (on item change, seek, rate change, play/pause). Remote command handling covers the full lifecycle including returning appropriate success/failure values and updating Now Playing info after seek operations.

## Key Topics
- **Now Playing display surfaces** — `AVPlayerViewController` overlay, iOS TV Remote app, audio badge notification, full-screen idle display
- **TVML `MediaItem`** — `title`, `subtitle`, `description`, `artworkImageURL`, content rating, explicit flag
- **`AVMutableMetadataItem`** — `identifier` (e.g., `AVMetadataCommonIdentifierTitle`), `value`, `extendedLanguageTag` (`"und"` for worldwide display), `dataType` (for artwork JPEG/PNG)
- **`extendedLanguageTag: "und"`** — critical; language-specific tags (like `"en"`) hide metadata from users whose locale doesn't match
- **`AVPlayerItem.externalMetadata`** — array of `AVMetadataItem` objects attached to the player item
- **`AVMetadataCommonIdentifierCreationDate`** — release date as ISO-formatted string (year displayed in `AVPlayerViewController`)
- **`AVMetadataCommonIdentifierArtwork`** — artwork as raw JPEG/PNG bytes with `dataType` indicating format
- **`MPNowPlayingInfoCenter`** — singleton; `nowPlayingInfo` dictionary; keys: `MPMediaItemPropertyTitle`, `MPMediaItemPropertyArtist`, `MPMediaItemPropertyAlbumTitle`, `MPMediaItemPropertyArtwork`, `MPNowPlayingInfoPropertyMediaType`, `MPMediaItemPropertyPlaybackDuration`, `MPNowPlayingInfoPropertyElapsedPlaybackTime`, `MPNowPlayingInfoPropertyPlaybackRate`
- **When to update Now Playing info** — on item change, metadata change, seek, rate change, playback start/stop; not required every second
- **`MPMediaItemArtwork`** — initialized with native image size; block called with requested size to return closest available `UIImage`
- **`MPRemoteCommandCenter`** — singleton; per-command properties; register handler block or target-action; `isEnabled` for temporary unavailability; handler must return `MPRemoteCommandHandlerStatus`
- **Remote command types** — `playCommand`, `pauseCommand`, `togglePlayPauseCommand`, `skipForwardCommand`, `skipBackwardCommand`, `changePlaybackPositionCommand`, `nextTrackCommand`, `previousTrackCommand`
- **`MPSkipIntervalCommand`** — `preferredIntervals` expresses preference; `MPSkipIntervalCommandEvent.interval` provides actual interval from system
- **Post-seek Now Playing update** — must update `MPNowPlayingInfoPropertyElapsedPlaybackTime` after seek using `CMTimeGetSeconds(player.currentTime())`

## APIs & Frameworks

### Media Player
- **`MPNowPlayingInfoCenter.default()`** — singleton; `nowPlayingInfo: [String: Any]?`
- **`MPNowPlayingInfoPropertyMediaType`** — `.video` or `.audio`
- **`MPNowPlayingInfoPropertyElapsedPlaybackTime`** — `Double` (seconds); update on seek and play/pause
- **`MPMediaItemPropertyPlaybackDuration`** — total duration in seconds
- **`MPNowPlayingInfoPropertyPlaybackRate`** — `1.0` = playing, `0.0` = paused
- **`MPMediaItemArtwork(boundsSize:requestHandler:)`** — artwork object; block returns `UIImage` for requested `CGSize`
- **`MPRemoteCommandCenter.shared()`** — singleton; properties for each command type
- **`MPRemoteCommandCenter.skipBackwardCommand`** / **`skipForwardCommand`** — `MPSkipIntervalCommand`; `preferredIntervals: [NSNumber]`
- **`MPRemoteCommandCenter.changePlaybackPositionCommand`** — `MPChangePlaybackPositionCommand`; `MPChangePlaybackPositionCommandEvent.positionTime`
- **`MPRemoteCommandHandlerStatus`** — `.success`, `.commandFailed`, `.noSuchContent`, `.noActionableNowPlayingItem`, `.deviceNotFound`

### AVFoundation
- **`AVMutableMetadataItem`** — `identifier`, `value`, `extendedLanguageTag`, `dataType`
- **`AVMetadataCommonIdentifierTitle`** — title string
- **`AVMetadataCommonIdentifierDescription`** — description string
- **`AVMetadataCommonIdentifierCreationDate`** — release date as ISO 8601 string (format: `"yyyy-MM-dd'T'HH:mm:ssZZZZZ"`)
- **`AVMetadataCommonIdentifierArtwork`** — artwork as `NSData`; `dataType` = `"public.jpeg"` or `"public.png"`
- **`AVMetadataIdentifierQuickTimeMetadataArtist`** — artist name (for audio)
- **`AVMetadataIdentifierQuickTimeMetadataAlbumName`** — album name (for audio)
- **`AVPlayerItem.externalMetadata`** — `[AVMetadataItem]`; set before playback begins

### TVMLKit
- **`MediaItem` JavaScript object** — `mediaItemURL`, `title`, `description`, `artworkImageURL`, `contentRatingDomain`, `contentRatingRanking`, `isExplicit`

### Core Media
- **`CMTimeGetSeconds(_:)`** — converts `CMTime` to `Double` seconds for `MPNowPlayingInfoCenter`

## Code Highlights

```swift
// AVFoundation metadata item for title
let titleItem = AVMutableMetadataItem()
titleItem.identifier = .commonIdentifierTitle
titleItem.value = "WWDC 2017 Talk" as NSString
titleItem.extendedLanguageTag = "und"  // show to ALL users

// Artwork metadata
let artworkItem = AVMutableMetadataItem()
artworkItem.identifier = .commonIdentifierArtwork
artworkItem.value = jpegData as NSData
artworkItem.dataType = "public.jpeg"
artworkItem.extendedLanguageTag = "und"

playerItem.externalMetadata = [titleItem, artworkItem]

// MPNowPlayingInfoCenter
let artwork = MPMediaItemArtwork(boundsSize: nativeSize) { size in
    return imageForSize(size)  // return closest pre-sized UIImage
}
MPNowPlayingInfoCenter.default().nowPlayingInfo = [
    MPMediaItemPropertyTitle: "My Podcast",
    MPMediaItemPropertyArtwork: artwork,
    MPNowPlayingInfoPropertyMediaType: MPNowPlayingInfoMediaType.audio.rawValue,
    MPNowPlayingInfoPropertyElapsedPlaybackTime: CMTimeGetSeconds(player.currentTime()),
    MPMediaItemPropertyPlaybackDuration: 3600.0,
    MPNowPlayingInfoPropertyPlaybackRate: 1.0
]

// Remote command handling
let center = MPRemoteCommandCenter.shared()
center.skipBackwardCommand.preferredIntervals = [10]
center.skipBackwardCommand.addTarget { event in
    guard let e = event as? MPSkipIntervalCommandEvent else { return .commandFailed }
    let newTime = CMTimeSubtract(player.currentTime(), CMTimeMakeWithSeconds(e.interval, 1))
    player.seek(to: newTime) { _ in self.updateNowPlaying() }
    return .success
}
```

## Takeaways
- Always use `extendedLanguageTag = "und"` on `AVMutableMetadataItem`; language-specific tags silently hide metadata from users in other locales.
- Update `MPNowPlayingInfoCenter.nowPlayingInfo` on seek, rate change, and item change — not continuously — using `CMTimeGetSeconds(player.currentTime())` for accurate elapsed time.
- `MPRemoteCommandCenter` handlers must return a `MPRemoteCommandHandlerStatus`; commands are implicitly enabled when a handler is registered, disabled if no handler is provided.
- For audio-only or custom video player apps, `MPNowPlayingInfoCenter` is the only way to populate the TV Remote app's scrubber and the tvOS idle display; `AVPlayerViewController` handles this automatically for standard video playback.

---
_Source: WWDC17 Session 251 page (abstract, chapter summaries, code samples, and resource links)._
