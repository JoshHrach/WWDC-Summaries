# Track Down Hangs with Xcode and On-Device Detection
**WWDC22 · Session 10082** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10082/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
A "hang" occurs when an app's main thread is blocked for 250 ms or more, rendering the UI unresponsive. This session introduces a suite of new tools in Xcode 14 and iOS 16 that help catch hangs at every phase of the development lifecycle: at the desk in Xcode, in TestFlight/beta distribution, and after App Store release.

Four new capabilities are covered: the Thread Performance Checker (proactive at-desk alerting), hang detection and labeling in Instruments' Time Profiler, on-device hang detection in iOS 16 Developer Settings, and Hang Reports in the Xcode 14 Organizer. The session also recommends submitting apps to the App Store with symbol information to enable one-click navigation from Organizer stack traces to source code.

## Key Topics

### What Is a Hang?
A hang is any period in which the main thread is busy doing work (CPU-bound) or waiting on another thread or system resource, causing a UI update delay of at least 250 ms. Common causes: priority inversions, synchronous network or disk I/O on the main thread, and excessive work dispatched to the main queue.

### Thread Performance Checker (New in Xcode 14)
Enabled in the scheme's Diagnostics section, the Thread Performance Checker detects priority inversions and non-UI work performed on the main thread while debugging (no profiling required). Issues appear in the Xcode Issue Navigator as runtime warnings, pinpointing the call stack at the moment the violation was detected.

### Hang Detection in Instruments (New in Xcode 14)
The Time Profiler and CPU Profiler instruments now automatically detect and label hangs in the process timeline with their duration. A new standalone Hang Tracing instrument can be added to any trace document and supports configuring a minimum hang duration threshold. Hangs are highlighted as intervals, making it easy to triple-click to create a time filter and inspect the call stacks of all threads during the hang.

### On-Device Hang Detection (New in iOS 16)
Available in Settings → Developer → Hang Detection for development-signed and TestFlight apps. Features:
- Configurable hang threshold (250 ms, 500 ms, or higher)
- Real-time push notification when a hang is detected
- Logs available for each detected hang: a human-readable text log and a tailspin (`.ips`) file
- Diagnostics are processed asynchronously in the background to minimize overhead
- Tailspins can be opened in Instruments for deep thread-interaction analysis

### Hang Reports in Xcode Organizer (New in Xcode 14)
The Xcode Organizer gains a Hang Reports section alongside existing Crashes and Energy reports. Reports are aggregated from customers who have opted in to share analytics. Similar stack traces are grouped into signatures, sorted by user impact. Each signature shows sample hang logs with main-thread call stacks, hang durations, and device/OS breakdown. One-click navigation from a symbolicated frame to source code is available when the app was submitted with symbol information.

### App Store Connect REST API
Hang report data is also accessible via the App Store Connect Power and Performance APIs, enabling integration with custom dashboards or CI/CD systems.

## APIs & Frameworks

**Xcode 14 (tools, not runtime APIs)**
- Thread Performance Checker — scheme Diagnostics setting **[NEW]**; detects priority inversions and non-UI main-thread work
- Hang Tracing instrument **[NEW]** — standalone instrument; configurable minimum hang duration
- Time Profiler instrument — existing; hang detection and labeling added **[NEW]**
- CPU Profiler instrument — existing; hang detection and labeling added **[NEW]**
- Xcode Organizer Hang Reports **[NEW]** — aggregated hang signatures from App Store customers

**iOS 16 System**
- On-device hang detection — Developer Settings → Hang Detection **[NEW]**
  - Hang threshold configuration: 250 ms, 500 ms, 1 s, etc.
  - Real-time hang notification
  - Text hang log (`.txt`) **[NEW]**
  - Tailspin (`.ips`) file **[NEW]** — openable in Instruments for thread analysis

**MetricKit** (existing, referenced)
- `MXHangDiagnostic` — existing; per-user hang diagnostics from beta/release apps
- `MXAppResponsivenessMetric` — existing; hang rate metrics

**App Store Connect REST API** (existing)
- Power and Performance APIs — existing; now includes hang report data

## Code Highlights

No new Swift/Objective-C runtime APIs are introduced in this session. All new features are tooling-level changes. Key setup steps:

1. Enable Thread Performance Checker: Product → Scheme → Edit → Diagnostics → "Thread Performance Checker"
2. Profile with Time Profiler: Product → Profile → Time Profiler template → record → triple-click hang interval
3. Enable on-device hang detection: Settings → Developer → Hang Detection → toggle on, set threshold
4. View Hang Reports: Xcode → Window → Organizer → Hang Reports tab

## Takeaways
- Address hangs at the earliest possible phase: Thread Performance Checker catches priority inversions at desk without profiling; Instruments labels hangs during traces; on-device detection finds real-world hangs in TestFlight without Xcode attached.
- On-device hang detection in iOS 16 is indispensable for catching hangs that only manifest under real-world conditions (poor network, slow disk, etc.) that are hard to reproduce at a desk.
- The Xcode 14 Organizer Hang Reports, sorted by user impact, make it practical to prioritize which hangs to fix after release — especially the top signature responsible for the most hang time.
- Submit apps to the App Store with symbol information to unlock symbolicated Organizer stack traces and one-click source navigation.

---
_Source: WWDC22 Session 10082 page (abstract, chapter summaries, and resource links)._
