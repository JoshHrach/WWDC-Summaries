# What's New in HLS Interstitials
**WWDC22 · Session 10145** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10145/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
HLS Interstitials allow ads and other interstitial content to be scheduled as separate assets on a primary HLS stream timeline using DATERANGE tags, without requiring discontinuity-tag stitching. Introduced in 2021, this year's update adds ad cueing options for pre/post-roll and play-once behavior, SNAP attributes for handling live clock drift at segment boundaries, and new query parameters for optimizing ad inventory and session tracking.

The `X-CUE` attribute enables placement of ads before or after primary content (PRE/POST), or restricts an interstitial to playing only once (ONCE). The `X-SNAP` attribute resolves the challenge of clock drift in live streams by snapping interstitial entry and exit points to the nearest segment boundary. Two new HLS query parameters (`HLS_start_offset` and `HLS_primary_id`) help ad servers deliver optimal inventory and track playback sessions.

`AVPlayerInterstitialEvent` is now mutable, separating object creation from property configuration, and has been updated to expose all new HLS attributes as first-class Swift properties.

## Key Topics

### Ad Cueing Options (X-CUE)
New `CUE` attribute values for DATERANGE tags:
- **PRE**: Ad plays before primary content begins (preroll). Ideal for live premium spots.
- **POST**: Ad plays after primary content ends (postroll). Useful for event stream end credits.
- **ONCE**: Ad plays only once; skipped if user seeks back. Useful for rating screens.

### SNAP Attributes for Live Streams
Clock drift between packager and encoder clocks can cause interstitials to start/end mid-segment. The `X-SNAP` attribute with `OUT` snaps the exit from primary content to the nearest segment boundary, and `IN` snaps the return to the nearest boundary after the ad. Should only be used for live content, not VOD.

### New HLS Query Parameters
- **HLS_start_offset**: Sent with Asset-list URL requests. For live join, specifies how far into the ad pod playback should begin, allowing servers to fill remaining time with highest-value inventory.
- **HLS_primary_id**: Associates interstitial requests with primary playback sessions for deduplication and session tracking at the ad server.

### Mutable AVPlayerInterstitialEvent
`AVPlayerInterstitialEvent` is now mutable. Create with just `primaryItem` and `time`, then set properties individually before assigning to the controller. Note: changes after assigning to controller require re-setting the event (controller makes a copy).

## APIs & Frameworks

**AVFoundation**
- `AVPlayerInterstitialEvent` — client-side interstitial scheduling
- `AVPlayerInterstitialEvent.cue` **[NEW]** — cueing options (`.pre`, `.post`, `.none`)
- `AVPlayerInterstitialEvent.willPlayOnce` **[NEW]** — play-once flag (maps to `CUE=ONCE`)
- `AVPlayerInterstitialEvent.alignsStartWithPrimarySegmentBoundary` **[NEW]** — SNAP-OUT behavior
- `AVPlayerInterstitialEvent.alignsResumptionWithPrimarySegmentBoundary` **[NEW]** — SNAP-IN behavior
- `AVPlayerInterstitialEvent.templateItems` — array of interstitial player items
- `AVPlayerInterstitialEvent.identifier` — event identifier
- `AVPlayerInterstitialEvent.restrictions` — navigation restrictions
- `AVPlayerInterstitialEvent.resumptionOffset` — where to resume primary content
- `AVPlayerInterstitialEvent.playoutLimit` — maximum playout duration
- `AVPlayerInterstitialEventController` — manages interstitial scheduling on an AVPlayer
- `AVPlayerItem.translatesPlayerInterstitialEvents` — controls whether HLS interstitials are translated

**HLS Playlist Attributes (new)**
- `X-CUE` DATERANGE attribute **[NEW]** — values: `PRE`, `POST`, `ONCE`
- `X-SNAP` DATERANGE attribute **[NEW]** — values: `OUT`, `IN` (for live clock drift)
- `HLS_start_offset` query parameter **[NEW]** — offset into asset list for live join
- `HLS_primary_id` query parameter **[NEW]** — primary session ID for ad request correlation

## Code Highlights

```swift
// Client-side interstitial event scheduling (now mutable)
let player = AVPlayer(url: movieURL)
let controller = AVPlayerInterstitialEventController(primaryPlayer: player)
let adPodTemplates = [AVPlayerItem(url: ad1URL), AVPlayerItem(url: ad2URL)]

let event = AVPlayerInterstitialEvent(
    primaryItem: player.currentItem,
    time: CMTime(seconds: 10, preferredTimescale: 1))
event.templateItems = adPodTemplates
event.identifier = "Ad1"
event.restrictions = []
event.resumptionOffset = .zero
event.playoutLimit = .invalid
event.cue = .none  // .pre, .post, or .none

controller.events = [event]
player.currentItem.translatesPlayerInterstitialEvents = true
```

## Takeaways

- Use `X-CUE=PRE/POST` for prerolls and postrolls in live/event streams; use `X-CUE=ONCE` for single-display interstitials like rating screens.
- Apply `X-SNAP=OUT,IN` in live streams to handle packager/encoder clock drift at segment boundaries — do not use for VOD.
- `HLS_start_offset` enables ad servers to optimize inventory during live join; `HLS_primary_id` enables session-based ad deduplication.
- `AVPlayerInterstitialEvent` is now mutable — create the object first, then configure properties before assigning to the controller; re-assign after changes.

---
_Source: WWDC22 Session 10145 page (abstract, chapter summaries, code samples, and resource links)._
