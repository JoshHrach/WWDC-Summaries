# Explore Enhancements to App Intents
**WWDC23 · Session 10103** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10103/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10

## Overview
This session covers the major expansions to App Intents in iOS 17, organized into three areas: widget integration, developer experience improvements, and Shortcuts app enhancements. The biggest additions are `WidgetConfigurationIntent` for defining configurable widget schemas directly in Swift code (replacing Intent Definition Files), interactive widget support via `Button` and `Toggle` with App Intents, and framework-level App Intents sharing via `AppIntentsPackage`.

Additional notable additions include `IntentParameterDependency` for dynamic cross-parameter option filtering, array parameter sizing per widget family, `ForegroundContinuableIntent` for intents that need to hand off to the foreground app, Apple Pay support inside `perform()`, `ProgressReportingIntent` for long-running intents, `EnumerableEntityQuery` for simple find actions, and `resultValueName` on `IntentDescription`.

## Key Topics

- **Widget configuration with App Intents** — Replace `IntentConfiguration` + Intent Definition File with `AppIntentConfiguration` + `WidgetConfigurationIntent` protocol; define queries/providers in widget extension without a separate Intents extension; one-click migration in Xcode via "Convert to App Intent" button.
- **Interactive widgets** — `Button` and `Toggle` in SwiftUI now accept App Intents; `perform()` executes on button/toggle interaction; same App Intent reusable in Shortcuts.
- **IntentParameterDependency** — New property wrapper allowing a `DynamicOptionsProvider` or `EntityQuery` to read values from other parameters of a parent intent for context-aware option filtering; supports multiple parameter dependencies.
- **Array parameter sizing** — Widget configuration array parameters can now declare `size` constraints per `WidgetFamily`.
- **ParameterSummary with widget family** — `When` statement on `widgetFamily` in `ParameterSummary` to show/hide parameters based on widget size.
- **widgetConfigurationIntent on user activity** — Retrieve configuration intent in app via `userActivity.widgetConfigurationIntent(of:)` for deep navigation on widget tap.
- **Framework support (AppIntentsPackage)** — Frameworks expose App Intents directly; app imports framework conforming to `AppIntentsPackage`; no more duplicate compilation; `AppShortcutsProvider` now definable in App Intents extensions.
- **ForegroundContinuableIntent** — `needsToContinueInForegroundError()` stops intent and requests foreground; `requestToContinueInForeground()` pauses and awaits foreground result.
- **Apple Pay in App Intents** — Create `PKPaymentRequest` and use `PKPaymentAuthorizationController` inside `perform()`.
- **ProgressReportingIntent** — `progress.totalUnitCount` / `completedUnitCount` for Shortcuts progress display.
- **EnumerableEntityQuery** — New protocol; implement `allEntities()` returning all possible entities; framework handles filtering; suitable for small entity sets only.
- **IntentDescription.resultValueName** — Names the output value for display in the Shortcuts editor; new `findIntentDescription` property on query types.
- **isDiscoverable** — Set to `false` to hide an App Intent from Shortcuts/Siri/Spotlight while still using it in widgets.
- **RelevantIntentManager / RelevantIntent** — New Swift-friendly API for Smart Stack widget suggestions (replaces `INRelevantShortcut`).

## APIs & Frameworks

