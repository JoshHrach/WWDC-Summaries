# Bring Your Live Activity to Apple Watch
**WWDC24 · Session 10068** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10068/)

_Platforms:_ watchOS 11, iOS 18 (Live Activities originate on iPhone; watchOS 11 displays them on Apple Watch)

## Overview
watchOS 11 introduces the ability to display iPhone Live Activities on Apple Watch—specifically in the Smart Stack and as a watch face complication. This session explains the new `supplementalActivityFamilies` API that lets developers opt their existing Live Activities into watchOS display, and the `activityFamily` SwiftUI environment value that allows Live Activity views to adapt their layout for the watch's smaller display.

The session covers two levels of adoption: a simple opt-in that reuses existing compact Lock Screen views scaled for Apple Watch, and a more tailored approach using the `activityFamily` environment to provide Apple Watch-specific layouts. It also covers the lifecycle implications—Live Activities on Apple Watch receive updates via the same mechanisms as iPhone (push notifications and ActivityKit), with no additional API changes needed for update delivery.

## Key Topics
- **Smart Stack** — Live Activities appear in the Apple Watch Smart Stack when the activity is active on the paired iPhone; shown as a widget-style card
- **`supplementalActivityFamilies`** — new static property on `ActivityAttributes`; opt-in your Live Activity for watchOS display
- **`.activityFamily` environment value** — new SwiftUI environment key; value is `.small` on Apple Watch Smart Stack, `.medium` on Lock Screen compact views
- **View adaptation** — use `@Environment(\.activityFamily)` to provide Apple Watch-optimized layout branches
- **No separate push infrastructure** — Live Activity updates via ActivityKit or broadcast push channels reach the watch automatically through the paired iPhone
- **Always-On Display (AOD)** — Live Activities shown in Smart Stack respect AOD; keep contrast and legibility high for reduced-luminance rendering
- **Complication slot** — a Live Activity can also occupy a watch face complication slot while active

## APIs & Frameworks
### ActivityKit / Live Activities
- `ActivityAttributes` — unchanged; add `supplementalActivityFamilies` static property
- **[NEW] `supplementalActivityFamilies: [ActivityFamily]`** — static property on `ActivityAttributes`; `.small` enables Smart Stack display on watchOS 11
  ```swift
  static var supplementalActivityFamilies: [ActivityFamily] { [.small] }
  ```
- `Activity<Attributes>` — unchanged; start/update/end APIs remain the same
- `ActivityUIDynamicIslandConfiguration` — Dynamic Island config; unrelated to watchOS display
- `ActivityConfiguration` — existing configuration; watchOS uses `compactLeading`/`compactTrailing` or the new `supplemental` view builder

### SwiftUI (WidgetKit / Live Activity views)
- **[NEW] `activityFamily` environment value** — `@Environment(\.activityFamily) var activityFamily`; `.small` = Apple Watch Smart Stack, `.medium` = Lock Screen compact
- `WidgetFamily.accessoryRectangular` — used for Apple Watch complication slot display of Live Activities
- `ContainerRelativeFrame` — use for sizing that adapts to the watch display area
- `TimelineView` — can be used within Live Activity views for animated content (respect performance budget)

### WidgetKit
- `WidgetConfiguration` — Live Activities use `ActivityConfiguration`; no separate watch widget target needed
- Watch display is driven from the same `ActivityConfiguration` as iPhone; the `activityFamily` environment value distinguishes rendering context

## Code Highlights
```swift
// 1. Opt in to watchOS Smart Stack display
struct DeliveryAttributes: ActivityAttributes {
    // New: opt in to watchOS display
    static var supplementalActivityFamilies: [ActivityFamily] { [.small] }

    struct ContentState: Codable, Hashable {
        var status: String
        var estimatedMinutes: Int
    }
}

// 2. Adapt Live Activity view for Apple Watch
struct DeliveryLiveActivityView: View {
    @Environment(\.activityFamily) var activityFamily
    let context: ActivityViewContext<DeliveryAttributes>

    var body: some View {
        if activityFamily == .small {
            // Compact Apple Watch layout
            VStack(alignment: .leading) {
                Label(context.state.status, systemImage: "shippingbox")
                    .font(.caption2)
                Text("\(context.state.estimatedMinutes) min")
                    .font(.title3).bold()
            }
            .padding(8)
        } else {
            // Full iPhone Lock Screen layout
            DeliveryFullLockScreenView(context: context)
        }
    }
}

// 3. ActivityConfiguration includes watch display automatically
let config = ActivityConfiguration(for: DeliveryAttributes.self) { context in
    DeliveryLiveActivityView(context: context)
} dynamicIsland: { context in
    // Dynamic Island config unchanged
    DynamicIsland { ... } compactLeading: { ... } compactTrailing: { ... } minimal: { ... }
}
```

## Takeaways
- Adopting watchOS Live Activity display requires only two additions: the `supplementalActivityFamilies` static property and an `activityFamily`-aware view branch—no separate watch target or update infrastructure needed
- The Smart Stack is automatically prioritized by watchOS 11 when a Live Activity is active; prioritize legibility at small sizes and for Always-On Display's reduced luminance mode
- Design the `.small` family layout for glanceability: one number or status string should dominate, with minimal supporting text
- Updates sent via ActivityKit or broadcast APNs channels reach the watch automatically through the paired iPhone; no changes to your update delivery code are required

---
_Source: WWDC24 Session 10068 page (abstract, chapter summaries, code samples, and resource links)._
