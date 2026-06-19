# Explore Dynamic Pre-rolls and Mid-rolls in HLS
**WWDC21 · Session 10140** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10140/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session introduces HLS Interstitials, a new first-party mechanism for inserting dynamic, late-bound advertisements and other interstitial content into HLS streams. HLS Interstitials replace the legacy approach of stitching ads into primary streams using `EXT-X-DISCONTINUITY` tags, which required static, pre-conditioned ad content at segment boundaries. With Interstitials, ads are kept as self-contained assets referenced by their own master playlist and scheduled onto the primary timeline using `EXT-X-DATERANGE` tags with a dedicated class attribute.

Two new AVFoundation objects—`AVPlayerInterstitialEventMonitor` and `AVPlayerInterstitialEventController`—enable client-side observation and programmatic scheduling of interstitial events. AVKit on tvOS provides built-in navigation restriction enforcement and progress UI out of the box.

## Key Topics
- **Problems with Legacy Ad Insertion:** Static stitching with `EXT-X-DISCONTINUITY` requires prebaked ad segments at segment boundaries, prevents late binding and rebinding, forces matching codecs and quality tiers, and is complex for live packagers.
- **HLS Interstitials Architecture:** Ads remain as independent assets (referenced by master playlist URI). Scheduled via `EXT-X-DATERANGE` with `CLASS="com.apple.hls.interstitial"`. No segment splitting or codec matching required. AVFoundation coordinates buffering between primary and interstitial players for seamless transitions.
- **DATERANGE Attributes:** `ID` (unique identifier), `START-DATE` (wall-clock ad start), `DURATION`, `X-ASSET-URI` (static ad master playlist), `X-ASSET-LIST` (URL to JSON with late-bound asset array), `X-RESUME-OFFSET` (0 = resume where left off; absent = rejoin at live edge), `X-PLAYOUT-LIMIT` (early return from live ad break), `X-RESTRICT` (jump / skip navigation restrictions).
- **Late Binding with X-ASSET-LIST:** The JSON endpoint is fetched at buffering time (not playlist insertion time), enabling real-time ad decisioning per viewer and per playthrough.
- **Navigation Restrictions and AVKit:** `X-RESTRICT=jump` prevents seeking past the ad; `X-RESTRICT=skip` prevents playing at non-normal rate. AVKit enforces these automatically on tvOS when `AVPlayerItem.translatesPlayerInterstitialEvents = true`. Other platforms must enforce in app code.
- **Client-Side Scheduling:** `AVPlayerInterstitialEventController` (subclass of Monitor) exposes a writable `events` array for programmatic ad insertion without DATERANGE tags in the playlist.
- **Buffering Model:** Pre-buffers ads sequentially before they are needed; primary buffering resumes after all ads in a pod are buffered. For live, playhead rejoins at live-edge offset after ad duration.

## APIs & Frameworks

**AVFoundation**
- `AVPlayerInterstitialEventMonitor` **[NEW]** – Observes interstitial scheduling and playback transitions on a primary player
  - `primaryPlayer: AVPlayer` – The primary content player
  - `interstitialPlayer: AVQueuePlayer` – The interstitial content player (read-only)
  - `events: [AVPlayerInterstitialEvent]` – Scheduled interstitial events (read-only on Monitor)
  - `currentEvent: AVPlayerInterstitialEvent?` – Currently playing interstitial, or nil
  - `AVPlayerInterstitialEventMonitor.eventsDidChangeNotification` **[NEW]** – Posted when event schedule changes
  - `AVPlayerInterstitialEventMonitor.currentEventDidChangeNotification` **[NEW]** – Posted on transition in/out of interstitials
- `AVPlayerInterstitialEventController: AVPlayerInterstitialEventMonitor` **[NEW]** – Adds programmatic scheduling
  - `events: [AVPlayerInterstitialEvent]!` – Read-write; set to schedule client-side interstitials
  - `cancelCurrentEvent(withResumptionOffset:)` **[NEW]** – Cancels the currently playing interstitial
