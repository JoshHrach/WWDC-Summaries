# New Ways to Work with Workouts
**WWDC18 · Session 707** · [Watch](https://developer.apple.com/videos/play/wwdc2018/707/)

_Platforms:_ iOS 12, watchOS 5

## Overview
This session presents sweeping updates to HealthKit's workout APIs in watchOS 5 and iOS 12. The centerpiece is `HKWorkoutBuilder` and its watchOS subclass `HKLiveWorkoutBuilder`, which together provide a single coherent API for the entire workout lifecycle — setup, active data collection, state control, saving, and crash recovery — replacing the previous patchwork of manual sample management.

A new `HKCumulativeQuantitySeriesSample` type is introduced to address the performance problem of storing thousands of individual high-frequency samples (e.g., per-second distance during a workout) as separate `HKQuantitySample` objects. The series type stores a single sample backed by many quantities, enabling efficient storage while retaining access to individual values via a companion query API.

The session opens with a strong emphasis on HealthKit privacy best practices: request only what you need, only when you need it, and always query HealthKit directly rather than caching authorization state.

## Key Topics

### HealthKit Privacy & Authorization
- Request only the types actually used by the app, and only when about to use them
- Never cache authorization state — always re-query `HKHealthStore` as source of truth
- Handle authorization resets (e.g., user resets location & privacy) by treating HealthKit as authoritative
- Use `HKHealthStore.requestAuthorization(toShare:read:)` at the point of need

### HKWorkoutBuilder (iOS 12 / watchOS 5)
- New class that manages the full workout lifecycle and saves an `HKWorkout` to HealthKit
- `beginCollection(withStart:completion:)` — starts data collection
- `add(_:completion:)` — adds `[HKSample]` to the workout
- `addWorkoutEvents(_:completion:)` — adds `[HKWorkoutEvent]`
- `addMetadata(_:completion:)` — adds `[String: Any]` metadata
- `endCollection(withEnd:completion:)` — stops data collection
- `finishWorkout(completion:)` — saves completed workout + all associated data
- `statistics(for:)` — returns `HKStatistics` (min/max/avg/mostRecent) for any collected quantity type
- `elapsedTime` property — accounts for pause/resume events automatically
- Delegates: `workoutBuilderDidCollectData(of:)` and `workoutBuilderDidCollectEvents(_:)`

### HKLiveWorkoutBuilder (watchOS 5)
- watchOS-only subclass of `HKWorkoutBuilder`; retrieved from an `HKWorkoutSession` via `associatedWorkoutBuilder()`
- Works in tandem with `HKWorkoutSession` for sensor activation and background running
- Supports `HKLiveWorkoutDataSource` for automatic sample collection based on activity type

### HKLiveWorkoutDataSource (watchOS 5)
- Automatically determines which quantity types to collect from the `HKWorkoutConfiguration`
- `enableCollection(for:predicate:)` and `disableCollection(for:)` for manual override

### Workout Session State Machine
- States: `.notStarted` → `.prepared` → `.running` ↔ `.paused` → `.stopped` → (ended)
- `.prepared` state: sensors active, background mode granted, but workout not yet started (useful for pre-workout countdown)
- `HKWorkoutSession.startActivity(with:)`, `.pause()`, `.resume()`, `.end()`

### Workout Recovery (watchOS 5)
- If app crashes during an active workout, watchOS automatically relaunches the app
- `WKExtensionDelegate.handleActiveWorkoutRecovery()` — called on relaunch
- `HKHealthStore.recoverActiveWorkoutSession(completion:)` — returns the recovered session in its previous state
- Builder is also recovered; do not call `startActivity` or `beginCollection` again
- Must re-attach `HKLiveWorkoutDataSource` after recovery

### HKCumulativeQuantitySeriesSample (iOS 12 / watchOS 5)
- New subclass of `HKQuantitySample` for storing high-frequency quantity data as one sample backed by multiple quantities
- Eliminates the overhead of storing thousands of individual samples
- Preserves temporal relationships between quantities within a series
- `HKStatisticsQuery` and `HKStatisticsCollectionQuery` automatically work with series samples (no migration needed)

### HKQuantitySeriesSampleQuery (iOS 12)
- New query type to retrieve individual `HKQuantity` values within a series sample
- Initialize with the series sample; quantities delivered in completion handler

### HKQuantitySeriesSampleBuilder (iOS 12 / watchOS 5)
- New builder for creating `HKCumulativeQuantitySeriesSample` objects
- `insert(_:for:)` — adds individual quantity values with timestamps
- `finishSeries(completion:)` — finalizes and saves the series sample

## APIs & Frameworks

**HealthKit** — `import HealthKit`
- `HKWorkoutBuilder` — **[NEW]** iOS 12 / watchOS 5; incremental workout construction (`add(_:completion:)`, `addWorkoutEvents`, `addMetadata`, `endCollection`, `finishWorkout`)
- `HKLiveWorkoutBuilder` — **[NEW]** watchOS 5; real-time workout building on Apple Watch
- `HKLiveWorkoutDataSource` — **[NEW]** watchOS 5; automatic live data collection from the workout session
- `HKWorkoutSession` — workout session lifecycle and state machine (`HKWorkoutSessionState`: `.running`, `.paused`, `.stopped`, `.ended`)
- `HKWorkoutConfiguration` — `activityType`, `locationType`
- `HKCumulativeQuantitySeriesSample` — **[NEW]** iOS 12 / watchOS 5; cumulative series sample
- `HKQuantitySeriesSampleQuery` — **[NEW]** iOS 12; retrieve individual quantities within a series
- `HKQuantitySeriesSampleBuilder` — **[NEW]** iOS 12 / watchOS 5; `insert(_:for:)`, `finishSeries(completion:)`
- `HKHealthStore` — authorization (`requestAuthorization`), saving/querying
- Workout recovery via `HKWorkoutSession` re-association after app relaunch (watchOS 5)

## Code Highlights

Setting up and starting a workout (watchOS):
```swift
let config = HKWorkoutConfiguration()
config.activityType = .running

let session = try HKWorkoutSession(healthStore: store, configuration: config)
let builder = session.associatedWorkoutBuilder()
builder.dataSource = HKLiveWorkoutDataSource(healthStore: store,
                                              workoutConfiguration: config)
session.startActivity(with: Date())
builder.beginCollection(withStart: Date()) { _, _ in }
```

Recovering a crashed workout session:
```swift
func handle(_ workoutSession: HKWorkoutSession) { }
// In WKExtensionDelegate:
func handleActiveWorkoutRecovery() {
    let store = HKHealthStore()
    store.recoverActiveWorkoutSession { session, error in
        // session is already running — do not call startActivity again
        // re-attach data source:
        session?.associatedWorkoutBuilder().dataSource =
            HKLiveWorkoutDataSource(healthStore: store, workoutConfiguration: session!.workoutConfiguration)
    }
}
```

Displaying live statistics:
```swift
func workoutBuilderDidCollectData(_ workoutBuilder: HKLiveWorkoutBuilder,
                                   collectedTypes: Set<HKSampleType>) {
    guard let hrType = HKQuantityType.quantityType(forIdentifier: .heartRate),
          collectedTypes.contains(hrType),
          let stats = workoutBuilder.statistics(for: hrType) else { return }
    let bpm = stats.mostRecentQuantity()?.doubleValue(for: .count().unitDivided(by: .minute()))
    // update UI
}
```

## Takeaways
- `HKWorkoutBuilder` / `HKLiveWorkoutBuilder` replaces manual sample assembly — adopt them for cleaner code and automatic data association.
- Workout crash recovery in watchOS 5 is a one-time delegate call and a single API call; every workout app should implement it to prevent data loss.
- Use `HKCumulativeQuantitySeriesSample` and its builder for any high-frequency data (per-second GPS, cadence, etc.) to minimize HealthKit storage overhead.
- Always re-query HealthKit for authorization state rather than caching — the user can revoke access at any time.

---
_Source: WWDC18 Session 707 page (abstract, chapter summaries, code samples, and resource links)._
