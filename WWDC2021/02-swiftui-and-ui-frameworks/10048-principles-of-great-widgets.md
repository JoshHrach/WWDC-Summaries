# Principles of Great Widgets
**WWDC21 · Session 10048** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10048/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8

## Overview
This session dives into the core principles that make widgets truly great: keeping them relevant, adapting to their presentation environment, and offering meaningful customization. The talk revisits WidgetKit concepts introduced in WWDC20 and extends them with new iOS 15 capabilities.

Relevance is broken down across three dimensions — time, presentation, and location — each with concrete guidelines and real app examples. Timelines are revisited with emphasis on forward-looking content, multiple entries, and budget-aware reloads, helping developers understand how the system manages update cadence.

Customization is explored through three axes: supporting multiple widget sizes (including the new Extra Large iPad size), publishing multiple widget kinds via `WidgetBundle`, and offering user-configurable experiences through `IntentConfiguration`.

## Key Topics

### Timeline Entries and Reload Policies
- Timelines with multiple entries allow widgets to surface fresh content without additional reloads (Photos, Weather examples).
- Three `TimelineReloadPolicy` options: `.atEnd` (for endless/future content), `.afterDate` (for periodically stale data), and `.never` (for app-driven updates).
- Update budget: heavily viewed widgets receive roughly 40–70 background updates per day (~every 15–30 min).
- `WidgetCenter` reload API triggers immediate, budget-free refreshes when the container app is in the foreground or in a user session (Navigation, Now Playing).

### Presentation Relevance
- WidgetKit auto-handles Light/Dark Mode via SwiftUI; use `BackgroundStyle` for standard system background colors.
- New in iOS 15: partial privacy redactions via `.privacySensitive()` view modifier — applies to any view or container.
- Full redaction: adopt the `default-data-protection` entitlement to withhold timeline updates while the device is passcode-locked.

### Location Relevance
- Declare `NSWidgetUsesLocation` in Info.plist; use `CLLocationManager` from within `TimelineProvider`.
- `CLLocationManager.isAuthorizedForWidgetUpdates` checks widget-specific location permission.
- Significant location changes trigger a budget-free reload (applied at next widget view).
- "While using app or widgets" permission grants location access up to 15 minutes after the widget was last viewed.

### Widget Customization
- **Sizes**: small, medium, large, and the new **Extra Large** (iOS 15, iPad-only); auto-supported if no families are specified when building with iOS 15 SDK.
- **Kinds**: publish multiple widget kinds (e.g., Stocks symbol vs. overview) using `WidgetBundle` with `@main`; bundle order defines gallery order.
- **Configuration**: `StaticConfiguration` for uniform content; `IntentConfiguration` + `IntentTimelineProvider` for user-personalized instances.

## APIs & Frameworks

- `WidgetKit` framework
- `TimelineProvider` protocol
- `IntentTimelineProvider` protocol **[NEW in context of iOS 15 updates]**
- `Timeline<Entry>` struct
- `TimelineReloadPolicy` — `.atEnd`, `.afterDate(_:)`, `.never`
- `WidgetCenter` — `reloadTimelines(ofKind:)`, `reloadAllTimelines()`
- `StaticConfiguration` struct
- `IntentConfiguration` struct
- `WidgetBundle` protocol **[NEW]**
- `WidgetPreviewContext` — used with `previewContext(_:)` in Xcode Previews
- `.previewContext(WidgetPreviewContext(family:))` modifier
- `.environment(\.colorScheme, .dark)` modifier
- `BackgroundStyle` — used with `Rectangle().fill(BackgroundStyle())`
- `.privacySensitive()` view modifier **[NEW]**
- `WidgetFamily.systemExtraLarge` **[NEW]** — new Extra Large iPad widget size
- `CLLocationManager` — `isAuthorizedForWidgetUpdates` property **[NEW]**
- `NSWidgetUsesLocation` Info.plist key
- `default-data-protection` entitlement (for full privacy redaction)

## Code Highlights

Previewing a widget in Dark Mode:
```swift
struct MyWidget_Previews: PreviewProvider {
    static var previews: some View {
        MyWidgetEntryView(date: Date())
            .previewContext(WidgetPreviewContext(family: .systemSmall))
            .environment(\.colorScheme, .dark)
    }
}
```

Applying partial privacy redaction to a balance field:
```swift
Text("$128.45")
    .privacySensitive()
```

Publishing multiple widget kinds in a `WidgetBundle`:
```swift
@main
struct MyWidgetBundle: WidgetBundle {
    var body: some Widget {
        IndividualSymbolWidget()
        StocksOverviewWidget()
    }
}
```

Intent-based widget configuration:
```swift
IntentConfiguration(kind: "com.sample.myIntentSampleWidgetKind",
                    intent: SampleConfigurationIntent.self,
                    provider: Provider()) { entry in
    SampleWidgetEntryView(entry: entry)
}
```

## Takeaways
- Use multiple timeline entries whenever content can be forecast; single-entry timelines should only be used when future data is genuinely unavailable.
- Choose `.atEnd` for endless content, `.afterDate` for periodically stale data, and `.never` for purely app-driven widgets.
- Adopt `.privacySensitive()` on any view that should be masked on the Lock Screen, and use `default-data-protection` to fully redact sensitive widgets.
- Support the new Extra Large family and publish multiple widget kinds via `WidgetBundle` to maximize discoverability and personalization.

---
_Source: WWDC21 Session 10048 page (abstract, chapter summaries, code samples, and resource links)._
