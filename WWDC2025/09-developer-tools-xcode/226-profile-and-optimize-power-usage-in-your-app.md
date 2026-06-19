# Profile and optimize power usage in your app
**WWDC25 · Session 226** · [Watch](https://developer.apple.com/videos/play/wwdc2025/226/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26

## Overview
Power efficiency directly impacts user satisfaction and engagement: apps that drain the battery quickly are deleted. This session introduces significant new capabilities in the Power Profiler instrument, including on-device power tracing that works entirely without a Xcode connection — enabling power profiling of scenarios impossible to reproduce at a desk (CarPlay, outdoor AR, background processing that takes hours to manifest).

The session also demonstrates how to use Power Profiler to compare the power impact of two competing implementations before shipping, turning architectural decisions into data-driven choices rather than intuition. A fictional video streaming app called Destination Video serves as the running example throughout.

## Key Topics

### Power Profiler in Instruments
The Power Profiler instrument records both system-level power metrics and per-app power impact scores. Key tracks:
- **System power usage lane** — overall device energy consumption (% per hour)
- **CPU power impact lane** — per-app CPU energy score (dimensionless, relative; higher = worse)
- **GPU power impact lane** — GPU energy score
- **Display power impact lane** — display-related energy
- **Network power impact lane** — radio energy

CPU Profiler can be combined in the same recording to correlate power spikes with specific call-tree nodes. The workflow: profile → identify spike → find heavy call tree → fix code → re-profile.

### Debug Example: VStack → LazyVStack
The session demonstrates a real regression: adding a Library pane to a video app caused a large CPU power spike and a hang when opening the pane. Power Profiler shows the spike; CPU Profiler narrows it to `LibraryThumbnailView`, which uses `VStack` to materialize all thumbnails at once. Replacing `VStack` with `LazyVStack` drops the CPU power impact score from 21 to 4.3 and eliminates the hang.

### On-Device Power Profiling (New)
Power Profiler is now available on-device without Xcode connected. Setup:
1. Enable **Developer Mode** in Settings (requires prior Xcode pairing).
2. Enable **Performance Trace** in Developer settings.
3. Toggle **Power Profiler** on and select the target app.
4. Use **Control Center** to start/stop recording.
5. Share the `.trace` file and open it in Instruments on Mac.

The trace includes system-level metrics, per-app power impact, and a lower-sample-rate Time Profiler. This mode is used to diagnose issues that only manifest during real-world usage — the example is a location-based recommendation feature that fires on every location update during commuting but never triggers at a developer's desk.

### Root Cause: Location-Triggered JSON Parsing
The on-device trace reveals a periodic CPU spike pattern driven by `videoSuggestionsForLocation()`. The function reloads and JSON-parses a large `RecommendationRules` file on every location update. The fix: lazy-load and cache the parsed rules once on first call. This eliminates the per-update I/O and JSON parsing cost.

### Comparing Implementations
Power Profiler supports A/B comparison workflows: record a power trace for Approach 1, then for Approach 2 under equivalent conditions, and compare the per-app CPU power impact scores. The session recommends multiple runs per approach to account for thermal variation, device state differences, and background system activity.

### Proactive Tooling Ecosystem
The session catalogs the full power-optimization toolkit:
- **Xcode Energy Gauges** — live power indicator during development runs
- **Instruments** — deep-dive traces (connected and on-device)
- **XCTest** — automated performance regression detection in CI
- **Xcode Organizer** — aggregate energy reports from App Store users
- **MetricKit** — per-device energy diagnostics delivered to the app
- **App Store Connect API** — programmatic access to Organizer power data

## APIs & Frameworks

- **Power Profiler instrument** (Instruments) **[major update]** — per-app power impact lanes with Time Profiler correlation
- **On-device Power Profiling** **[NEW]** — Xcode-free trace recording via Control Center
- **Performance Trace** (Developer settings) **[NEW]** — on-device trace control
- **CPU Profiler instrument** (Instruments, existing) — call-tree profiler, used alongside Power Profiler
- **Xcode Energy Gauges** (existing) — live power feedback during debugging
- **MetricKit** (existing) — `MXEnergyPayload` field energy diagnostics
- **Xcode Organizer energy reports** (existing) — aggregated field data
- **App Store Connect API** (existing) — programmatic Organizer data access
- **SwiftUI `LazyVStack`** (existing) — on-demand view materialization (fix demonstrated in session)
- **`os_signpost`** (existing) — custom intervals visible in power trace timeline

## Code Highlights

```swift
// Fix: replace eager VStack with lazy loading
// Before — materializes all thumbnails immediately
VStack {
    ForEach(videos) { video in
        VideoCardView(video: video)  // creates thumbnail for every video
    }
}

// After — only creates visible views
LazyVStack {
    ForEach(videos) { video in
        VideoCardView(video: video)
    }
}
```

```swift
// Fix: cache expensive computation instead of repeating on every location update
class RecommendationService {
    private var cachedRules: RecommendationRuleMap?

    func videoSuggestionsForLocation(_ location: CLLocation) -> [Video] {
        if cachedRules == nil {
            let data = try! Data(contentsOf: rulesURL)
            cachedRules = try! JSONDecoder().decode(RecommendationRuleMap.self, from: data)
        }
        return filter(videos: allVideos, using: cachedRules!, near: location)
    }
}
```

## Takeaways

- On-device Power Profiler is the only tool for diagnosing battery issues that require real-world conditions (commuting, outdoor AR, hours-long background tasks) — use it by default for any field-reported battery complaint.
- The CPU power impact score is relative and dimensionless — use it to compare before/after states and to identify the highest-impact subsystem, not as an absolute energy measurement.
- Repeated heavy I/O or JSON parsing inside high-frequency callbacks (location, motion, audio) is a common source of surprising battery drain; cache aggressively.
- Build power regression testing into CI with `XCTest` performance baselines so battery regressions are caught before they reach users.

---
_Source: WWDC25 Session 226 page (abstract, chapter summaries, code samples, and resource links)._
