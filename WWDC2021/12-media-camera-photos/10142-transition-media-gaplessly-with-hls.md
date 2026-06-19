# Transition Media Gaplessly with HLS
**WWDC21 · Session 10142** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10142/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session introduces gapless HLS item transitions in `AVQueuePlayer`, enabling seamless playback between consecutive HLS-backed `AVPlayerItem` instances. Previously, transitions between items could produce audible glitches, visual hiccups, or buffering indicators. With the new gapless transition capability, apps can deliver unbroken playback experiences for episodic video, music albums, workout sequences, instructional content, and live event continuity.

The key authoring requirement is that consecutive items must provide variants with matching audio formats — specifically matching FairPlay Streaming usage, audio codec (as specified by the `CODECS` attribute of the `EXT-X-STREAM-INF` tag), and channel count (from the `CHANNELS` attribute of the `EXT-X-MEDIA` tag). If matching variants are available at transition time, `AVQueuePlayer` automatically selects the matching variant in the next item to preserve audio continuity, prioritizing gapless transition over adaptive bitrate quality upgrades. Quality upgrades happen naturally once playback is established in the new item.

Apple Music itself uses this HLS gapless transition capability to seamlessly transition between songs.

## Key Topics

### Gapless Transition Use Cases
- Episodic TV: seamless end-of-episode to next-episode transitions
- Music albums: accurate track-to-track continuity replicating live or studio sequences
- Workout apps: dynamically stitch warm-up, exercise, and cooldown segments
- Instructional content: scene-to-scene transitions in interactive or programmatic sequences
- Linear programming: replication of broadcast continuity

### HLS Authoring Requirements for Gapless Playback
- **FairPlay Streaming**: must match across consecutive items
- **Audio codec**: specified by `CODECS` attribute in `EXT-X-STREAM-INF` (e.g., HE-AAC, AAC-LC); must match
- **Channel count**: specified by `CHANNELS` attribute in `EXT-X-MEDIA` tag; must match
- Sample rate and bit depth differences may inhibit gapless transition — keep consistent across items
- Follow CMAF (Common Media Application Format) authoring guidance: use edit lists to signal priming and remainder frames
- When matching variants are available, `AVQueuePlayer` selects them at transition; adaptive bitrate upgrades occur naturally once in the new item
- If no matching variant exists in the second item, gapless transition is not possible; player selects the best variant for current conditions

### AVQueuePlayer Integration
- `AVQueuePlayer`: queues multiple `AVPlayerItem` instances; handles gapless transitions automatically when authoring requirements are met
- `AVQueuePlayer.insert(_:after:)`: enqueue items in intended playback order
- No special API beyond correct enqueuing — gapless transitions happen automatically
- One asset, multiple segments: create multiple `AVPlayerItem` instances from a single `AVAsset`, define in/out points with `seekToTime` and `forwardPlaybackEndTime`
- `AVPlayerItem.seek(to:)`: define in point
- `AVPlayerItem.forwardPlaybackEndTime`: define out point (CMTime)

## APIs & Frameworks

- `AVFoundation` framework
- `AVQueuePlayer` **[ENHANCED]** — gapless HLS transitions now supported
- `AVPlayerItem`
- `AVPlayerItem.seek(to:)` — define in point for a segment
- `AVPlayerItem.forwardPlaybackEndTime` — define out point for a segment
- `AVQueuePlayer.insert(_:after:)` — enqueue items in playback order
- `AVAsset` — can back multiple `AVPlayerItem` instances for segment-based playback
- HLS playlist tags:
  - `EXT-X-STREAM-INF` with `CODECS` attribute (audio codec specification)
  - `EXT-X-MEDIA` with `CHANNELS` attribute (channel count specification)
  - `EXT-X-KEY` (FairPlay Streaming)
- CMAF (Common Media Application Format) — edit list for priming/remainder frames

## Code Highlights

Basic gapless playback of two HLS items:
```swift
let item1 = AVPlayerItem(url: url1)
let item2 = AVPlayerItem(url: url2)

let player = AVQueuePlayer()
player.insert(item1, after: nil)
player.insert(item2, after: item1)

player.play()
```

Creating custom segments from a single asset:
```swift
let asset = AVAsset(url: url)

let item1 = AVPlayerItem(asset: asset)
item1.seek(to: CMTime(seconds: 0, preferredTimescale: 600)) { _ in }
item1.forwardPlaybackEndTime = CMTime(seconds: 120, preferredTimescale: 600)

let item2 = AVPlayerItem(asset: asset)
item2.seek(to: CMTime(seconds: 120, preferredTimescale: 600)) { _ in }
item2.forwardPlaybackEndTime = CMTime(seconds: 300, preferredTimescale: 600)

let player = AVQueuePlayer(items: [item1, item2])
player.play()
```

## Takeaways

- Gapless HLS transitions require matching audio codec and channel count across consecutively enqueued items; no new API is needed on the client side beyond correct `AVQueuePlayer.insert(_:after:)` usage.
- When authoring content, follow CMAF guidance for edit lists (priming/remainder frames) and keep audio format consistent across all items in a gapless sequence.
- A single `AVAsset` can power multiple `AVPlayerItem` segments using `seek(to:)` and `forwardPlaybackEndTime`, enabling dynamic programmatic stitching of content.
- `AVQueuePlayer` prioritizes gapless transition over quality — adaptive bitrate upgrades occur naturally once playback is underway in the new item.

---
_Source: WWDC21 Session 10142 page (abstract, chapter summaries, code samples, and resource links)._
