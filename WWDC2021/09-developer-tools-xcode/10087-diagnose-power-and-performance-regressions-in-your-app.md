# Diagnose Power and Performance regressions in your app
**WWDC21 · Session 10087** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10087/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session introduces two major Xcode 13 Organizer enhancements: a new **Regressions** pane that surfaces your app's most important performance degradations across releases, and a new **Insights** field in disk write diagnostics that identifies known anti-patterns and provides actionable optimization suggestions.

The session covers the seven metric categories in Xcode Organizer, how regressions are algorithmically detected, three concrete regression examples (onscreen terminations, background terminations, disk writes), the Instruments File Activities tool for validating disk-write fixes, and the App Store Connect API endpoints for programmatic access to the same data.

## Key Topics

### Seven Metric Categories in Xcode Organizer
- Battery, Launches, Hangs, Memory, Disk Writes, Scroll Hitches, Terminations
- Each category supports per-device-type filtering and per-percentile (typical/top user) breakdowns
- Data spans the last 16 releases for trend analysis

### Regression Detection (Algorithm)
- A regression is flagged when a metric trends upward over recent releases AND the latest value exceeds the average of the last few releases.
- Displayed in the new **Regressions** (Insights) pane: metric name, category, percent change, affected percentiles, affected devices, and a chart of recent history.

### Regressions Pane (NEW in Xcode 13)
- One-stop view of all flagged regressions across all metric categories, grouped by category and subcategory.
- Left column: which metric regressed, magnitude, percentiles affected.
- Right column: chart of last 4 releases, latest value vs. average, device breakdown.
- Replaces manual cross-category scanning.

### Termination Regressions — Two Types
- **Onscreen terminations** (illegal instruction exits, memory pressure): disruptive—users see the app quit.
  - Diagnose via crash diagnostics to identify invalid function pointer accesses.
- **Background terminations** (task timeouts): apps have 30 seconds to complete background tasks; exceeding this causes silent background kills.
  - Fix: call `endBackgroundTask(_:)` properly; integrate UIKit state restoration for smooth re-launch.

### Disk Write Regressions and Insights (NEW in Xcode 13)
- Disk writes reports: aggregated from consented devices; show stack-trace signatures sorted by total bytes written.
- Top signature: identifies the call path responsible for the majority of writes.
- **Insights field**: Xcode 13 scans incoming reports against a repository of known anti-patterns and surfaces an optimization suggestion inline with each signature.
  - Example: "Add an SQLite index" — identifies a missing index causing full-table scans and excess SQLite write amplification.
  - Each insight links to documentation with steps to reproduce and fix the issue.
- The anti-pattern library is continuously expanded by Apple.

### Validating Disk Write Fixes with File Activities Instruments
- Instruments → File Activities track: shows every read/write with byte counts and latency.
- Used to confirm before/after behavior after applying Organizer insights.
- Example result: adding an SQLite index reduced temporary-file writes from 180 MB to 0 MB and eliminated 780 ms of latency.

### App Store Connect API (Programmatic Access)
- `GET /v1/apps/{application-id}/perfPowerMetrics` — metrics + insights for recent versions.
- `GET /v1/builds/{id}/perfPowerMetrics` — metrics for a specific build.
- `GET /v1/builds/{id}/diagnosticSignatures` — list of top disk write / crash / hang signatures.
- `GET /v1/diagnosticSignatures/{id}/logs` — detailed log entries for a signature.
- JSON response includes: `regressions` array (metric category, summary, populations of impacted percentiles/devices), `diagnosticSignatures` list, and per-signature `insights`.
- Integrate into existing analytics pipelines to automate regression detection across releases.

## APIs & Frameworks

**Xcode Organizer (Xcode 13)**
- Seven metric categories: Battery, Launches, Hangs, Memory, Disk Writes, Scroll Hitches, Terminations **[existing, expanded]**
- Per-device and per-percentile filtering **[existing]**
- Regressions / Insights pane — algorithmic detection of performance regressions across releases **[NEW in Xcode 13]**
- Disk write insights with anti-pattern matching and optimization suggestions **[NEW in Xcode 13]**
- Disk write diagnostic reports with stack-trace signatures sorted by byte volume **[existing]**

**Instruments**
- File Activities instrument — per-read/write breakdown with byte counts and latency **[existing]**

**UIKit**
- `UIApplication.beginBackgroundTask(expirationHandler:)` / `UIApplication.endBackgroundTask(_:)` — background task lifecycle management **[existing]**
- State restoration APIs (`UISceneDelegate.stateRestorationActivity(for:)`) — smooth re-launch after background termination **[existing]**

**MetricKit**
- `MXMetricPayload` / `MXDiagnosticPayload` — on-device metric and diagnostic delivery **[existing]**

**App Store Connect API**
- `GET /v1/apps/{id}/perfPowerMetrics` **[existing, updated response schema]**
- `GET /v1/builds/{id}/perfPowerMetrics` **[existing]**
- `GET /v1/builds/{id}/diagnosticSignatures` **[existing]**
- `GET /v1/diagnosticSignatures/{id}/logs` **[existing]**
- JSON `insights` field in regressions response **[NEW]**
- JSON `populations` field with impacted percentile/device breakdown **[NEW/expanded]**

## Code Highlights

App Store Connect API endpoints:
```
GET /v1/apps/{application-id}/perfPowerMetrics
GET /v1/builds/{id}/perfPowerMetrics

GET /v1/builds/{id}/diagnosticSignatures
GET /v1/diagnosticSignatures/{id}/logs
```

Ending a background task to avoid background termination:
```swift
var backgroundTask: UIBackgroundTaskIdentifier = .invalid

func startWork() {
    backgroundTask = UIApplication.shared.beginBackgroundTask {
        // Expiration handler — clean up and end the task
        UIApplication.shared.endBackgroundTask(self.backgroundTask)
        self.backgroundTask = .invalid
    }
    // ... perform work ...
    UIApplication.shared.endBackgroundTask(backgroundTask)
    backgroundTask = .invalid
}
```

## Takeaways
- The new Regressions pane in Xcode Organizer is the first place to check after every release; it summarizes your top performance priorities across all metric categories and devices.
- Disk write insights in Xcode 13 automatically match your top signatures against known anti-patterns (e.g., missing SQLite index) and provide fix suggestions and documentation links.
- Background task timeouts are a silent but frequent source of background terminations; always call `endBackgroundTask(_:)` and implement state restoration for a seamless re-launch.
- Use the App Store Connect API to integrate regression detection and diagnostic signature data into your own CI/analytics pipeline.

---
_Source: WWDC21 Session 10087 page (abstract, full transcript, code samples, and resource links)._
