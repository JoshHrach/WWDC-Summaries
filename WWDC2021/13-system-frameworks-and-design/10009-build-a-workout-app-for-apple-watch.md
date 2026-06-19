# Build a Workout App for Apple Watch
**WWDC21 · Session 10009** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10009/)

_Platforms:_ watchOS 8

## Overview
This code-along session builds a complete watchOS workout app from scratch using SwiftUI and HealthKit. The app covers the full lifecycle: a start screen with a carousel list of workout types, a three-panel in-session TabView (controls, metrics, Now Playing), a summary sheet with Activity Rings, and proper Always On state support. A single `WorkoutManager` observable object bridges HealthKit to all views via the SwiftUI environment.

The session demonstrates two key watchOS-specific SwiftUI additions for 2021: `TimelineView` for continuously updating workout metrics, and `isLuminanceReduced` for adapting the UI to the Always On (low-luminance) state.

## Key Topics

### App Architecture
- `WorkoutManager: NSObject, ObservableObject` — central model; manages `HKWorkoutSession`, `HKLiveWorkoutBuilder`, and all published metric values
- Injected as an environment object from the root `NavigationView`; all views read it via `@EnvironmentObject`
- Published properties: `selectedWorkout`, `running`, `heartRate`, `activeEnergy`, `distance`, `averageHeartRate`, `workout`, `showingSummaryView`

### SwiftUI Views
- **StartView**: `List` with `.listStyle(.carousel)` — gives the depth-effect scroll typical of watchOS workout apps
- **SessionPagingView**: `TabView` with three pages — `ControlsView` (End/Pause), `MetricsView`, `NowPlayingView`; `selection` defaults to `.metrics` so metrics are shown first
- **MetricsView**: `VStack` with `ElapsedTimeView`, active energy, heart rate, and distance; `.scenePadding()` aligns to navigation bar; `ignoresSafeArea(edges: .bottom)` allows content to reach screen edge
- **NowPlayingView**: provided directly by WatchKit — import WatchKit and use `WKInterfaceController.reloadRootPageControllers` or simply place `NowPlayingView()` (SwiftUI wrapper built into WatchKit)
- **SummaryView**: `ScrollView` of `SummaryMetricView` items + `ActivityRingsView` + Done button; presented as a sheet

### HealthKit Integration
- `HKWorkoutConfiguration` — set activity type and location type (`.outdoor` vs `.indoor` affects sensor behavior)
- `HKWorkoutSession` — starts/pauses/resumes/ends the workout; runs app in background during active session
- `HKLiveWorkoutBuilder` — automatically collects samples from active session sensors; call `beginCollection(withStart:completion:)` and `endCollection(withEnd:completion:)` then `finishWorkout(completion:)`
- `HKLiveWorkoutDataSource` — provides live data from session; assign to `builder.dataSource`
- Delegate: `HKLiveWorkoutBuilderDelegate.workoutBuilder(_:didCollectDataOf:)` — called per new sample type; read `builder.statistics(for:)` for current statistics
- `HKStatistics.mostRecentQuantity()`, `.averageQuantity()`, `.sumQuantity()` — extract values per quantity type
- `HKHealthStore.requestAuthorization(toShare:read:completion:)` — must request share permission for `.workoutType()` and read permission for activity summary, heart rate, energy, distance

### Always On State Support
- `isLuminanceReduced` environment variable — `true` when device is in Always On (low-luminance) state
- `TimelineView(_:content:)` — wraps content that should update on a schedule; context provides `.cadence` (`.live` vs `.lowFrequency`)
- **Custom `TimelineSchedule`**: `MetricsTimelineSchedule` — normal mode: 30 Hz; `lowFrequency` mode (Always On): 1 Hz
- `ElapsedTimeView.showSubseconds` driven by `context.cadence == .live` — hides tenths/hundredths in Always On
- Page indicator: `tabViewStyle(.page(indexDisplayMode: isLuminanceReduced ? .never : .automatic))` — hides dots in Always On

### Activity Rings
- `ActivityRingsView` wraps `WKInterfaceActivityRing` via `WKInterfaceObjectRepresentable`
- Query current activity summary with `HKActivitySummaryQuery(predicate:updateHandler:)` and assign to `activityRingsObject.setActivitySummary(_:animated:)`

