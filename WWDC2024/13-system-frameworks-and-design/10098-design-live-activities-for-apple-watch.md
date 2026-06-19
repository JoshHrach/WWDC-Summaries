# Design Live Activities for Apple Watch
**WWDC24 · Session 10098** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10098/)

_Platforms:_ watchOS 11, iOS 18

## Overview
watchOS 11 brings Live Activities to Apple Watch for the first time, displaying them on the Smart Stack and as the active complication while the associated iPhone activity is running. This session is a design guide covering how Live Activity content adapts from the iPhone Dynamic Island and Lock Screen to the Watch face and Smart Stack, the layout constraints unique to the round watch display, and best practices for update frequency and notification pairing.

The session uses a delivery tracking app, a sports scoreboard, and a workout timer as running examples, showing how the same `ActivityKit` data model drives distinct presentations on each surface with minimal extra code.

## Key Topics
- **Live Activity surfaces on watchOS** — the Smart Stack shows the Live Activity as a card; the watch face shows a compact form as the active complication; the full detail view appears on tap.
- **Layout adaptation** — watchOS uses the same `ActivityAttributes` and `ContentState` types as iOS; developers add a new `watchOS` view variant via `ActivityConfiguration.watchOSComplication` and `ActivityConfiguration.watchOSContent`.
- **Content hierarchy** — the watch face complication must convey the single most important piece of data at a glance; the Smart Stack card can show 2–3 values; the detail view mirrors the Lock Screen presentation.
- **Update cadence** — Live Activity updates on watchOS are batched and delivered from the paired iPhone; updates more frequent than once every 15 seconds are not guaranteed to reach the watch; design state to be legible even when stale.
- **Notification pairing** — a Live Activity update can include an `AlertConfiguration` to surface a transient notification on the watch wrist raise; use sparingly for truly time-critical events.

## APIs & Frameworks

**ActivityKit**
- `ActivityAttributes` — define the static and dynamic data model for a Live Activity; unchanged
- `Activity<Attributes>` — manage a Live Activity lifecycle: `request`, `update`, `end`; unchanged
- `ActivityContent` — wraps `ContentState` + staleness info; unchanged
- `ActivityConfiguration` — the widget configuration type for Live Activities
  - **[NEW]** `ActivityConfiguration.watchOSComplication` — provide a view for the watch face complication slot (minimal / circular form)
  - **[NEW]** `ActivityConfiguration.watchOSContent` — provide the Smart Stack card view (full card, similar to Lock Screen compact)
- `AlertConfiguration` — attach to an `ActivityContent` update to trigger a wrist notification; include `title`, `body`, `sound`

**WidgetKit (Live Activity views)**
- `ActivityViewContext<Attributes>` — context type passed into the Live Activity view builders; `.state` gives the current `ContentState`, `.isStale` indicates the data may be outdated
- `ContainerRelativeShape` — use for clipping Live Activity views to the surface's shape (round on watch, rounded rectangle on phone)
- `Text`, `ProgressView`, `Gauge` — all supported in watchOS Live Activity views; `Gauge` is especially well-suited for progress indication

**SwiftUI modifiers relevant to watchOS Live Activities**
- `.activityBackgroundTint(_:)` — tint the Live Activity background (iOS equivalent applied on watch too)
- `.activitySystemActionForegroundColor(_:)` — color for system action affordances
- `isLuminanceReduced` environment value — detect always-on display state; dim non-essential content

## Code Highlights
Add a watchOS complication to an existing Live Activity configuration:

```swift
struct DeliveryActivityConfiguration: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: DeliveryAttributes.self) { context in
            // iPhone Lock Screen view
            DeliveryLockScreenView(context: context)
        } dynamicIsland: { context in
            // Dynamic Island views
            DynamicIsland { … } compactLeading: { … } compactTrailing: { … } minimal: { … }
        }
        .watchOSComplication { context in
            // Watch face minimal complication
            Gauge(value: context.state.progress) {
                Image(systemName: "box.truck")
            }
        }
        .watchOSContent { context in
            // Smart Stack card
            DeliveryWatchCardView(context: context)
        }
    }
}
```

## Takeaways
- Add `.watchOSComplication` and `.watchOSContent` to every existing Live Activity widget to enable watchOS 11 support — the data model is already in place.
- Design the watch face complication to show a single value (estimated arrival time, score, elapsed time); the Smart Stack card can hold 2–3 values; the detail view can be as rich as the Lock Screen.
- Respect the 15-second update batching on watchOS — do not rely on sub-15-second update intervals for time-critical UI; use `AlertConfiguration` for truly urgent events instead.
- Use `isLuminanceReduced` to dim non-essential elements during always-on display mode; keep the primary value legible at low brightness.

---
_Source: WWDC24 Session 10098 page (abstract, chapter summaries, code samples, and resource links)._
