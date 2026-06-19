# Build a Productivity App for Apple Watch
**WWDC22 · Session 10133** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10133/)

_Platforms:_ watchOS 9

## Overview
This session walks through building a full-featured productivity app for Apple Watch using watchOS 9 APIs and SwiftUI. It covers the key system features that make watchOS apps useful in practice: background tasks for keeping data fresh, smart stack widgets for glanceable information, push notifications with actions, and Workout sessions for fitness-related workflows. The session demonstrates how each of these capabilities applies in a task management app context and explains how to combine them for a coherent, high-value wrist experience.

watchOS 9 brings significant new capabilities including NavigationStack and NavigationSplitView for richer app structure, native SwiftUI charts, and the smart stack — a redesigned complication experience that surfaces relevant widgets automatically based on context such as time of day, location, and workout state. Apps that embrace these building blocks can deliver genuinely desktop-class productivity on the wrist.

## Key Topics

### Navigation & App Structure
- `NavigationStack` on watchOS 9 — push/pop navigation for multi-level app flows **[NEW on watchOS]**
- `NavigationSplitView` — two-column navigation for watchOS **[NEW on watchOS]**
- Toolbar items placed with `.toolbar` modifier; `ToolbarItem(placement: .topBarTrailing)` for watch crown area buttons

### Background App Refresh
- `WKApplicationRefreshBackgroundTask` — system-scheduled background task for refreshing app state
- `WKURLSessionRefreshBackgroundTask` — background `URLSession` data download
- Schedule next refresh using `WKApplication.shared().scheduleBackgroundRefresh(withPreferredDate:userInfo:scheduledCompletion:)`
- Budgeted by the system; prefer lightweight updates and defer heavy work

### Smart Stack Widgets
- Smart stack is the new complication surface in watchOS 9; widgets appear and reorder by relevance **[NEW]**
- Built with WidgetKit: `Widget` conforming type with `WKWidgetBundle` or standalone `@main Widget`
- `WidgetConfiguration`: `.accessoryCircular`, `.accessoryRectangular`, `.accessoryInline`, `.accessoryCorner` families — watchOS widget families **[NEW]**
- `TimelineProvider` drives content updates; returning `TimelineEntry` objects with dates
- `WidgetRelevance` / `WidgetRelevanceEntry` — provide relevance scores so the system decides smart stack order **[NEW]**
- Tapping a widget deep-links into the app via widget URL set with `.widgetURL(_:)`

### Complications & WidgetKit Migration
- ClockKit complications replaced by WidgetKit widgets for watchOS 9 **[NEW approach]**
- `CLKComplicationDataSource` deprecated in favor of WidgetKit `TimelineProvider`
- Complication families map to widget families: `.modularSmall` → `.accessoryCircular`, etc.
- Existing ClockKit complications continue to work; WidgetKit is recommended for new development

### Notifications
- `WKUserNotificationHostingController` — SwiftUI-based notification view controller **[existing, enhanced]**
- Actionable notifications via `UNNotificationCategory` with `UNNotificationAction` identifiers
- `userNotificationCenter(_:didReceive:withCompletionHandler:)` in `WKApplicationDelegate` handles notification actions
- Notification payloads drive SwiftUI notification views via `notificationCenter.add(_:)`

### Workout Sessions
- `HKWorkoutSession` and `HKLiveWorkoutBuilder` — start, pause, end workout sessions from watchOS app
- `HKWorkoutConfiguration` — specifies activity type and location type
- `HKWorkoutSession.startActivity(with:)` / `end()`
- `HKLiveWorkoutBuilder.beginCollection(withStart:completion:)` / `endCollection(withEnd:completion:)`
- Workout sessions keep the app running in the foreground and prevent it from being suspended

## APIs & Frameworks

**SwiftUI (watchOS 9)** **[NEW on watchOS]**
- `NavigationStack` on watchOS **[NEW]**
- `NavigationSplitView` on watchOS **[NEW]**
- `.toolbar` modifier, `ToolbarItem` with `.topBarTrailing` placement

