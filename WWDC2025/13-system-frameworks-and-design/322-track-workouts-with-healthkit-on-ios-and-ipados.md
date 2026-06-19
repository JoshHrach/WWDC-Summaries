# Track Workouts with HealthKit on iOS and iPadOS
**WWDC25 · Session 322** · [Watch](https://developer.apple.com/videos/play/wwdc2025/322/)

_Platforms:_ iOS 26, iPadOS 26

## Overview
HealthKit workout sessions come to iPhone and iPad in iOS 26 and iPadOS 26. Previously, `HKWorkoutSession` was exclusive to watchOS. This session covers how to start, manage, and persist workout sessions on iPhone/iPad, how to display Live Activities during workouts, how to handle crash recovery for long-running sessions, and how to enable Siri workout intents from the lock screen.

## Key Topics

### HKWorkoutSession on iPhone/iPad
Apps running on iOS 26 and iPadOS 26 can now create and manage `HKWorkoutSession` objects, just as watchOS apps have done since earlier OS versions. Sessions track workout state (prepared, started, paused, stopped, ended) and can be used to record high-frequency data like heart rate, pace, and power throughout the session. An active session keeps the workout context alive and enables system integrations.

### HKLiveWorkoutDataSource
`HKLiveWorkoutDataSource` provides a real-time stream of workout metrics (heart rate, active energy burned, distance, etc.) during an active session. Apps subscribe to the data source to power in-workout UI (metrics display, charting). The data source automatically associates data with the current session.

### Live Activities Integration
While a workout session is active, the app can publish a Live Activity (using ActivityKit) to the Dynamic Island and Lock Screen. This allows users to glance at current workout metrics (duration, heart rate, calories) without opening the app. The session demonstrates how to structure workout data as `ActivityAttributes` and update the activity as metrics change.

### Crash Recovery
A long-running workout session may outlive the app process (due to a crash or system termination). iOS 26 provides a recovery mechanism via the scene delegate: when the app restarts, it can detect an orphaned active session and reconnect to it. The session demonstrates how to query HealthKit for an interrupted session and resume workout tracking without data loss.

### Siri Workout Intents from Lock Screen
Workout-related App Intents (start workout, pause, end, etc.) can now be invoked by Siri directly from the lock screen without unlocking the device. The session explains how to declare workout intents as lock-screen accessible and how to handle the intent response in a session-aware way.

## APIs & Frameworks

**HealthKit**
- `HKWorkoutSession` on iOS/iPadOS **[NEW]** — workout session management previously available only on watchOS
- `HKLiveWorkoutDataSource` **[NEW on iOS/iPadOS]** — real-time live metric streaming (heart rate, active energy, distance, etc.)
- Workout session crash recovery via scene delegate **[NEW]** — detect and reconnect to interrupted sessions on app relaunch

**ActivityKit**
- Live Activities for workout sessions **[NEW integration]** — publish Dynamic Island / Lock Screen activity while a workout session is active

**App Intents / Siri**
- Workout intents from lock screen **[NEW]** — start, pause, end workout via Siri without unlocking device

## Code Highlights

```swift
// Create and start a workout session on iPhone
let config = HKWorkoutConfiguration()
config.activityType = .running
config.locationType = .outdoor

let session = try HKWorkoutSession(healthStore: healthStore, configuration: config)
let dataSource = HKLiveWorkoutDataSource(healthStore: healthStore, workoutSession: session)

session.startActivity(with: Date())

// Observe live metrics
for await statistics in dataSource.statisticsCollection {
    let heartRate = statistics.statistics(for: HKQuantityType(.heartRate))?.mostRecentQuantity()
    // Update UI
}
```

```swift
// Crash recovery in scene delegate
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, ...) {
    Task {
        let activeSessions = try await healthStore.workoutSession(for: .running)
        if let orphaned = activeSessions.first(where: { $0.state == .running }) {
            // Reconnect to the interrupted session
            self.workoutSession = orphaned
        }
    }
}
```

## Takeaways
- Adopt `HKWorkoutSession` on iOS 26 / iPadOS 26 to provide a first-class workout tracking experience on iPhone and iPad — no Apple Watch required.
- Use `HKLiveWorkoutDataSource` to stream real-time metrics into your workout UI.
- Publish a Live Activity when a session starts so users can monitor metrics from the Lock Screen and Dynamic Island.
- Implement crash recovery in your scene delegate to detect and reconnect to orphaned workout sessions, preventing data loss.
- Expose workout control intents to Siri and mark them as lock-screen accessible for hands-free workout management.

---
_Source: WWDC25 Session 322 page (abstract, chapter summaries, code samples, and resource links)._