- `AVPlayerInterstitialEvent` **[NEW]** – Describes a single interstitial event
  - `primaryItem: AVPlayerItem?` – The primary asset's player item
  - `identifier: String` – Unique event ID
  - `time: CMTime` – Start time in media timeline
  - `date: Date?` – Start time as wall-clock date
  - `templateItems: [AVPlayerItem]` – Ad pod items (used to create interstitial player items)
  - `restrictions: AVPlayerInterstitialEvent.Restrictions` – `.requiresPlaybackAtPreferredRateOnly`, `.requiresLinearPlayback`
  - `resumptionOffset: CMTime` – Primary resume offset; `.zero` for VOD
  - `playoutLimit: CMTime` – Maximum playout duration; `.invalid` for no limit
  - `userDefinedAttributes: [AnyHashable: Any]` – Custom DATERANGE attributes surfaced to app
- `AVPlayerItem.translatesPlayerInterstitialEvents: Bool` **[NEW]** – Enables AVKit timeline markers and restriction enforcement on tvOS

**HLS Playlist Tags**
- `EXT-X-PROGRAM-DATE-TIME` – Required for interstitial scheduling; provides wall-clock reference
- `EXT-X-DATERANGE CLASS="com.apple.hls.interstitial"` **[NEW]** – Schedules interstitial
  - `ID=` – Unique event identifier
  - `START-DATE=` – ISO 8601 start date/time
  - `DURATION=` – Ad duration in seconds
  - `X-ASSET-URI=` – Static asset master playlist URL
  - `X-ASSET-LIST=` – URL to JSON asset list for late binding
  - `X-RESUME-OFFSET=` – Primary resume offset (0 for VOD, omit for live)
  - `X-PLAYOUT-LIMIT=` – Maximum playout for early return
  - `X-RESTRICT=` – `JUMP`, `SKIP`, or both

**AVKit**
- `AVPlayerViewController` – Built-in interstitial UI: timeline markers, countdown timer, skip option (when unrestricted), navigation restriction enforcement on tvOS **[NEW behavior]**

## Code Highlights
Monitoring server-side interstitial playback:
```swift
let player = AVPlayer(url: movieURL) // movieURL has EXT-X-DATERANGE interstitial tags
let observer = AVPlayerInterstitialEventMonitor(primaryPlayer: player)
NotificationCenter.default.addObserver(
    forName: AVPlayerInterstitialEventMonitor.currentEventDidChangeNotification,
    object: observer,
    queue: .main) { _ in
        self.updateUI(observer.currentEvent, observer.interstitialPlayer)
}
```

Programmatic client-side ad pod scheduling:
```swift
let player = AVPlayer(url: movieURL)
let controller = AVPlayerInterstitialEventController(primaryPlayer: player)
let adPodTemplates = [AVPlayerItem(url: ad1URL), AVPlayerItem(url: ad2URL)]
let event = AVPlayerInterstitialEvent(
    primaryItem: player.currentItem,
    time: CMTime(seconds: 10, preferredTimescale: 1),
    templateItems: adPodTemplates,
    restrictions: [],
    resumptionOffset: .zero,
    playoutLimit: .invalid)
controller.events = [event]
player.currentItem?.translatesPlayerInterstitialEvents = true
let vc = AVPlayerViewController()
vc.player = player
player.play()
```

DATERANGE tag for a static ad (server-side HLS playlist):
```
#EXT-X-PROGRAM-DATE-TIME:2021-06-07T19:00:00Z
#EXT-X-DATERANGE:ID="ad1",CLASS="com.apple.hls.interstitial",START-DATE="2021-06-07T19:00:05Z",DURATION=30,X-ASSET-URI="https://example.com/ad1.m3u8",X-RESUME-OFFSET=0
```

## Takeaways
- HLS Interstitials decouple ads from primary segments entirely: no codec matching, no segment splitting, and no static stitching at packaging time.
- Use `X-ASSET-LIST` instead of `X-ASSET-URI` whenever late-binding to ad inventory is needed—the JSON is fetched at buffering time, enabling per-viewer ad decisioning without playlist regeneration.
- Set `resumptionOffset=0` for VOD streams to resume from the exact ad start point; omit it for live streams to automatically rejoin at the live edge.
- Always set `AVPlayerItem.translatesPlayerInterstitialEvents = true` before assigning to `AVPlayerViewController` to enable built-in timeline markers and restriction enforcement.

---
_Source: WWDC21 Session 10140 page (abstract, transcript, and code samples)._
