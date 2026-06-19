# Get Started with HealthKit in visionOS
**WWDC24 · Session 10083** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10083/)

_Platforms:_ visionOS 2, iOS 18, iPadOS 18

## Overview
HealthKit comes to visionOS 2, enabling apps to read, write, and observe health data on Apple Vision Pro. The session covers the capabilities of HealthKit on the new platform, how to bring an existing iPadOS app to visionOS (often requiring zero code changes), and how to craft experiences that take full advantage of the spatial canvas using SwiftUI, Swift Charts, and Swift concurrency.

A key focus is the Guest User feature unique to visionOS: when someone else uses your device in Guest mode, apps inherit the owner's HealthKit authorizations but cannot request new ones or write data. The session explains the new error types and best practices for gracefully handling these restrictions.

## Key Topics

**HealthKit in visionOS**
- HealthKit on visionOS behaves similarly to iPadOS: read/write/observe data, compute statistics, sync via iCloud
- Apps compiled against iOS 17 or later using the iOS SDK already offer a "Designed for iPad" experience on visionOS 2.0 with no changes needed
- Use `HKHealthStore.isHealthDataAvailable()` to guard HealthKit features — handles device/OS variation automatically
- Request authorization with `healthDataAccessRequest(store:shareTypes:readTypes:trigger:completion:)` (HealthKitUI SwiftUI modifier)

**Spatial Health Experiences**
- Standard SwiftUI components (TabView, Sheets) translate directly to visionOS without modification
- Leverage `onGeometryChange(for:_:action:)` (new SwiftUI API) to dynamically adjust chart density as the window resizes
- Open additional windows using the `openWindow` environment action with a window identifier — easy if the iPad app already supports split view
- Build fully immersive experiences using `ImmersiveSpace` to present emotional logging or meditation flows with spatial audio and passthrough dimming

**Guest User Support**
- Apps running for a Guest User inherit the owner's HealthKit authorizations — no authorization sheet is presented
- Calling `healthDataAccessRequest` during a Guest session fails with `HKError.errorNotPermissibleForGuestUserMode` — defer the request
- Writing health data during a Guest session fails with the same error — discard the data to prevent it from being saved to the owner's health store later
- Present an alert notifying the Guest that their data was not saved

## APIs & Frameworks

**HealthKit**
- `HKHealthStore.isHealthDataAvailable()` — check platform support before using HealthKit
- `HKHealthStore.save(_:)` — save a health sample (async)
- `HKStateOfMind` **[NEW]** — sample type for logging momentary emotions; properties: `date`, `kind`, `valence`, `labels`, `associations`
- `HKStateOfMind.Association` — contextual association (e.g., calendar event)
- `.stateOfMindType()` — `HKSampleType`/`HKObjectType` for state of mind data
- `HKError.errorNotPermissibleForGuestUserMode` **[NEW]** — error thrown when HealthKit operations are blocked during Guest User sessions

**HealthKitUI (SwiftUI)**
- `healthDataAccessRequest(store:shareTypes:readTypes:trigger:completion:)` — SwiftUI view modifier for requesting authorization; completion returns `Result<Bool, Error>`

**SwiftUI**
- `onGeometryChange(for:_:action:)` **[NEW]** — observe view size changes and respond; used to scale chart point count dynamically
- `@Environment(\.openWindow)` — open a new named window
- `openWindow(id:)` — present a secondary window (e.g., a chart viewer)
- `ImmersiveSpace` — full immersive environment for visionOS experiences

**Swift Charts**
- `Chart { ... }` — used with `onGeometryChange` to adapt displayed data to window size

## Code Highlights

Observe window size changes to scale chart density:
```swift
Chart { ... }
    .onGeometryChange(for: Int.self) { proxy in
        Int(proxy.size.width / 80)
    } action: { newValue in
        chartBinCount = newValue
    }
```

Handle Guest User write error and alert the user:
```swift
do {
    try await healthStore.save(sample)
} catch HKError.errorNotPermissibleForGuestUserMode {
    // Drop data generated in a Guest User session
    didError.wrappedValue = true
}
```

## Takeaways
- Apps compiled against iOS 17+ automatically work on visionOS 2 — start there and add visionOS-specific enhancements incrementally.
- Use `onGeometryChange` and multi-window support to create richer spatial data visualizations that scale with the infinite canvas.
- Always handle `HKError.errorNotPermissibleForGuestUserMode` for both authorization requests and data saves — discard Guest data and present a clear alert.
- Consider building an `ImmersiveSpace` experience for high-value health interactions like emotion logging or mindfulness.

---
_Source: WWDC24 Session 10083 page (abstract, chapter summaries, code samples, and resource links)._
