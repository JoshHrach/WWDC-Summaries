# What's New in watchOS 11
**WWDC24 · Session 10205** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10205/)

_Platforms:_ watchOS 11, iOS 18

## Overview
watchOS 11 developer story covers four areas: Live Activities from iOS apps automatically appearing in the Smart Stack on Apple Watch (with custom small-family views), a powerful new relevance API so the system can suggest widgets at the right time, interactive widgets on watchOS (buttons and toggles via App Intents), a new `AccessoryWidgetGroup` layout template, Double Tap expansion to work inside apps (automatic scrolling + `.handGestureShortcut` for primary actions), and health/fitness additions in WorkoutKit (pool swimming custom workouts, `distanceWithTime` goal, `displayName`) and HealthKit (State of Mind / mood API).

## Key Topics

### Live Activities on Apple Watch
iOS Live Activities appear automatically in the Smart Stack on watchOS 11 with no code changes required. The Dynamic Island's leading and trailing views are used by default. To provide a customized watchOS layout, add `.supplementalActivityFamilies([.small])` to the `ActivityConfiguration`. Inside that view, use `@Environment(\.activityFamily)` to distinguish `.small` (Smart Stack on Watch) from `.medium` (Lock Screen on iPhone/iPad) and tailor the layout accordingly.

### Relevant Widgets
The new `relevances()` method on `TimelineProvider` returns `WidgetRelevances<Intent?>` — an array of `WidgetRelevanceEntry` objects, each wrapping a `RelevantContext`. Available contexts include:
- `.date(DateInterval)` — relevant during a specific time window
- `.location(CLLocation, radius:)` — relevant when near a location
- `.sleep` — relevant at bedtime or wake-up time
- `.fitness` — relevant based on workout state (e.g., `.activityRingsIncomplete`)

For `AppIntentConfiguration` widgets, `WidgetRelevanceEntry` accepts a `configuration` parameter tied to a specific intent instance, enabling per-configuration relevance (e.g., each saved coffee shop location gets its own proximity relevance). Call `invalidateRelevances()` when data changes.

### Interactive Widgets on watchOS
`Button` and `Toggle` backed by App Intents now work inside watchOS widgets across all widget families. For actions needing confirmation (e.g., unlocking a door remotely), use `requestConfirmation(with:conditions:)` with `.lowConfidenceSource` to let the system decide when accidental taps are possible and prompt accordingly.

### AccessoryWidgetGroup
A new SwiftUI view template for `accessoryRectangular` widgets. Displays up to three content views (more are silently dropped) with a label. Key properties:
- `Label` — defaults to the widget extension bundle name; provide a custom `Text` or `Label` view
- `Content` — up to 3 views, each individually interactive or linkable via `Link`
- `.accessoryWidgetGroupStyle(.circular)` or `.roundedSquare` — content view masking shape (default `.circular`)
- Empty views auto-inserted if fewer than 3 provided; empty view color not configurable
- `.containerBackground()` — tints group background; `.foregroundStyle()` — tints label

### Double Tap Expansion
Double Tap (Apple Watch Series 9 and Ultra 2) now automatically scrolls through `List`, `ScrollView`, and vertical `TabView` — no code needed. For primary actions, apply `.handGestureShortcut(.primaryAction)` to any `Button` or `Toggle`. The system outlines the control with a highlight when Double Tap would invoke it. Only one element can be the primary action at a time. If the control is off screen, Double Tap scrolls toward it rather than activating it. Do not combine list/scroll auto-scrolling with an additional `.primaryAction` in the same view.

### WorkoutKit Updates
- Custom pool swimming workouts (joins cycling and running in Custom Workouts API)
- New goal type `distanceWithTime` — a single step can have both a distance and a time goal
- `displayName` property for Work, Recovery, Warmup, and Cooldown steps in all custom workout types

### HealthKit Updates
- New State of Mind API for reading and writing mood/emotion data (`HKStateOfMind`) — see "Explore wellbeing APIs in HealthKit"

## APIs & Frameworks

**ActivityKit / Live Activities**
- iOS Live Activities auto-appear in watchOS 11 Smart Stack (no code required) **[NEW behavior]**
- `.supplementalActivityFamilies([.small])` on `ActivityConfiguration` **[NEW]** — watchOS custom layout
- `@Environment(\.activityFamily)` — `.small` (Watch) vs `.medium` (iPhone Lock Screen)

