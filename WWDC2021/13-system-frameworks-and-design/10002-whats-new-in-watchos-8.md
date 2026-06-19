# What's New in watchOS 8
**WWDC21 · Session 10002** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10002/)

_Platforms:_ watchOS 8, Apple Watch Series 5+

## Overview
watchOS 8 expands Apple Watch capabilities in four main directions: bringing the Always-On Display to third-party apps for the first time (previously only the watch face used it), enabling background delivery of HealthKit data and Bluetooth scanning to keep complications current without active sessions, adding region-based user notifications to watchOS, and delivering a broad set of SwiftUI and API enhancements.

The session is particularly developer-relevant for the `isLuminanceReduced` environment property, `TimelineView` schedules, background HealthKit delivery, Bluetooth background app refresh, and the new `UNLocationNotificationTrigger` on watchOS.

## Key Topics

### Always-On Display for Apps
When rebuilt with the watchOS 8 SDK, third-party app UIs are shown in a dimmed state (instead of blurred) when the wrist is down; they are immediately interactive on tap. The system reduces overall display brightness automatically.

`isLuminanceReduced` — new SwiftUI `Environment` property that is `true` in the always-on state. Use it to dim non-essential elements, redact private information (balances, account numbers), and remove or pause animations. Design principles: keep the UI's spatial layout consistent between active and dimmed states; only dim secondary content, keep key information prominent.

**Update frequencies**: apps with an active session (workout, audio) can update up to once per second; apps without an active session can update up to once per minute.

### TimelineView for Scheduled Updates
`TimelineView(schedule:)` — constructs a view that rerenders at scheduled dates even when the app is not in the foreground. Built-in schedules:
- `.everyMinute` — updates at the top of each minute (aligned to system clock)
- `.periodic(from:by:)` — updates on a custom interval (not clock-aligned)
- `.explicit(_:)` — updates at specific dates
- `.animation` — for frame-by-frame animations

Custom schedules allow logic like "every minute but every second in the last 60 seconds," though sub-minute updates outside an active session are not guaranteed.

### Background Delivery of HealthKit Data
`HKObserverQuery` with background delivery is now supported on watchOS 8. When a predicate matches new health data, the app is woken to receive results — up to once per hour without complications, up to four times per hour if complications are on the active watch face. Critical health data types (fall events, low SpO2, heart rate events) are delivered immediately. All wakes count against the background app refresh budget.

### Bluetooth Scanning During Background App Refresh
Core Bluetooth connections can now be initiated during background app refresh opportunities granted to complications. Apps with complications on the active watch face get up to four opportunities per hour. Initial connection must be made in the foreground. The new `WKRefreshBackgroundTask.expirationHandler` property notifies the app when background time is nearly exhausted.

### Region-Based User Notifications
`UNLocationNotificationTrigger` **[NEW on watchOS]** — delivers local notifications when the user enters or exits a geofence, without requiring an iPhone. "When in use" location permission is required. For privacy, users first see a static notification and can tap to see the full dynamic content. Best practice: limit regions to nearby or explicitly chosen points of interest; regions of ~200m radius are most power-efficient.

### Location Button on watchOS
`CLLocationButton` (Core Location) is now available on watchOS 8, granting one-time location authorization on tap without repeated permission prompts.

### Always-On Altimeter
New Core Motion API provides real-time elevation updates with minimal battery impact and without requiring location permission.

### SwiftUI and UI Enhancements
- `.searchable(text:suggestions:)` — search fields with customized suggestions on watchOS
- List swipe actions — custom swipe actions (e.g., favorite) alongside delete
- `Button(role: .destructive)` + `.controlProminence(.increased)` — destructive buttons with extra haptic
- `Canvas` view — programmatic GPU-accelerated drawing available on watchOS 8
- Large titles on scrolling views — consistent with iOS
- Text input revamp — remembers preferred input method (Scribble vs. Dictation) per app

### Accessibility
- **AssistiveTouch** — recognizes hand gestures (for users with limb differences) to navigate UI without touching the screen
- **Large accessibility text size** added to watchOS 8
- Unit and UI testing for Watch apps (introduced in Xcode 12.5) benefits from new accessibility features

### Health Data
- **Respiratory rate during sleep** — new HealthKit data type available to third-party apps

## APIs & Frameworks

- `isLuminanceReduced` SwiftUI environment property **[NEW]** — `true` in the Always-On state
- `TimelineView(_:content:)` **[NEW]** — schedule-driven view updates for always-on state
- `TimelineSchedule` — `.everyMinute`, `.periodic(from:by:)`, `.explicit(_:)`, `.animation`, custom **[NEW]**
- `HKObserverQuery` with `enableBackgroundDelivery(for:frequency:withCompletion:)` **[NEW on watchOS]**
- `WKRefreshBackgroundTask.expirationHandler` **[NEW]** — notified when background time is nearly up
- `UNLocationNotificationTrigger` **[NEW on watchOS]** — geofence-based local notifications
- `CLLocationButton` **[NEW on watchOS]** — one-time location authorization button
- Core Motion always-on altimeter API **[NEW]** — real-time elevation without location permission
- `.searchable(text:suggestions:)` SwiftUI modifier **[NEW on watchOS]**
- `Button(role: .destructive)` **[NEW]** — red-tinted destructive button
- `.controlProminence(.increased)` **[NEW]** — extra haptic feedback for prominent buttons
- `Canvas` **[NEW on watchOS]** — GPU-accelerated programmatic drawing
- AssistiveTouch — gesture-based navigation for users with limb differences **[NEW]**

## Code Highlights

Responding to Always-On state:
```swift
@Environment(\.isLuminanceReduced) var isLuminanceReduced

var body: some View {
    ZStack {
        mainContent
        if isLuminanceReduced {
            Color.black.opacity(0.5)
        }
        sensitiveInfo
            .redacted(reason: isLuminanceReduced ? .privacy : [])
    }
}
```

TimelineView updating every minute:
```swift
TimelineView(.everyMinute) { context in
    WorkoutMetricsView(currentDate: context.date)
}
```

Previewing Always-On state in Xcode:
```swift
ContentView()
    .environment(\.isLuminanceReduced, true)
```

## Takeaways

- `TimelineView` with `isLuminanceReduced` is the foundation for any watchOS 8 Always-On experience — all apps with glanceable information should adopt it.
- Background HealthKit delivery and Bluetooth background app refresh give complications a path to real-time data without requiring active sessions; they share the background budget so use them judiciously.
- Region-based notifications (geofences) are now first-class on watchOS — a powerful tool for location-aware apps targeting users who go out with only their watch.
- AssistiveTouch and large text support mean accessible Watch apps are now measurably more testable and broadly usable.

---
_Source: WWDC21 Session 10002 page (abstract, chapter summaries, code samples, and resource links)._
