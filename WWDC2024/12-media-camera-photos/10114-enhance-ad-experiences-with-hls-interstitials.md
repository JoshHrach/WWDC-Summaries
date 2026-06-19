# Enhance Ad Experiences with HLS Interstitials
**WWDC24 · Session 10114** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10114/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, tvOS 18, visionOS 2

## Overview
HLS Interstitials allow ads and auxiliary content to be seamlessly spliced into primary HLS streams without modifying the primary content. This session introduces the **Integrated Timeline** API, a new data model that gives apps a unified, snapshot-based view of the playback timeline including interstitials, enabling custom transport UIs that reflect ad positions accurately. It also covers how to enable **SharePlay coordination** for interstitial content.

Integrated Timeline is particularly valuable for broadcast-style live streams where ad ranges need to be displayed accurately in the scrubber, and for VOD content where ads are marked as points on the timeline without affecting the total duration.

## Key Topics

### Interstitials Recap
HLS interstitials schedule ads via `X-DATERANGE` tags in the primary playlist. They avoid the need to condition ad content with primary content, support late-binding ad decisioning, and can be used for show promos, studio banners, and recap segments — not just traditional ads.

### Integrated Timeline
`AVPlayerItemIntegratedTimeline` exposes `currentSnapshot` — an `AVPlayerItemIntegratedTimelineSnapshot` that provides a consistent, immutable view of all segments at a moment in time. Each `AVPlayerItemSegment` describes a region of the timeline (primary or interstitial), its `timeMapping.target` range, its `segmentType`, and optionally the `AVPlayerInterstitialEvent` behind it.

Three timeline occupancy modes: `.singlePoint` (ad appears as a dot, playhead pauses during playback), `.fill` (ad occupies a range in the scrubber), and fill with `supplementsPrimaryContent = true` (range but indistinguishable from primary in UI).

`snapshotsOutOfSyncNotification` fires when the timeline changes, with a reason (`.segmentsChanged` or `.currentSegmentChanged`) to drive targeted UI updates.

### SharePlay with Interstitials
Set `event.contentMayVary = false` to signal that an interstitial is static across all participants, enabling coordinated playback of the interstitial in a SharePlay session.

### HLS Playlist Syntax
New `DateRange` attributes: `X-TIMELINE-OCCUPIES` (POINT or RANGE) and `X-TIMELINE-STYLE` (HIGHLIGHT or PRIMARY). `X-CONTENT-MAY-VARY=NO` enables SharePlay coordination from the server side.

## APIs & Frameworks

**AVFoundation**
- `AVPlayerItem.integratedTimeline: AVPlayerItemIntegratedTimeline` **[NEW]**
- `AVPlayerItemIntegratedTimeline` **[NEW]**
  - `.currentSnapshot: AVPlayerItemIntegratedTimelineSnapshot`
  - `snapshotsOutOfSyncNotification: NSNotification.Name` **[NEW]**
  - `snapshotsOutOfSyncReasonKey` in userInfo **[NEW]**
- `AVPlayerItemIntegratedTimelineSnapshot` **[NEW]**
  - `.segments: [AVPlayerItemSegment]`
  - `.duration: CMTime`
  - `.currentTime: CMTime`
- `AVPlayerItemSegment` **[NEW]**
  - `.segmentType: AVPlayerItemSegment.SegmentType` (`.primary`, `.interstitial`)
  - `.timeMapping: CMTimeMapping` (`.target` is the timeline range)
  - `.interstitialEvent: AVPlayerInterstitialEvent?`
- `AVPlayerInterstitialEvent` — new properties **[NEW]**
  - `.timelineOccupancy: AVPlayerInterstitialEvent.TimelineOccupancy` **[NEW]** (`.singlePoint`, `.fill`)
  - `.plannedDuration: CMTime` **[NEW]**
  - `.supplementsPrimaryContent: Bool` **[NEW]**
  - `.contentMayVary: Bool` **[NEW]**
- `AVPlayerIntegratedTimelineSnapshotsOutOfSyncReason` **[NEW]**
  - `.segmentsChanged`, `.currentSegmentChanged`
- HLS `DateRange` attributes: `X-TIMELINE-OCCUPIES`, `X-TIMELINE-STYLE`, `X-CONTENT-MAY-VARY` **[NEW]**

## Code Highlights

```swift
// Create a fill interstitial
let fillEvent = AVPlayerInterstitialEvent(primaryItem: playerItem, time: ten)
fillEvent.timelineOccupancy = .fill
fillEvent.plannedDuration = CMTime(value: 15, timescale: 1)

// Draw transport bar from snapshot
let snapshot = item.integratedTimeline.currentSnapshot
drawUISlider(start: .zero, duration: snapshot.duration, currentPosition: snapshot.currentTime)

// Highlight fill interstitials
let fillSegments = snapshot.segments.filter {
    $0.segmentType == .interstitial &&
    $0.interstitialEvent?.timelineOccupancy == .fill &&
    !($0.interstitialEvent?.supplementsPrimaryContent ?? false)
}

// Enable SharePlay coordination
event.contentMayVary = false
```

## Takeaways
- Use `AVPlayerItemIntegratedTimeline` to build a transport bar that accurately reflects ad positions as points or ranges.
- Subscribe to `snapshotsOutOfSyncNotification` to redraw only when the timeline structure changes, not on every frame.
- Set `plannedDuration` on fill interstitials so the scrubber shows the correct duration before the actual ad duration is known.
- Set `contentMayVary = false` to enable SharePlay synchronization for static interstitials shared across all participants.

---
_Source: WWDC24 Session 10114 page (abstract, chapter summaries, code samples, and resource links)._
