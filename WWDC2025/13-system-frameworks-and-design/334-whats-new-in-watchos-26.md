# What's new in watchOS 26
**WWDC25 · Session 334** · [Watch](https://developer.apple.com/videos/play/wwdc2025/334/)

_Platforms:_ watchOS 26

## Overview
watchOS 26 brings three broad categories of change: foundational platform updates (new design system and arm64 architecture on Series 9/Ultra 2), expanded app surface areas (Controls, MapKit APIs, workout-driven Smart Stack suggestions), and a brand-new mechanism for contextual content — the `RelevanceConfiguration` widget powered by a new **RelevanceKit** framework.

The session uses a beach-activity tracking app as a running example to illustrate configurable widgets, relevant widgets, and RelevanceKit's location-based point-of-interest contexts.

## Key Topics

### Platform updates: design system and arm64
Every app built for watchOS 10+ automatically adopts the new Liquid Glass design system (updated toolbar and control styles, new materials). App icons require updating via Icon Composer. Apple Watch Series 9 and later and Apple Watch Ultra 2 now run the **arm64 architecture** on watchOS 26. Use the "Standard Architectures" build setting on Watch targets. The Apple Watch Simulator on Apple Silicon has always been arm64 — if you test on the simulator, you're already arm64-validated.

### Controls on watchOS
Controls (WidgetKit-based) are now available on Apple Watch in three locations: Control Center (accessible from the side button), Smart Stack, and the Action button on Apple Watch Ultra. Controls can come from a watchOS app or from a paired iPhone app (action executed on iPhone). `AppIntentControlConfiguration` and `AppIntentControlValueProvider` are the same APIs used on iOS. To make a control configurable, add `.promptsForUserConfiguration()` to the configuration.

### Configurable widgets
On watchOS 26, widgets can be made configurable by the user (rather than offering pre-configured recommendations). Return an empty `recommendations()` array in `AppIntentTimelineProvider`. Use an `#available(watchOS 26, *)` guard to fall back to recommendations on older watchOS.

### MapKit on watchOS
MapKit APIs previously iOS-only are now available on watchOS 26: local search for nearby points of interest, routing/directions with transport type (walk, drive, cycle), route overlays on SwiftUI maps.

### Workout-driven Smart Stack suggestions
Apps that record workouts with HealthKit may be suggested in the Smart Stack based on the user's routine. To improve suggestion quality: use the correct `HKWorkoutActivityType`, record accurate start/end times, and use `HKWorkoutRouteBuilder` to add location data.

### RelevanceKit and Relevant Widgets
**RelevanceKit** is a new framework powering a new widget configuration type: **`RelevanceConfiguration`**. Relevant widgets appear exclusively in the Smart Stack and only when relevant; multiple cards can appear simultaneously for overlapping configurations (e.g., three concurrent beach events each get their own card).

Key types:
- `RelevanceConfiguration(kind:provider:content:)` — analogous to `StaticConfiguration` / `AppIntentConfiguration`
- `RelevanceEntriesProvider` — protocol; implement `relevance()` and `entry(configuration:context:)` and `placeholder(context:)`
- `WidgetRelevance<ConfigurationIntent>` — returned from `relevance()`
- `WidgetRelevanceAttribute(configuration:context:)` — pairs a config with a `RelevantContext`
- `RelevantContext` types: `.date(interval:kind:)`, `.date(_:kind:)`, `.location(category:)` (MapKit `MKPointOfInterestCategory`)
- `.associatedKind(_:)` modifier — links `RelevanceConfiguration` to a timeline widget so only one card shows per event
- Widget push updates (APNs) — now supported on watchOS via the same `WidgetPushHandler` mechanism available on other platforms

## APIs & Frameworks

### WidgetKit
- **`RelevanceConfiguration`** **[NEW]** — new widget configuration for Smart Stack relevant widgets (watchOS 26)
- **`RelevanceEntriesProvider`** protocol **[NEW]** — `relevance()`, `entry(configuration:context:)`, `placeholder(context:)`
- **`WidgetRelevance<Intent>`** **[NEW]** — `init(_ attributes: [WidgetRelevanceAttribute])`
- **`WidgetRelevanceAttribute`** **[NEW]** — `init(configuration:context:)` and `init(context:)` (void intent)
- **`RelevantContext`** **[NEW]** — `.date(interval:kind:)`, `.date(_:kind:)`, `.location(category:)` **[NEW for `.location`]**
- `RelevantContext.DateKind` — `.default`
- `.associatedKind(_:)` modifier on `RelevanceConfiguration` **[NEW]**
- `AppIntentTimelineProvider.recommendations()` — return `[]` to make widget configurable **[NEW behavior]**
- **`AppIntentControlConfiguration`** — `.promptsForUserConfiguration()` **[NEW]**
- **`AppIntentControlValueProvider`** protocol (existing, now available on watchOS)
- `WidgetPushHandler` protocol (existing) — now supported on watchOS 26

### RelevanceKit (new framework)
- `RelevantContext` (see above)
- `WidgetRelevanceAttribute` (see above)

### MapKit (new on watchOS)
- Local search / `MKLocalSearch` — **[NEW on watchOS 26]**
- Route directions with transport types — **[NEW on watchOS 26]**
- SwiftUI `Map` with route overlays — **[NEW on watchOS 26]**
- `MKPointOfInterestCategory` — `.beach` and others; used in `RelevantContext.location(category:)`

### HealthKit
- `HKWorkoutActivityType` — existing, correct value improves Smart Stack suggestions
- `HKWorkoutRouteBuilder` — existing, location data improves suggestions

### SwiftUI / watchOS
- Controls APIs (same as iOS): `ControlWidget`, `ControlWidgetConfiguration`

## Code Highlights

```swift
// Relevant widget configuration
struct BeachEventWidget: Widget {
    var body: some WidgetConfiguration {
        RelevanceConfiguration(kind: "BeachWidget", provider: BeachEventRelevanceProvider()) { entry in
            BeachWidgetView(entry: entry)
        }
        .associatedKind(WidgetKinds.beachEventsTimeline)
    }
}

// Relevance for beach point-of-interest category
func relevance() async -> WidgetRelevance<BeachEventConfigurationIntent> {
    let attributes = events.map { event in
        WidgetRelevanceAttribute(
            configuration: BeachEventConfigurationIntent(event: event),
            context: .date(interval: event.date, kind: .default)
        )
    }
    return WidgetRelevance(attributes)
}

// Location-based context
guard let context = RelevantContext.location(category: .beach) else { return .init([]) }
return WidgetRelevance([WidgetRelevanceAttribute(context: context)])

// Make a widget configurable
func recommendations() -> [AppIntentRecommendation<BeachConfigurationIntent>] {
    if #available(watchOS 26, *) { return [] }
    return recommendedBeaches
}
```

## Takeaways
- Build and test on watchOS 26 with a physical Series 9/Ultra 2 device to catch any arm64-specific Float/Int type-size issues.
- Bring iOS controls to Apple Watch with zero Watch-specific code if actions run on the paired iPhone; add a Watch-native `ControlWidget` if you have a Watch app.
- Implement `RelevanceConfiguration` for content where timing and location determine relevance (calendar events, workout suggestions, nearby POI info) — multiple cards can surface simultaneously, far more expressive than a timeline.
- Adopt widget push updates (`WidgetPushHandler`) to keep Smart Stack cards current without polling.

---
_Source: WWDC25 Session 334 page (abstract, chapter summaries, code samples, and resource links)._
