# Live Activities Essentials
**WWDC26 · Session 223** · [Watch](https://developer.apple.com/videos/play/wwdc2026/223/)

_Platforms:_ iOS, iPadOS, watchOS, CarPlay

## Overview
This essentials session covers the complete lifecycle of a Live Activity from data model definition through real-time updates to optimization for each presentation context. Using a coffee-shop order-tracking app as the running example, it demonstrates how to structure `ActivityAttributes`, build tailored SwiftUI views for the Lock Screen, Dynamic Island (compact, expanded, and minimal), Apple Watch, and StandBy, and update the activity locally via `ActivityKit` or remotely via push notifications.

New in iOS 27: a landscape Dynamic Island style that provides more horizontal space when iPhone is rotated, a small `activityFamily` for Apple Watch, and an `isDynamicIslandLimitedInWidth` environment value for adapting compact views to the landscape constraint. The session pairs well with "Bring your Live Activity to Apple Watch" (WWDC24) for deeper watchOS coverage.

## Key Topics

### Create and Update (1:53)
Live Activities are defined by two types:
- **`ActivityAttributes`** — static, set at creation time (shop name, drink type, order ID)
- **`ContentState`** — dynamic, updated throughout the activity's life (order phase, estimated ready date, rating)

The widget extension declares `ActivityConfiguration(for: DrinkOrderAttributes.self)` with two closures: a lock-screen `ActivityView` and a `DynamicIsland` builder with regions (leading, center, trailing, bottom), compact leading/trailing, and minimal. Start with `Activity.request(attributes:content:)`, update with `activity.update(ActivityContent(state:staleDate:))`, and end with `activity.end(...)`.

Remote updates via ActivityKit push notifications allow server-driven updates; see "Starting and updating Live Activities with ActivityKit push notifications" in Resources.

### Optimize (9:51)
Four optimization areas:

1. **Landscape Dynamic Island** — **[NEW]** `isDynamicIslandLimitedInWidth` environment value is `true` in the new landscape style. Use it to show a compact icon instead of a text-heavy view when width is constrained.

2. **StandBy** — Use `showsWidgetContainerBackground` to detect the StandBy context and extend a gradient background; use `activityBackgroundTint` to tint the background color for the Lock Screen.

3. **Small activity family (Apple Watch)** — **[NEW]** Opt in with `.supplementalActivityFamilies([.small])` on `ActivityConfiguration`. Read `@Environment(\.activityFamily)` to show a compact `SmallView` for watch vs. the full `DetailView` on Lock Screen.

4. **Interactive elements** — Implement `LiveActivityIntent` (a specialized `AppIntent`) with `@Parameter` values, then wire `Button(intent:)` inside the Live Activity view. The intent's `perform()` runs in the app extension and can update the activity state (e.g., log a drink rating).

## APIs & Frameworks

**ActivityKit** — `import ActivityKit`
- `ActivityAttributes` protocol — static activity data
- `ActivityAttributes.ContentState` — dynamic state (Codable, Hashable)
- `Activity<Attributes>` — the running activity
- `Activity.request(attributes:content:)` — start a Live Activity
- `Activity.update(_ content: ActivityContent<ContentState>)` — update state
- `Activity.end(_ content:dismissalPolicy:)` — end the activity
- `ActivityContent<ContentState>` — wraps state with optional `staleDate`
- `ActivityAuthorizationInfo().areActivitiesEnabled` — check system permission
- Push notification support — APNS with `activity-identifier` and `content-state`

**WidgetKit** — `import WidgetKit`
- `ActivityConfiguration(for:content:dynamicIsland:)` — declare the Live Activity widget
- `DynamicIsland { } compactLeading: compactTrailing: minimal:` — Dynamic Island builder
- `DynamicIslandExpandedRegion` — `.leading`, `.center`, `.trailing`, `.bottom`
- `ActivityViewContext<Attributes>` — provides `.attributes` and `.state` in views
- `.supplementalActivityFamilies([.small])` — **[NEW]** opt in to small (watch) family
- `WidgetConfiguration` modifier

**SwiftUI environment values (Live Activity context)**
- **[NEW]** `\.isDynamicIslandLimitedInWidth: Bool` — landscape Dynamic Island
- `\.showsWidgetContainerBackground: Bool` — detect StandBy
- **[NEW]** `\.activityFamily: ActivityFamily` — `.medium` (Lock Screen) or `.small` (watch)
- `.activityBackgroundTint(_:)` modifier — Lock Screen tint color

**App Intents** — `import AppIntents`
- `LiveActivityIntent` protocol — intent type for Live Activity interactivity
- `@Parameter(title:)` — declare intent parameters
- `Button(intent:)` — interactive button inside the activity view
- `IntentResult` — returned from `perform()`

## Code Highlights

Define the Live Activity model:
```swift
public struct DrinkOrderAttributes: ActivityAttributes {
    let shopName: String
    let drink: Drink
    let orderID: UUID
    public struct ContentState: Codable, Hashable {
        var phase: DrinkOrder.Phase = .waiting
        var estimatedReadyDate: Date
        var rating: DrinkOrder.Rating?
    }
}
```

Start a Live Activity:
```swift
let activity = try Activity.request(
    attributes: attributes,
    content: ActivityContent(state: contentState, staleDate: nil)
)
```

Adapt compact trailing view for landscape Dynamic Island:
```swift
@Environment(\.isDynamicIslandLimitedInWidth) var isDynamicIslandLimitedInWidth
// ...
if isDynamicIslandLimitedInWidth {
    StepProgressIconView(context: context)
} else {
    EstimatedReadyView(context: context, ...)
}
```

Opt into small activity family for Apple Watch:
```swift
ActivityConfiguration(for: DrinkOrderAttributes.self) { ... } dynamicIsland: { ... }
    .supplementalActivityFamilies([.small])
```

Interactive rating button:
```swift
Button(intent: RateDrinkIntent(orderID: context.attributes.orderID.uuidString, isPositive: true)) {
    Label("Great", systemImage: "hand.thumbsup.fill")
}
```

## Takeaways
- Separate data strictly into `ActivityAttributes` (static, set once) and `ContentState` (dynamic, updated frequently) — this keeps push payloads small and updates efficient.
- Always provide a `.supplementalActivityFamilies([.small])` opt-in and a dedicated `SmallView` for Apple Watch; the medium family Lock Screen view rarely fits the watch display well.
- Handle `isDynamicIslandLimitedInWidth` to gracefully compact text-heavy compact trailing views in the new landscape Dynamic Island style.
- Use `LiveActivityIntent` + `Button(intent:)` for the single most time-critical action (confirm, rate, cancel) — users act on Live Activities without switching apps.

---
_Source: WWDC26 Session 223 page (abstract, chapter summaries, code samples, and resource links)._
