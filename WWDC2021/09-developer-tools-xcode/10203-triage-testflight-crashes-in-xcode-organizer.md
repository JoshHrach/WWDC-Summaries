# Triage TestFlight Crashes in Xcode Organizer
**WWDC21 · Session 10203** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10203/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
Xcode 13 introduces significant improvements to the Crashes Organizer and adds a new Terminations Organizer. The biggest changes focus on speed and context: TestFlight crash logs now arrive in Xcode Organizer within moments of occurring (instead of being batched), and tester feedback submitted after crashes is now surfaced directly alongside crash reports in the Organizer's new Feedback Inspector.

The session also covers new filtering capabilities (by time period up to one year, by product — App Clip, Watch app, extensions, iOS on Apple silicon), distribution graphs showing crash prevalence across versions/builds and TestFlight vs. App Store, and a share feature for sending crash deep-links to team members. A new Terminations Organizer tracks app process terminations (memory limits, launch timeouts, etc.) separately from crashes. MetricKit gains faster crash diagnostic delivery (next-launch instead of daily) and new macOS support.

## Key Topics

### Crashes Organizer — New in Xcode 13

**Speedy TestFlight Crash Delivery**
- Crash logs from TestFlight builds now delivered to Xcode Organizer within moments of occurring **[NEW]**
- Hourly graph showing when a selected crash occurred **[NEW]**
- One year of crash history now available (up from ~2 weeks) **[NEW]**

**New Filters**
- Filter by time period (last day, week, two weeks, months, up to one year) **[NEW]**
- Filter by version and build **[NEW]**
- Filter by product: main app, App Clip, Watch app, app extensions, iOS on Apple silicon **[NEW]**

**Distribution Graphs**
- Version distribution graph: see crash prevalence across app versions and builds **[NEW]**
- TestFlight vs. App Store crash source breakdown **[NEW]**
- Month-to-month time distribution graph in inspector **[NEW]**

**Crash Sharing**
- New Share button in Organizer toolbar generates a deep-link to a specific crash **[NEW]**
- Clicking shared link opens Organizer and focuses on that crash
- Crash report can be renamed for easier recognition

**TestFlight Feedback Integration**
- New Feedback Inspector in Crashes Organizer **[NEW]**: shows tester feedback submissions for the selected crash
- Feedback includes: tester comment, app version/build, device model, battery level, available disk space, network connection type
- App Store Connect "Open in Xcode" button opens the associated crash directly in Organizer **[NEW]**

### Terminations Organizer (New)
- Separate from Crashes Organizer; tracks process terminations not caused by programming failures **[NEW]**
- Categories: timeout on launch, system memory limit exceeded, background terminations, foreground terminations
- Compare termination counts across app versions to identify regressions
- Helps distinguish background vs. foreground termination impact

### MetricKit Improvements
- Crash diagnostics now delivered on the next app launch, not aggregated daily **[NEW]**
- `MXDiagnosticPayload.crashDiagnostics`: array of `MXCrashDiagnostic` received in `didReceive(_:)`
- MetricKit now supports macOS **[NEW]**

### Other Crash Viewing Tools
- Devices window: crashes for connected devices
- XCTest: collects crashes from test runs
- Console app: Mac and Simulator crashes
- Direct device sharing of crash logs

## APIs & Frameworks

- `MetricKit` framework
- `MXMetricManager` — shared singleton
- `MXMetricManager.add(_:)` — register subscriber
- `MXMetricManager.remove(_:)` — unregister subscriber
- `MXMetricManagerSubscriber` protocol
- `MXMetricManagerSubscriber.didReceive(_:)` — receives `[MXDiagnosticPayload]`
- `MXDiagnosticPayload`
- `MXDiagnosticPayload.crashDiagnostics` — `[MXCrashDiagnostic]?`
- `MXCrashDiagnostic`
- Xcode Organizer (no public API — IDE tool)
- TestFlight (no public API — distribution service)

## Code Highlights

Collecting crash diagnostics with MetricKit:
```swift
import MetricKit

class Subscriber: NSObject {
    override init() {
        super.init()
        MXMetricManager.shared.add(self)
    }
    deinit {
        MXMetricManager.shared.remove(self)
    }
}

extension Subscriber: MXMetricManagerSubscriber {
    func didReceive(_ payloads: [MXDiagnosticPayload]) {
        payloads.forEach {
            if let crashDiagnostics = $0.crashDiagnostics {
                // Analyze crash diagnostic payload
            }
        }
    }
}
```

## Takeaways

- Xcode 13's near-instant TestFlight crash delivery and integrated Feedback Inspector dramatically shorten the feedback loop for beta testing — crashes and tester context are now available in one place.
- Use the new product filters (App Clip, Watch app, extensions) and year-long crash history to track down regressions that span multiple releases.
- The new Terminations Organizer surfaces non-crash process terminations (memory limits, launch timeouts) that were previously harder to discover; compare across versions to find regressions.
- MetricKit crash diagnostics now deliver on next launch rather than daily, making programmatic crash monitoring more timely; MetricKit is now also available on macOS.

---
_Source: WWDC21 Session 10203 page (abstract, chapter summaries, code samples, and resource links)._