## APIs & Frameworks

### HealthKit
- `HKWorkoutConfiguration()` — `.activityType`, `.locationType`
- `HKWorkoutSession(healthStore:configuration:)` — `.startActivity(with:)`, `.pause()`, `.resume()`, `.end()`
- `HKWorkoutSessionDelegate` — `workoutSession(_:didChangeTo:from:with:)` — state changes
- `HKLiveWorkoutBuilder` — `session.associatedWorkoutBuilder()`
  - `dataSource: HKLiveWorkoutDataSource`
  - `beginCollection(withStart:completion:)`, `endCollection(withEnd:completion:)`, `finishWorkout(completion:)`
  - `statistics(for: HKQuantityType) -> HKStatistics?`
  - `elapsedTime: TimeInterval` — elapsed seconds
- `HKLiveWorkoutBuilderDelegate` — `workoutBuilder(_:didCollectDataOf:)`, `workoutBuilderDidCollectEvent(_:)`
- `HKLiveWorkoutDataSource(healthStore:workoutConfiguration:)` — assign to `builder.dataSource`
- `HKActivitySummaryQuery(predicate:updateHandler:)` — for Activity Rings
- Capability requirements: HealthKit + Background Modes (Workout processing)
- Info.plist keys: `NSHealthShareUsageDescription`, `NSHealthUpdateUsageDescription`

### SwiftUI (watchOS)
- `TimelineView(_:content:)` — schedule-driven updating view **[NEW]**
  - `context.cadence: TimelineView.Context.Cadence` — `.live` or `.lowFrequency`
  - `context.date: Date` — current schedule tick date
- `isLuminanceReduced: Bool` — environment value for Always On state **[NEW]**
- `.listStyle(.carousel)` — depth-effect list scroll on watchOS
- `.scenePadding()` — aligns content to navigation bar inset
- `WKInterfaceObjectRepresentable` — protocol to wrap WatchKit UI objects in SwiftUI

### WatchKit
- `NowPlayingView` — SwiftUI view wrapping WatchKit's Now Playing interface
- `WKInterfaceActivityRing` — activity rings widget; `.setActivitySummary(_:animated:)`

## Code Highlights

Custom TimelineSchedule for Always On:
```swift
struct MetricsTimelineSchedule: TimelineSchedule {
    var startDate: Date
    func entries(from startDate: Date, mode: TimelineScheduleMode) -> PeriodicTimelineSchedule.Entries {
        PeriodicTimelineSchedule(
            from: self.startDate,
            by: mode == .lowFrequency ? 1.0 : 1.0 / 30.0
        ).entries(from: startDate, mode: mode)
    }
}
```

MetricsView wrapped in TimelineView:
```swift
TimelineView(MetricsTimelineSchedule(startDate: workoutManager.builder?.startDate ?? Date())) { context in
    VStack(alignment: .leading) {
        ElapsedTimeView(elapsedTime: workoutManager.builder?.elapsedTime ?? 0,
                        showSubseconds: context.cadence == .live)
        // ...other metrics
    }
}
```

Hiding page indicator in Always On:
```swift
@Environment(\.isLuminanceReduced) var isLuminanceReduced

TabView(selection: $selection) { ... }
    .tabViewStyle(.page(indexDisplayMode: isLuminanceReduced ? .never : .automatic))
    .onChange(of: isLuminanceReduced) { _ in displayMetricsView() }
```

## Takeaways
- `HKWorkoutSession` + `HKLiveWorkoutBuilder` handle all sensor collection and data saving automatically — the app only needs to react to delegate callbacks and update its published metric properties.
- `TimelineView` with a custom `TimelineSchedule` is the correct way to drive continuously updating workout metrics in SwiftUI; it also enables Always On support by reducing update frequency when `cadence == .lowFrequency`.
- `isLuminanceReduced` should hide non-essential UI (page indicators, subseconds) and simplify the layout — Always On display is a primary user experience on Apple Watch.
- A single `ObservableObject` (`WorkoutManager`) passed as an environment object cleanly separates HealthKit logic from SwiftUI views while keeping all views automatically reactive to state changes.

---
_Source: WWDC21 Session 10009 page (abstract, transcript, and code samples)._
