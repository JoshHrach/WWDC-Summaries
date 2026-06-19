# Meet ActivityKit
**WWDC23 · Session 10184** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10184/)

_Platforms:_ iOS 16.1+, iPadOS 17 (Live Activities on iPad new in iOS 17)

## Overview
ActivityKit is the framework for creating and managing Live Activities — glanceable, real-time displays of ongoing tasks shown on the Lock Screen, the Dynamic Island (iPhone 14 Pro and later), and StandBy. This session introduces the full lifecycle of a Live Activity: from requesting one, updating it with new data, observing state changes, through to ending and dismissing it.

Live Activities are declared with SwiftUI and WidgetKit — the same infrastructure as Home Screen widgets. They require all Dynamic Island presentations (compact, minimal, expanded) in addition to the Lock Screen UI. iOS 17 adds iPad support and interactive widgets (buttons/toggles) via WidgetKit.

## Key Topics

### Live Activity Presentations
- **Lock Screen / StandBy**: primary rectangular presentation; StandBy scales the Lock Screen view to fill the screen.
- **Dynamic Island — Compact**: single app's activity fills the Dynamic Island with a leading and trailing view joined into one cohesive pill.
- **Dynamic Island — Minimal**: two activities share the Dynamic Island; each gets a tiny attached or detached view showing only the most critical data.
- **Dynamic Island — Expanded**: long-press expands the island into a taller view divided into `.leading`, `.trailing`, and `.bottom` regions; each region can deep-link into different parts of the app.

### New in iOS 17
- iPadOS support for Live Activities.
- StandBy mode scales the Lock Screen view to fill the display.
- Interactive Live Activities (buttons, toggles) via WidgetKit — see "Bring Widgets to Life."
- ActivityKit push notifications for remote updates (see "Update Live Activities with Push Notifications").

### Defining ActivityAttributes
- Conform a struct to `ActivityAttributes` to describe a Live Activity.
- Static data: properties on `ActivityAttributes` (do not change after start).
- Dynamic data: define a nested `ContentState: Codable & Hashable` struct; updated throughout the activity's lifetime.

### Lifecycle

**Requesting:**
- App must be in the foreground.
- Request only after a discrete user action (e.g., starting a task, opting to follow an event).
- Create an `ActivityContent` with the initial `ContentState`, an optional `staleDate`, and a `relevanceScore`.
- Call `Activity.request(attributes:content:pushType:)`.
- `pushType: .token` enables remote push updates; `nil` allows only local updates.
- Live Activity availability is user-moderated (users can turn off per-app).

**Updating:**
- Create a new `ContentState` and an `ActivityContent`.
- Call `activity.update(_:alertConfiguration:)`.
- `AlertConfiguration` triggers a sound/notification alert on iPhone, iPad, and a synced Apple Watch.
- On Apple Watch, `AlertConfiguration.title` and `body` are shown as a notification; on iPhone/iPad, the Live Activity UI updates with the specified sound.

**Observing state:**
- Async stream: `for await activityState in activity.activityStateUpdates { ... }`
- Synchronous: `activity.activityState`
- States: `.started`, `.finished`, `.dismissed`, `.stale`

**Ending:**
- Create a final `ContentState` representing the concluded state.
- Choose a `ActivityUIDismissalPolicy`: `.default` (remains briefly on Lock Screen), `.immediate` (dismiss at once), or `.after(Date)` (dismiss at a specific time).
- Call `activity.end(ActivityContent(...), dismissalPolicy: policy)`.

### Building the UI
- Add `ActivityConfiguration` to the app's `WidgetBundle` alongside other widgets.
- `ActivityConfiguration(for: MyAttributes.self)` takes two closures: Lock Screen view and `dynamicIsland` builder.
- `ActivityViewContext` provides `context.attributes` (static), `context.state` (current ContentState), `context.isStale`, and `context.activityID`.
- `.activityBackgroundTint(Color)` — sets the Lock Screen tinted background.
- `DynamicIsland` view builder takes four trailing closures: expanded (`DynamicIslandExpandedRegion`), `compactLeading`, `compactTrailing`, `minimal`.
- `DynamicIslandExpandedRegion(.leading/.trailing/.bottom/.center)` — positions content within the expanded island.

