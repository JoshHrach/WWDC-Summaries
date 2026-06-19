# Discover Media Performance Metrics in AVFoundation
**WWDC24 · Session 10113** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10113/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, tvOS 18, visionOS 2

## Overview
AVFoundation gains a unified media performance metrics API in iOS 18: `AVMetrics`. Previously, developers relied on access logs, error logs, and AVPlayer notifications to diagnose HLS streaming issues in the field. The new API replaces those fragmented sources with a single publisher/subscriber model that delivers typed metric events in chronological order as playback progresses.

The session walks through two common problems — slow startup due to DRM key fetching and mid-stream stalls caused by HTTP 404 errors — and shows how the timeline of metric events makes the root cause immediately visible. A summary event at the end of each session provides key performance indicators (stall count, variant switch count) for fleet-level health monitoring.

## Key Topics

### What Are Events?
Every significant activity in an HLS playback session generates a typed `AVMetricEvent`:
- **Playlist events** — multi-variant playlist fetch, video/audio/subtitle playlist fetches
- **Media segment events** — per-segment fetch details including `NSURLSessionTaskMetrics`
- **Content key events** — DRM key request lifecycle
- **Likely-to-keep-up event** — marks the moment buffering is sufficient to start; includes startup time and references to preceding events
- **Stall events** — when the player runs out of buffer
- **Variant switch events** — quality switches during playback
- **Rate change / seek / error events** — from `AVPlayerItem`
- **Summary event** — end-of-session KPIs (stall count, switch count, etc.)

### Subscribing to Events
`AVPlayerItem` now conforms to `AVMetricEventStreamPublisher`. Apps obtain a typed async sequence for one or more event types, then merge them with `chronologicalMerge(with:)` to process events in timeline order. Objective-C developers use `AVMetricEventStream` with a subscriber object.

## APIs & Frameworks

**AVFoundation — AVMetrics (all NEW)**
- **`AVMetricEventStreamPublisher`** protocol **[NEW]**
  - `func metrics<MetricType: AVMetricEvent>(forType:) -> AVMetrics<MetricType>` **[NEW]**
  - `func allMetrics() -> AVMetrics<AVMetricEvent>` **[NEW]**
- **`AVMetrics<MetricType>`** — async sequence of metric events **[NEW]**
- `AVPlayerItem: AVMetricEventStreamPublisher` conformance **[NEW]**
- **`AVMetricPlayerItemLikelyToKeepUpEvent`** **[NEW]**
- **`AVMetricPlayerItemPlaybackSummaryEvent`** **[NEW]**
- `AVMetrics.chronologicalMerge(with:)` **[NEW]**
- **`AVMetricEventStream`** (Objective-C) **[NEW]**
  - `+eventStream`
  - `setSubscriber(_:queue:)`
  - `subscribeToMetricEvent(_:)`
  - `addPublisher(_:)`
- `AVMetricEvent` base protocol **[NEW]**
- Segment / playlist / content-key / stall / variant-switch event types **[NEW]**
- `NSURLSessionTaskMetrics` — referenced in media segment events (existing)

## Code Highlights

**Swift — subscribe to two event types:**
```swift
let ltkuMetrics = item.metrics(forType: AVMetricPlayerItemLikelyToKeepUpEvent.self)
let summaryMetrics = item.metrics(forType: AVMetricPlayerItemPlaybackSummaryEvent.self)

for await (metricEvent, publisher) in ltkuMetrics.chronologicalMerge(with: summaryMetrics) {
    // send metricEvent to analytics server
}
```

**Objective-C — event stream:**
```objc
AVMetricEventStream *stream = [AVMetricEventStream eventStream];
[stream setSubscriber:subscriber queue:mySerialQueue];
[stream subscribeToMetricEvent:[AVMetricPlayerItemLikelyToKeepUpEvent class]];
[stream subscribeToMetricEvent:[AVMetricPlayerItemPlaybackSummaryEvent class]];
[stream addPublisher:item];
```

## Takeaways
- Adopt `AVMetricEventStreamPublisher` on `AVPlayerItem` to get detailed HLS playback telemetry in iOS 18.
- Use `chronologicalMerge(with:)` to process multiple event types in timeline order without extra coordination.
- Mine segment events for `NSURLSessionTaskMetrics` to identify CDN-level issues (response headers, failure reasons).
- Send summary events to a backend analytics server to track stall count and variant switch count across your user base.

---
_Source: WWDC24 Session 10113 page (abstract, chapter summaries, code samples, and resource links)._
