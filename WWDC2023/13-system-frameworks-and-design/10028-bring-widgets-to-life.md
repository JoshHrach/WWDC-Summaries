# Bring Widgets to Life
**WWDC23 · Session 10028** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10028/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10

## Overview
iOS 17 brings two major capabilities to WidgetKit: **animations** and **interactivity**. Animations allow smooth, customizable transitions between widget timeline entries, giving users a visual sense of how content changes. Interactivity lets users take actions directly from a widget — on the Home Screen, Lock Screen, or Standby — using `Button` and `Toggle` backed by App Intents, without opening the app.

The session explains how widget rendering architecture differs from a regular SwiftUI app: widget view code runs only during timeline archiving in the extension process, while display happens in the system process. This means interactivity must go through App Intents, and reloads triggered by interactions are always guaranteed to happen (unlike normal best-effort reloads). A new Xcode Preview API for widgets makes it easy to iterate on animations without waiting for actual timeline progression.

## Key Topics

### Animations
- Widgets automatically get an implicit spring animation and implicit content transitions when compiled with the latest SDK.
- Animations are driven by **differences between timeline entries** — SwiftUI detects what changed and animates those parts.
- Use `contentTransition(.numericText(value:))` for important numeric values to give them animated count-up/count-down prominence.
- Use the `.id(_:)` modifier to tell SwiftUI when a view should be treated as a new instance (triggering a transition) vs. an update to the same view.
- Apply `.transition(.push(from:))` or other standard SwiftUI transitions to animated removals/insertions.
- Use `.animation(_:value:)` to specify a custom spring or timing for a specific transition.
- The new **Xcode Preview API for widgets** (`#Preview(as:) { Widget } timeline: { entries }`) lets you click through timeline entries in the canvas and immediately see how animations play out.

### Widget Rendering Architecture
- Widget extension process runs at timeline archiving time; the system process renders the archived view representation at display time.
- Widget view code is **not running** when the widget is visible.
- `WidgetCenter.shared.reloadTimelines(ofKind:)` — call from the app when data changes; triggers re-archiving.
- Interactions (Button/Toggle taps) always trigger a guaranteed reload — unlike normal timeline reloads which are best-effort.

### Interactivity with App Intents
- Only `Button(intent:)` and `Toggle(isOn:intent:)` using `AppIntent` are supported in interactive widgets. Standard closures do not execute in the widget process.
- Define an `AppIntent` conforming type with `@Parameter` properties (only `@Parameter`-annotated properties are persisted and available when the intent is performed).
- `perform()` is `async` — use `await` for database writes, network calls, etc. Persist all necessary data before returning; the system triggers a timeline reload immediately when `perform()` returns.
- App Intents defined for widgets are automatically available in Shortcuts and Siri.

### Invalidatable Content
- `.invalidatableContent()` — modifier that shows a system-provided "loading" effect on a view while the widget is waiting for an updated timeline entry after an interaction. Use on meaningful values (e.g., a count that will change) but not on every view.
- `Toggle` optimistically updates its presentation immediately without waiting for a roundtrip; the system pre-renders both on/off states at archive time. Custom `ToggleStyle` implementations must check `configuration.isOn` to drive the appropriate appearance.

## APIs & Frameworks

### WidgetKit
- `WidgetConfiguration` — widget configuration protocol
- `TimelineProvider` — provides entries for the widget timeline
- `TimelineEntry` — entry in the widget timeline
- `WidgetCenter.shared.reloadTimelines(ofKind:)` — programmatic timeline reload
- `.containerBackground(for: .widget) { }` **[NEW]** — required modifier for widget backgrounds; enables support on Mac and iPad
- `#Preview(as: WidgetFamily) { Widget } timeline: { entries }` **[NEW]** — Xcode preview for widgets with timeline animation support

### SwiftUI (Widgets)
- `.contentTransition(.numericText(value:))` — animated numeric value transition **[NEW application in widgets]**
- `.id(_:)` — associates view identity with model data to trigger transitions on change
- `.transition(.push(from:))` — push transition from an edge
- `.animation(_:value:)` — binds animation to a value change
- `.invalidatableContent()` **[NEW]** — shows loading state while awaiting widget reload after interaction

### App Intents (Widgets)
- `AppIntent` protocol — defines an executable action
- `@Parameter` property wrapper — marks stored properties to be persisted and passed to `perform()`
- `IntentDescription(_:)` — human-readable description
- `perform() async throws -> some IntentResult` — async execution entry point
- `.result()` — empty intent result
- `Button(intent:label:)` — button initialized with an `AppIntent` **[NEW application in widgets]**
- `Toggle(isOn:intent:)` — toggle initialized with an `AppIntent` **[NEW application in widgets]**
- `OptionsProvider` — provides selectable options for `@Parameter` values (e.g., `DrinksOptionsProvider`)

## Code Highlights

Interactive button with App Intent:
```swift
struct LogDrinkIntent: AppIntent {
    static var title: LocalizedStringResource = "Log a drink"

    @Parameter(title: "Drink", optionsProvider: DrinksOptionsProvider())
    var drink: Drink

    func perform() async throws -> some IntentResult {
        await DrinksLogStore.shared.log(drink: drink)
        return .result()
    }
}

// In widget view:
Button(intent: LogDrinkIntent(drink: .espresso)) {
    Label("Espresso", systemImage: "plus")
}
```

Widget preview with animated timeline:
```swift
#Preview(as: WidgetFamily.systemSmall) {
    CaffeineTrackerWidget()
} timeline: {
    CaffeineLogEntry.log1
    CaffeineLogEntry.log2
    CaffeineLogEntry.log3
}
```

Invalidatable content for latency perception:
```swift
Text(totalCaffeine.formatted())
    .contentTransition(.numericText(value: totalCaffeine.value))
    .invalidatableContent()
```

## Takeaways
- Recompiling with iOS 17 SDK gives widgets automatic spring animations between entries for free; use `.contentTransition`, `.id`, `.transition`, and `.animation` to refine them.
- Interactive widgets use `Button(intent:)` and `Toggle(isOn:intent:)` backed by `AppIntent` — only `@Parameter`-annotated properties are available in `perform()`.
- Guaranteed reload after `perform()` means you must persist all data before returning from `perform()`.
- The new `#Preview` API for widgets lets you see timeline animations in the Xcode canvas without running on device.

---
_Source: WWDC23 Session 10028 page (abstract, chapter summaries, code samples, and resource links)._
