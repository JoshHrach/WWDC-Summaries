# Diagnose Performance Issues with the Xcode Organizer
**WWDC20 · Session 10076** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10076/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, tvOS (Xcode 12 tool; all app platforms)

## Overview
The Xcode Organizer in Xcode 12 receives a major update that makes it far more actionable for diagnosing performance and battery issues in shipped apps. Data is collected from consented devices, aggregated on Apple's servers, and delivered to the developer without any code changes required. Two major new data types are introduced: scroll hitch metrics and disk write diagnostic reports. The interactive UI allows side-by-side comparison of any two app versions with a single click.

The session walks through two concrete workflows: identifying a scroll performance regression introduced in a specific release, and triaging a disk write spike using the new diagnostic reports. The usage threshold required before data appears in the Organizer has been lowered by a factor of five, making the tool available to a much wider audience of developers.

## Key Topics

### New Interactive UI (Xcode 12)
- Side-by-side comparison of any two app versions in a single click — select an older version and the right panel updates to show diffs.
- Detailed subcategory breakdown for battery usage (Camera, Bluetooth, CPU, etc.) per version.
- Versions with limited usage are marked with an icon; associated margin-of-error information is displayed.
- Data appears sooner: usage threshold lowered 5x compared to Xcode 11.

### Scroll Hitch Metrics **[NEW]**
- A **scroll hitch** occurs when a rendered frame does not appear on screen at its expected time during a user scroll — perceived as jitter or skipping.
- **Hitch time**: total extra time frames take to appear on screen.
- **Scroll duration**: total time a user spends scrolling.
- **Hitch rate** = hitch time / scroll duration (measured in ms/s):
  - < 5 ms/s: good — users perceive smooth scroll.
  - 5–10 ms/s: warning — frames dropped every couple of seconds.
  - > 10 ms/s: critical — frequent frame drops, poor user experience.

### Disk Write Diagnostics Reports **[NEW]**
- Disk write metrics were available in Xcode 11; Xcode 12 adds detailed diagnostic reports.
- Reports are aggregated when an app exceeds **1 GB of disk writes in a 24-hour period**.
- Each report shows a signature (stack trace), percentage of total disk writes for that signature, breakdown by device type and OS, and a 14-day trend chart.
- Developers can mark signatures as resolved to track progress.
- Reducing writes improves performance, battery life, and overall device health.

### Existing Metrics (Carried Forward)
- Battery usage (with subcategory breakdown)
- Hang rate
- Launch time
- Memory usage
- Disk writes (now augmented with diagnostic reports)
- Terminations

## APIs & Frameworks

### Xcode Organizer (Xcode 12)
- Accessed via Xcode menu bar: Window > Organizer
- No SDK changes required — data collected automatically from consented devices
- **Metrics pane**: Battery, Hang Rate, Launch Time, Memory, Disk Writes, Scrolling (Hitch Rate) **[NEW]**
- **Reports pane**: Crashes, Energy, Disk Writes **[NEW]**
- Version comparison: select any two versions for side-by-side view **[NEW]**
- Limited-usage indicator with margin-of-error display **[NEW]**
- Lowered usage threshold (5x reduction) **[NEW]**

### MetricKit (companion framework)
- `MXMetricManager` — receive on-device metric reports
- `MXMetricPayload` — aggregated metric data (see companion session "What's New in MetricKit")
- Power and Performance API (see companion session "Identify Trends with the Power and Performance API")

## Code Highlights
No code samples in this session — the Xcode Organizer requires no code changes. See related sessions for MetricKit API details and XCTest hitch testing.

## Takeaways
- Upgrade to Xcode 12 and use the new interactive Organizer to compare battery/performance metrics across any two app versions without any code changes.
- Monitor the new **scroll hitch rate** metric; target < 5 ms/s; rates above 10 ms/s indicate a serious user-visible problem.
- Use **disk write diagnostic reports** to identify which code paths are responsible when writes exceed 1 GB/day, then mark signatures resolved as fixes ship.
- Lowered usage thresholds mean more developers can now see Organizer data for their apps, including early glimpses of brand-new releases.

---
_Source: WWDC20 Session 10076 page (abstract, transcript, and resource links)._