## APIs & Frameworks
- `ActivityKit` framework — request, update, observe, and end Live Activities
- `ActivityAttributes` protocol — defines static + dynamic data shape for a Live Activity
- `ActivityAttributes.ContentState` — nested type; `Codable & Hashable`; dynamic data
- `Activity<Attributes>` — the running Live Activity handle
- `Activity.request(attributes:content:pushType:)` — starts a Live Activity; must be in foreground
- `ActivityContent` **[NEW context iOS 16.2]** — wraps `ContentState` with `staleDate` and `relevanceScore`
- `ActivityContent.staleDate` — when the content is considered out-of-date
- `ActivityContent.relevanceScore` — ordering when multiple activities from same app
- `Activity.update(_:alertConfiguration:)` — pushes new `ActivityContent` and optionally alerts
- `AlertConfiguration` — alert presented when a significant update occurs
- `AlertConfiguration.title`, `.body`, `.sound` — title/body for Watch; sound for iPhone/iPad
- `Activity.activityStateUpdates` — async stream of `ActivityState` changes
- `Activity.activityState` — synchronous current state
- `ActivityState` enum — `.started`, `.finished`, `.dismissed`, `.stale`
- `Activity.end(_:dismissalPolicy:)` — concludes the Live Activity
- `ActivityUIDismissalPolicy` — `.default`, `.immediate`, `.after(Date)`
- `ActivityConfiguration` (WidgetKit) — Live Activity WidgetConfiguration
- `ActivityViewContext` (WidgetKit) — passed to Lock Screen and Dynamic Island view builders; provides `attributes`, `state`, `isStale`, `activityID`
- `.activityBackgroundTint(_:)` (SwiftUI modifier) — tints the Lock Screen background
- `DynamicIsland` (WidgetKit) — view builder for Dynamic Island presentations
- `DynamicIslandExpandedRegion` (WidgetKit) — positions content in expanded Dynamic Island
- `DynamicIslandExpandedRegionPosition` — `.leading`, `.trailing`, `.bottom`, `.center`
- `WidgetKit` framework — layout infrastructure for Live Activities and widgets
- `SwiftUI` — declarative UI for all Live Activity presentations

## Code Highlights

Define attributes and content state:
```swift
struct AdventureAttributes: ActivityAttributes {
    let hero: EmojiRanger          // static

    struct ContentState: Codable & Hashable {
        let currentHealthLevel: Double
        let eventDescription: String
    }
}
```

Request a Live Activity:
```swift
let adventure = AdventureAttributes(hero: hero)
let initialState = AdventureAttributes.ContentState(
    currentHealthLevel: hero.healthLevel,
    eventDescription: "Adventure has begun!"
)
let content = ActivityContent(state: initialState, staleDate: nil, relevanceScore: 0.0)
let activity = try Activity.request(attributes: adventure, content: content, pushType: nil)
```

Update with an alert:
```swift
let contentState = AdventureAttributes.ContentState(
    currentHealthLevel: hero.healthLevel,
    eventDescription: "\(heroName) has taken a critical hit!"
)
let alertConfig = AlertConfiguration(
    title: "\(heroName) has taken a critical hit!",
    body: "Open the app and use a potion to heal \(heroName)",
    sound: .default
)
activity.update(ActivityContent(state: contentState, staleDate: nil),
                alertConfiguration: alertConfig)
```

Observe state changes asynchronously:
```swift
Task {
    for await activityState in activity.activityStateUpdates {
        if activityState == .dismissed { cleanUpDismissedActivity() }
    }
}
```

End a Live Activity:
```swift
let finalContent = AdventureAttributes.ContentState(
    currentHealthLevel: hero.healthLevel,
    eventDescription: "Adventure over! \(hero.name) has defeated the boss!"
)
activity.end(ActivityContent(state: finalContent, staleDate: nil), dismissalPolicy: .default)
```

Define the Lock Screen and Dynamic Island UI:
```swift
struct AdventureActivityConfiguration: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: AdventureAttributes.self) { context in
            AdventureLiveActivityView(hero: context.attributes.hero,
                                     isStale: context.isStale,
                                     contentState: context.state)
                .activityBackgroundTint(Color.navyBlue)
        } dynamicIsland: { context in
            DynamicIsland {
                DynamicIslandExpandedRegion(.leading) { LiveActivityAvatarView(hero: context.attributes.hero) }
                DynamicIslandExpandedRegion(.trailing) { StatsView(hero: context.attributes.hero, isStale: context.isStale) }
                DynamicIslandExpandedRegion(.bottom) { HealthBar(currentHealthLevel: context.state.currentHealthLevel) }
            } compactLeading: {
                Avatar(hero: context.attributes.hero)
            } compactTrailing: {
                ProgressView(value: context.state.currentHealthLevel)
                    .progressViewStyle(.circular)
                    .tint(context.state.currentHealthLevel <= 0.2 ? .red : .green)
            } minimal: {
                ProgressView(value: context.state.currentHealthLevel) { Avatar(hero: context.attributes.hero) }
                    .progressViewStyle(.circular)
                    .tint(context.state.currentHealthLevel <= 0.2 ? .red : .green)
            }
        }
    }
}
```

## Takeaways
- Every Live Activity must implement all four presentations (Lock Screen, compact, minimal, expanded) — `ActivityConfiguration` requires them.
- Request Live Activities only in the foreground and only after a discrete user action; users can turn off Live Activities per app just like notifications.
- Use `AlertConfiguration` (not push notifications) for urgent same-device updates; the title and body surface as a notification on Apple Watch.
- `ActivityUIDismissalPolicy.default` keeps the final state on the Lock Screen briefly — important so users can glance the outcome after the activity ends.

---
_Source: WWDC23 Session 10184 page (abstract, transcript, chapter summaries, and code samples)._