**WidgetKit (watchOS 9)** **[NEW]**
- `WidgetFamily.accessoryCircular` **[NEW]**
- `WidgetFamily.accessoryRectangular` **[NEW]**
- `WidgetFamily.accessoryInline` **[NEW]**
- `WidgetFamily.accessoryCorner` **[NEW]**
- `WidgetRelevance` / `WidgetRelevanceEntry` **[NEW]**
- `.widgetURL(_ url: URL)` view modifier — deep-link target from tapping widget

**WatchKit / WKApplication**
- `WKApplicationRefreshBackgroundTask` — background refresh task
- `WKURLSessionRefreshBackgroundTask` — background URL session task
- `WKApplication.shared().scheduleBackgroundRefresh(withPreferredDate:userInfo:scheduledCompletion:)`
- `WKApplicationDelegate` — `handle(_ backgroundTasks:)` for background task handling

**HealthKit**
- `HKWorkoutSession` — manage workout lifecycle
- `HKLiveWorkoutBuilder` — collect live workout data
- `HKWorkoutConfiguration(activityType:locationType:)`
- `HKWorkoutSession.startActivity(with date:)` / `.end()`
- `HKLiveWorkoutBuilder.beginCollection(withStart:completion:)` / `.endCollection(withEnd:completion:)`

**UserNotifications**
- `UNNotificationCategory` — defines actionable notification categories
- `UNNotificationAction` — action in a notification category
- `WKUserNotificationHostingController` — SwiftUI notification view on watchOS

## Code Highlights

Scheduling a background refresh:
```swift
WKApplication.shared().scheduleBackgroundRefresh(
    withPreferredDate: Date(timeIntervalSinceNow: 15 * 60),
    userInfo: nil) { error in
        if let error { print("Schedule error: \(error)") }
}

// In WKApplicationDelegate:
func handle(_ backgroundTasks: Set<WKRefreshBackgroundTask>) {
    for task in backgroundTasks {
        switch task {
        case let refreshTask as WKApplicationRefreshBackgroundTask:
            // Update model, then schedule next refresh
            refreshTask.setTaskCompletedWithSnapshot(false)
        default:
            task.setTaskCompletedWithSnapshot(false)
        }
    }
}
```

WidgetKit timeline provider with relevance:
```swift
struct TaskWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: "TaskWidget", provider: TaskProvider()) { entry in
            TaskWidgetView(entry: entry)
        }
        .configurationDisplayName("Tasks")
        .supportedFamilies([.accessoryCircular, .accessoryRectangular])
    }
}

struct TaskProvider: TimelineProvider {
    func getTimeline(in context: Context, completion: @escaping (Timeline<TaskEntry>) -> Void) {
        let entries = buildEntries()
        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
}
```

Starting a workout session:
```swift
let config = HKWorkoutConfiguration()
config.activityType = .running
config.locationType = .outdoor
let session = try HKWorkoutSession(healthStore: healthStore, configuration: config)
let builder = session.associatedWorkoutBuilder()
builder.dataSource = HKLiveWorkoutDataSource(healthStore: healthStore, workoutConfiguration: config)
session.startActivity(with: Date())
try await builder.beginCollection(at: Date())
```

## Takeaways
- Adopt `NavigationStack` and `NavigationSplitView` on watchOS 9 to give productivity apps richer, multi-level navigation instead of flat paging.
- Use WidgetKit widget families (`.accessoryCircular`, `.accessoryRectangular`, `.accessoryInline`) instead of ClockKit for new watchOS 9 complication development; provide `WidgetRelevanceEntry` values to influence smart stack ordering.
- Background app refresh keeps data fresh between user sessions; schedule the next refresh at the end of each background task handler to maintain a regular cadence within system budget.
- Workout sessions are the correct mechanism for any app requiring extended foreground time on watchOS — they prevent suspension and give access to live HealthKit data via `HKLiveWorkoutBuilder`.

---
_Source: WWDC22 Session 10133 page (abstract, transcript, and code samples)._
