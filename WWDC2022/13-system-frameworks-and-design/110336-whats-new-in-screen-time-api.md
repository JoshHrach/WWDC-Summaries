# What's New in Screen Time API
**WWDC22 · Session 110336** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110336/)

_Platforms:_ iOS 16, iPadOS 16

## Overview
The Screen Time API — introduced in iOS 15 with three frameworks (FamilyControls, ManagedSettings, DeviceActivity) — receives three significant enhancements in iOS 16. First, FamilyControls gains an individual-device authorization mode so apps can build digital wellness tools for any user, not just parental controls for children. Second, ManagedSettings now supports up to 50 named stores per process, automatically shared between the app and all its extensions. Third, DeviceActivity adds a new `DeviceActivityReport` extension point that lets apps render completely custom SwiftUI usage-report views while preserving user privacy — the raw activity data never leaves the sandboxed report extension.

## Key Topics

### FamilyControls — Individual Authorization
iOS 15 supported only parental-controls authorization via iCloud (linking a parent device to a child device). iOS 16 adds `.individual` authorization: any user can authorize the Screen Time API for themselves, directly on their device. Multiple third-party apps can each hold individual authorization simultaneously. Individual authorization does not implicitly prevent iCloud sign-out or app deletion (those restrictions remain a parental-controls-only behavior). Authorization shows a system alert, then requires Face ID / Touch ID / passcode. Once granted, re-calling `requestAuthorization` silently succeeds. A new entry appears in Settings → Screen Time → Apps with Screen Time Access.

### ManagedSettings — Named Stores
iOS 15 limited each process to a single `ManagedSettingsStore` and required different stores for the app and its extensions. iOS 16 allows:
- Up to **50 named stores** per process, each identified by a `ManagedSettings.StoreName`
- Named stores are **automatically shared** between the app and all app extensions (e.g., DeviceActivityMonitor)
- `ManagedSettingsStore.clearAllSettings()` removes all restrictions from a given named store instantly
- The most restrictive setting always wins when multiple stores apply conflicting rules — clearing one store does not lift restrictions set by another

### DeviceActivity — Custom Usage Reports
A new `DeviceActivityReport` view + `DeviceActivityReportScene` extension point enables fully custom SwiftUI usage-report UIs:
- Define a `DeviceActivityReport.Context` enum value to identify the report type
- Place the `DeviceActivityReport(context:filter:)` SwiftUI view in your app's UI
- Implement `DeviceActivityReportScene` in a new Report Extension target; conform `makeConfiguration(representing:)` to map `[DeviceActivityData]` to a view-model
- The framework invokes `makeConfiguration` automatically when fresh usage data arrives
- Usage data never leaves the extension sandbox — the app only receives a rendered SwiftUI view

## APIs & Frameworks

**FamilyControls**
- `AuthorizationCenter.shared.requestAuthorization(for:)` — async/throws method; `.individual` authorization type **[NEW]**
- `FamilyControlsMember.individual` **[NEW]** — authorize current device user (non-parental-control use)
- `FamilyControlsMember.child` — existing parental-controls mode (unchanged)

**ManagedSettings**
- `ManagedSettingsStore(named:)` **[NEW]** — create a named store shared across app and extensions
- `ManagedSettingsStore.StoreName` **[NEW]** — string-typed name for a settings store
- `ManagedSettingsStore.clearAllSettings()` **[NEW]** — remove all settings in one call
- Up to 50 stores per process **[NEW limit]**
- `ManagedSettingsStore.shield.applicationCategories` — shield apps by `ActivityCategory` token
- `ManagedSettingsStore.shield.webDomainCategories` — shield web domains by category token

**DeviceActivity**
- `DeviceActivityReport` SwiftUI view **[NEW]** — embed in your app to display custom usage UI
- `DeviceActivityReport.Context` **[NEW]** — typed context identifier for a report
- `DeviceActivityFilter` **[NEW]** — specify timing window and segment for the report
- `DeviceActivityReportScene` protocol **[NEW]** — implement in a Report Extension; `makeConfiguration(representing:)` maps raw data to a view configuration
- `DeviceActivityReportExtension` protocol **[NEW]** — entry point for the Report Extension target
- `DeviceActivityData` — data type passed to `makeConfiguration`; provides per-app/category usage

## Code Highlights

```swift
// Request individual authorization (iOS 16)
import FamilyControls
let center = AuthorizationCenter.shared
try await center.requestAuthorization(for: .individual)
```

```swift
// Named ManagedSettingsStore — shared between app and extensions
import ManagedSettings
let socialStore = ManagedSettingsStore(named: .social)

// Restrict social media apps/sites
socialStore.shield.applicationCategories = .specific([socialCategoryToken])
socialStore.shield.webDomainCategories   = .specific([socialCategoryToken])

// Remove all restrictions in this store (e.g., at start of allowed window)
socialStore.clearAllSettings()
```

```swift
// Custom DeviceActivityReport — app side
import DeviceActivity
extension DeviceActivityReport.Context {
    static let pieChart = Self("Pie Chart")
}
// In SwiftUI body:
DeviceActivityReport(context: .pieChart,
                     filter: DeviceActivityFilter(segment: .daily(during: thisWeek)))

// Report Extension implementation
struct PieChartReport: DeviceActivityReportScene {
    let context: DeviceActivityReport.Context = .pieChart
    let content: (PieChartView.Configuration) -> PieChartView

    func makeConfiguration(representing data: [DeviceActivityData]) -> PieChartView.Configuration {
        let usageByCategory = data.map { /* map to category totals */ }
        return PieChartView.Configuration(totalUsageByCategory: usageByCategory)
    }
}
```

## Takeaways

- Use `.individual` authorization to build digital wellness and productivity apps for adult users — not just parental controls; multiple apps per device are now supported.
- Replace any single-store `ManagedSettingsStore` usage with named stores so your app and all DeviceActivityMonitor extensions share restrictions atomically; `clearAllSettings()` makes time-window toggling straightforward.
- Implement `DeviceActivityReportScene` in a dedicated Report Extension to render custom SwiftUI usage charts; the framework calls `makeConfiguration` automatically and the raw data never leaves the extension sandbox.
- The "most restrictive wins" rule across named stores means you can layer independent restriction categories (e.g., gaming + social) without risk of one store accidentally lifting another's rules.

---
_Source: WWDC22 Session 110336 page (abstract, chapter summaries, code samples, and resource links)._