**App Intents**
- `WidgetConfigurationIntent` protocol **[NEW]** — sub-protocol of `AppIntent`; defines widget configuration schema in Swift
- `AppIntentConfiguration` **[NEW]** — widget configuration type using `WidgetConfigurationIntent`; replaces `IntentConfiguration`
- `IntentParameterDependency` **[NEW]** — property wrapper in `DynamicOptionsProvider`/`EntityQuery` to read parent intent's parameter values
- `AppIntentsPackage` protocol **[NEW]** — lets frameworks and apps expose/re-export App Intents metadata
- `ForegroundContinuableIntent` protocol **[NEW]** — for intents that may need foreground continuation
- `ForegroundContinuableIntent.needsToContinueInForegroundError()` **[NEW]** — throws to stop intent and request foreground launch
- `ForegroundContinuableIntent.requestToContinueInForeground(_:)` **[NEW]** — async method; awaits foreground result from app
- `ProgressReportingIntent` protocol **[NEW]** — exposes `progress` object in `perform()`; displayed in Shortcuts app
- `EnumerableEntityQuery` protocol **[NEW]** — simple alternative to `EntityPropertyQuery`; implement `allEntities()`; framework handles filtering
- `IntentDescription.resultValueName` **[NEW]** — names the output value in Shortcuts editor
- `IntentDescription.findIntentDescription` **[NEW]** — provides description for auto-generated Find actions
- `AppIntent.isDiscoverable` **[NEW]** — set to `false` to hide from Shortcuts/Siri/Spotlight
- `AppShortcutsProvider` — now definable in App Intents extension (not only main app target) **[NEW]**
- `RelevantIntentManager` **[NEW]** — new API for Smart Stack widget relevance suggestions
- `RelevantIntent` **[NEW]** — wraps an App Intent with a `RelevantContext` (date range, etc.)
- `DynamicOptionsProvider` — existing; now supports `IntentParameterDependency`
- `EntityQuery` — existing; now supports `IntentParameterDependency` and `findIntentDescription`
- `EntityPropertyQuery` — existing; now supports `findIntentDescription`
- `ParameterSummary` — existing; new `When(_:in:_:otherwise:)` statement using `\.widgetFamily` **[NEW]**
- `@Parameter` property wrapper — existing; now accepts `size:` for array parameters with per-`WidgetFamily` mapping **[NEW]**

**SwiftUI**
- `Button(intent:)` / `Toggle(isOn:intent:)` — `Button` and `Toggle` now accept `AppIntent` directly **[NEW]**

**WidgetKit**
- `WidgetFamily` — used in `ParameterSummary.When` and array size declarations

**PassKit**
- `PKPaymentRequest` — use inside `AppIntent.perform()` for Apple Pay **[NEW support in intents]**
- `PKPaymentAuthorizationController` — present Apple Pay sheet from inside `perform()`

**UIKit / AppKit**
- `NSUserActivity.widgetConfigurationIntent(of:)` **[NEW]** — retrieve the widget's configuration intent on app launch from tap

## Code Highlights

Widget configuration intent (replaces Intent Definition File):
```swift
struct ShowNextBus: WidgetConfigurationIntent {
    static var title: LocalizedStringResource = "Show Next Bus"
    
    @Parameter(title: "Bus Stop") var busStop: BusStop
    @Parameter(title: "Route") var route: BusRoute?
    @Parameter(title: "Direction") var direction: Direction?
}
```

Interactive widget button with App Intent:
```swift
struct SetAlarm: AppIntent {
    @Parameter(title: "Bus Arrival Time") var arrivalTime: Date
    func perform() async throws -> some IntentResult { /* set alarm */ }
}

// In widget view:
Button(intent: SetAlarm(arrivalTime: nextBus.arrivalTime)) {
    Text("Set Alarm")
}
```

IntentParameterDependency for cross-parameter filtering:
```swift
struct BusRouteQuery: EntityQuery {
    @IntentParameterDependency<ShowNextBus>(\.$busStop) var showNextBus
    
    func suggestedEntities() async throws -> [BusRoute] {
        guard let stop = showNextBus?.busStop else { return allRoutes }
        return allRoutes.filter { $0.stops.contains(stop) }
    }
}
```

Foreground continuation:
```swift
func perform() async throws -> some IntentResult {
    guard hasConnectivity else {
        throw needsToContinueInForegroundError { /* navigate to error screen */ }
    }
    // ...
}
```

## Takeaways

- Use `WidgetConfigurationIntent` in your widget extension to define configurable widget schemas in Swift without separate Intent Definition Files or Intents extensions — migrate existing SiriKit widgets with one click in Xcode.
- `Button(intent:)` and `Toggle(isOn:intent:)` in SwiftUI enable interactive widgets without any additional boilerplate; the same App Intent is automatically surfaced in Shortcuts.
- Use `AppIntentsPackage` to share App Intents across frameworks and extensions without duplicating compiled code — critical when widgets and the main app share the same intents.
- Set `isDiscoverable = false` on App Intents that are designed only for internal widget/live-activity use and should not appear in Shortcuts or Spotlight.

---
_Source: WWDC23 Session 10103 page (abstract, chapter summaries, code samples, and resource links)._