**WidgetKit**
- `TimelineProvider.relevances()` **[NEW]** — returns `WidgetRelevances<Intent?>`
- `WidgetRelevanceEntry` **[NEW]** — wraps a `RelevantContext` (and optionally an intent configuration)
- `RelevantContext.date(_:)` **[NEW]** — date interval relevance
- `RelevantContext.location(_:radius:)` **[NEW]** — proximity relevance
- `RelevantContext.sleep` **[NEW]** — bedtime/wake-up relevance
- `RelevantContext.fitness(conditions:)` **[NEW]** — fitness state relevance
- `invalidateRelevances()` **[NEW]** — invalidate cached relevance entries
- Interactive widgets: `Button` / `Toggle` with App Intents in watchOS widget families **[NEW]**
- `requestConfirmation(with:conditions:)` **[NEW]** — confirmation prompt before intent execution
- `AccessoryWidgetGroup` **[NEW]** — 3-item rectangular widget template
  - `.accessoryWidgetGroupStyle(.circular)` / `.roundedSquare` **[NEW]**

**SwiftUI / WatchKit**
- `.handGestureShortcut(.primaryAction)` **[NEW]** — bind Double Tap to a `Button` or `Toggle`
- Automatic Double Tap scrolling for `List`, `ScrollView`, vertical `TabView` **[NEW behavior]**

**WorkoutKit**
- Pool swimming custom workout type **[NEW]**
- `distanceWithTime` goal type **[NEW]**
- `displayName` on workout steps **[NEW]**

**HealthKit**
- `HKStateOfMind` — mood/emotion data read/write **[NEW]**

## Code Highlights

Custom Live Activity view for Apple Watch:
```swift
ActivityConfiguration(for: MyAttributes.self) { context in
    // iPhone Lock Screen view
} dynamicIsland: { context in
    // Dynamic Island
}
.supplementalActivityFamilies([.small])

// In supplemental activity view:
struct WatchLiveActivityView: View {
    @Environment(\.activityFamily) var activityFamily
    var body: some View {
        if activityFamily == .small {
            // Compact watchOS layout
        } else {
            // iPhone medium layout
        }
    }
}
```

Relevant widget using date context (Reminders):
```swift
func relevances() async -> WidgetRelevances<Void> {
    let entries = reminders.map { reminder in
        WidgetRelevanceEntry(context: .date(reminder.dueDateInterval))
    }
    return WidgetRelevances(entries)
}
```

Relevant widget per App Intent configuration (Coffee Shop):
```swift
func relevances() async -> WidgetRelevances<CoffeeShopIntent> {
    let entries = favoriteShops.map { shop in
        WidgetRelevanceEntry(
            configuration: CoffeeShopIntent(shop: shop),
            context: .location(shop.location, radius: 500)
        )
    }
    return WidgetRelevances(entries)
}
```

AccessoryWidgetGroup:
```swift
AccessoryWidgetGroup(label: Text("Contacts")) {
    ForEach(pinnedContacts.prefix(3)) { contact in
        Link(destination: contact.deepLinkURL) {
            ContactAvatarView(contact: contact)
        }
    }
}
.accessoryWidgetGroupStyle(.circular)
```

Double Tap primary action:
```swift
Button("Lock Door") { lockDoor() }
    .handGestureShortcut(.primaryAction)
```

Interactive widget with confirmation:
```swift
Button(intent: UnlockDoorIntent()) {
    Image(systemName: "lock.open")
}
// In AppIntent:
func perform() async throws -> some IntentResult {
    try await requestConfirmation(
        with: UnlockConfirmationView(),
        conditions: .lowConfidenceSource
    )
    return .result()
}
```

## Takeaways
- iOS Live Activities appear on watchOS 11 automatically; add `.supplementalActivityFamilies([.small])` to provide a layout optimized for the wrist — especially for dense data like lyrics, scores, or timers.
- Use `RelevantContext` in `relevances()` to let the system surface your widget at the exact moment users need it — `.date`, `.location`, `.sleep`, and `.fitness` contexts cover the most common Smart Stack scenarios.
- `AccessoryWidgetGroup` provides a clean, system-consistent layout for showing three items (contacts, scenes, locations) in a rectangular widget with built-in interactivity support.
- Apply `.handGestureShortcut(.primaryAction)` to your app's single most important contextual button — Double Tap is best used sparingly and predictably.

---
_Source: WWDC24 Session 10205 page (abstract, chapter summaries, and resource links)._
