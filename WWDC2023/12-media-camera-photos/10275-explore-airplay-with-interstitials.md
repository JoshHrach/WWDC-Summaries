# Explore AirPlay with Interstitials
**WWDC23 · Session 10275** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10275/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17

## Overview
This session covers how to use HLS Interstitials alongside AirPlay to deliver advertisements and secondary content within a primary video stream, with proper navigation restrictions enforced on both local playback and AirPlay receivers (including non-Apple smart TVs). The focus is on best practices for seamless transitions, enforcement of skip/jump restrictions, and how client application code can override server-scheduled interstitials to support different subscription plans.

The session demonstrates a concrete VOD example with three subscription tiers (ads with full restrictions, no ads, ads with partial skip ability) all served from the same HLS playlist, showing how to use both server-driven (`EXT-X-DATERANGE`/`X-RESTRICT`) and client-driven (`AVFoundation`) methods to configure the experience. It also covers receiver-side optimization (matching codec, frame rate, audio parameters) to reduce transition delays on the wide range of AirPlay hardware.

## Key Topics

- **HLS Interstitials overview** — Ads defined as separate assets on a primary content timeline; compatible with VOD and live content; supports late binding, rebinding, dynamic scheduling; built-in AirPlay and PiP support.
- **Navigation restrictions** — `Skip` restriction: prevents playing ad at non-normal rate; `Jump` restriction: prevents seeking out of current interstitial; applied via `X-RESTRICT` in HLS playlist or `restrictions` property of `AVPlayerInterstitialEvent`; automatically forwarded to AirPlay receivers with no code changes needed on the receiver side.
- **Optimizing AirPlay playback** — Match video codec (AVC/HEVC), frame rate, aspect ratio, audio codec, sampling frequency, channel layout between primary and interstitial assets to avoid decoder-switch delays on lower-end receivers.
- **Interstitial localization** — Matching audio/subtitle tracks between primary and interstitial triggers automatic track selection for both local and AirPlay playback; no code changes required.
- **Testing** — Build test suite across Apple TV, Mac receiver, and representative high-end/low-end non-Apple smart TVs; test capability tiers (4K HDR, Dolby Vision, stereo-only, SDR).
- **Subscription plan example** — Same HLS playlist supports Plan A (ads + full restrictions), Plan B (no ads via `automaticallyHandlesInterstitialEvents = false`), Plan C (first ad restricted, second ad skippable via client override).

## APIs & Frameworks

**AVFoundation**
- `AVPlayerInterstitialEvent` — represents a single scheduled interstitial event
- `AVPlayerInterstitialEvent.init(primaryItem:time:)` — creates an event at a specific primary-content time
- `AVPlayerInterstitialEvent.identifier` — string ID matching `EXT-X-DATERANGE` tag ID in HLS playlist
- `AVPlayerInterstitialEvent.templateItems` — array of `AVPlayerItem` for the interstitial asset(s)
- `AVPlayerInterstitialEvent.restrictions` **[key property]** — `AVPlayerInterstitialEvent.Restrictions` option set
  - `.requiresPlaybackAtPreferredRateForAdvancement` — Skip restriction: prevents fast-forward/scrub through ad
  - Jump restriction: prevents seeking out of interstitial
- `AVPlayerInterstitialEvent.resumptionOffset` — `CMTime` offset into primary content on interstitial completion
- `AVPlayerInterstitialEvent.copy()` — creates a mutable copy for overriding server-scheduled events
- `AVPlayerInterstitialEventController` — schedules and manages interstitial events for an `AVPlayer`
- `AVPlayerInterstitialEventController.init(primaryPlayer:)` — creates controller for a player
- `AVPlayerInterstitialEventController.events` — array of `AVPlayerInterstitialEvent`; setting this schedules or overrides events
- `AVPlayerInterstitialEventMonitor` — observe interstitial scheduling and progress (server-driven)
- `AVPlayerItem.automaticallyHandlesInterstitialEvents` — set to `false` to disable all server-scheduled interstitials (Plan B scenario)

**HLS Playlist (server-driven)**
- `EXT-X-DATERANGE` tag — schedules an interstitial event; contains `X-ASSET-URI`, `X-ASSET-LIST`
- `X-RESTRICT` attribute — values: `SKIP`, `JUMP`, or `SKIP,JUMP`; maps to `AVPlayerInterstitialEvent.restrictions`
- `X-RESUME-OFFSET` attribute — specifies resumption offset after interstitial (equivalent to `resumptionOffset`)

## Code Highlights

Client-driven interstitial scheduling with Skip restriction:
```swift
let player = AVPlayer(url: movieURL)
let controller = AVPlayerInterstitialEventController(primaryPlayer: player)

let ad1event = AVPlayerInterstitialEvent(
    primaryItem: player.currentItem!,
    time: CMTime(seconds: 5, preferredTimescale: 1))
ad1event.identifier = "ad1"
ad1event.templateItems = [AVPlayerItem(url: ad1Url)]
ad1event.restrictions = [.requiresPlaybackAtPreferredRateForAdvancement] // Skip restriction

controller.events = [ad1event]
```

Overriding server-scheduled restrictions for Plan C (skip second ad):
```swift
let ad1Event = controller.events[0]
let ad2Event = controller.events[1]

let newAd2 = ad2Event.copy() as! AVPlayerInterstitialEvent
newAd2.restrictions = []  // Clear skip restriction on ad2

controller.events = [ad1Event, newAd2]
```

Disabling all server-scheduled interstitials (Plan B — no ads):
```swift
playerItem.automaticallyHandlesInterstitialEvents = false
```

## Takeaways

- Navigation restrictions (`Skip`, `Jump`) set in either the HLS playlist (`X-RESTRICT`) or client code (`restrictions` property) are automatically enforced on AirPlay receivers — no extra code needed for the receiver side.
- Use `AVPlayerItem.automaticallyHandlesInterstitialEvents = false` to completely disable server-scheduled ads for premium tiers; use `AVPlayerInterstitialEventController.events` overrides for per-event control.
- Match video codec, frame rate, audio codec, and channel layout between primary and interstitial assets to minimize transition delays on lower-capability AirPlay receivers.
- Test AirPlay with at least one high-end and one low-end non-Apple receiver in addition to Apple TV, since hardware differences significantly affect transition behavior.

---
_Source: WWDC23 Session 10275 page (abstract, chapter summaries, code samples, and resource links)._
