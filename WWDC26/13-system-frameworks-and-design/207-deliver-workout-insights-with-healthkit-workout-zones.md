# Deliver workout insights with HealthKit workout zones
**WWDC26 · Session 207** · [Watch](https://developer.apple.com/videos/play/wwdc2026/207/)

_Platforms:_ iOS 27, watchOS 27

## Overview
iOS and watchOS 27 add first-class workout zone support directly into HealthKit. Heart rate zones and cycling power zones are now modelled as `HKWorkoutZoneGroup` objects stored on completed workouts and delivered in real time during active workouts, removing the need for apps to maintain their own zone calculation logic.

A zone group pairs an `HKWorkoutZoneConfiguration` (the quantity type, contiguous boundary values, and source) with an array of `HKWorkoutZoneDuration` objects representing cumulative time spent in each zone. Apps can read zones from a completed `HKWorkout` or `HKWorkoutActivity` via the new `zoneGroupsByType` dictionary, or receive live per-zone updates during a workout by implementing the new `HKLiveWorkoutDelegate` method `didUpdateWorkoutZone`.

Apps that use proprietary zone models can supply a `HKWorkoutZoneConfiguration` via `HKWorkoutBuilder.setCustomZoneConfiguration(_:for:)` before calling `beginCollection`. If no custom configuration is supplied, HealthKit automatically uses the preferred zones from Health Settings (which sync across devices and can be auto-calculated from user metrics like age and resting heart rate).

## Key Topics

### Accessing Workout Zones (Post-Workout)
`HKWorkout.zoneGroupsByType` is a dictionary keyed by `HKQuantityType` (e.g., `.heartRate`, `.cyclingPower`). Each value is an `HKWorkoutZoneGroup` containing the configuration and an array of zone durations — ideal for post-workout charts showing time-in-zone breakdowns.

### Live Zone Updates
Adopt `HKLiveWorkoutDelegate.workoutBuilder(_:didUpdateWorkoutZone:)` to receive `HKLiveWorkoutZoneUpdate` objects as the user's metric crosses zone boundaries during an active workout. Each update includes the current zone index, cumulative zone durations so far, and the timestamp of the last processed sample — enabling real-time coaching cues.

### Preferred Zones
HealthKit uses thresholds from Health Settings by default. Before starting a workout that relies on zone data, call `HKWorkoutBuilder.zoneConfiguration(for:)` (async) to verify a preferred configuration exists. If it returns `nil`, fall back to a custom configuration.

### Custom Zones
Create an `HKWorkoutZoneConfiguration` with an `HKQuantityType` and an array of `HKQuantity` boundary values, then call `builder.setCustomZoneConfiguration(_:for:)` before `beginCollection`. Custom configurations apply only to that workout session and are not persisted to Health Settings.

## APIs & Frameworks

### HealthKit
- `HKWorkout.zoneGroupsByType` **[NEW]** — `[HKQuantityType: HKWorkoutZoneGroup]?` dictionary on completed workouts
- `HKWorkoutActivity.zoneGroupsByType` **[NEW]** — same, on individual workout activities
- `HKWorkoutZoneGroup` **[NEW]** — groups a configuration with zone duration data
  - `.configuration: HKWorkoutZoneConfiguration`
  - `.zoneDurations: [HKWorkoutZoneDuration]`
- `HKWorkoutZoneConfiguration` **[NEW]** — describes a zone scheme: `init(quantityType:zoneBoundaries:)`
  - `.zones: [HKWorkoutZone]` — array of zone descriptors
  - `.quantityType: HKQuantityType`
- `HKWorkoutZone` **[NEW]** — a single zone; `.index: Int`
- `HKWorkoutZoneDuration` **[NEW]** — `.duration: TimeInterval`, `.zone: HKWorkoutZone`
- `HKLiveWorkoutBuilder.zoneConfiguration(for:)` **[NEW]** — async; returns `HKWorkoutZoneConfiguration?` for preferred zones
- `HKLiveWorkoutBuilder.setCustomZoneConfiguration(_:for:)` **[NEW]** — sets a custom zone scheme before `beginCollection`
- `HKLiveWorkoutBuilder.beginCollection(at:)` — existing; must be called after setting custom zones
- `HKLiveWorkoutDelegate.workoutBuilder(_:didUpdateWorkoutZone:)` **[NEW]** — real-time zone change callback
- `HKLiveWorkoutZoneUpdate` **[NEW]** — payload in the live callback:
  - `.zoneGroup: HKWorkoutZoneGroup?`
  - `.currentZoneDuration: HKWorkoutZoneDuration?` — includes `.zone.index` for the active zone
- `HKQuantityType(.heartRate)` — existing quantity type; now supported in zone groups
- `HKQuantityType(.cyclingPower)` — existing quantity type; now supported in zone groups
- `HKUnit.count().unitDivided(by: HKUnit.minute())` — BPM unit for heart rate boundary values
- `HKQuantity(unit:doubleValue:)` — used to express zone boundary thresholds

## Code Highlights

Read zones from a completed workout:
```swift
if let hrGroup = workout.zoneGroupsByType?[HKQuantityType(.heartRate)] {
    let durations = hrGroup.zoneDurations.map(\.duration)
    // render time-in-zone chart
}
```

Handle live zone updates during a workout:
```swift
func workoutBuilder(_ workoutBuilder: HKLiveWorkoutBuilder,
                    didUpdateWorkoutZone zoneUpdate: HKLiveWorkoutZoneUpdate) {
    guard let zoneGroup = zoneUpdate.zoneGroup,
          let currentIndex = zoneUpdate.currentZoneDuration?.zone.index else { return }
    let durations = zoneGroup.zoneDurations.map(\.duration)
    // update UI with currentIndex and durations
}
```

Set a custom zone configuration before starting:
```swift
let boundaries = [91.0, 114.0, 136.0, 158.0].map {
    HKQuantity(unit: .count().unitDivided(by: .minute()), doubleValue: $0)
}
let config = try HKWorkoutZoneConfiguration(
    quantityType: HKQuantityType(.heartRate), zoneBoundaries: boundaries)
try await builder.setCustomZoneConfiguration(config, for: HKQuantityType(.heartRate))
try await builder.beginCollection(at: Date())
```

## Takeaways
- `HKWorkout.zoneGroupsByType` provides ready-made post-workout zone data — no need to maintain zone logic in your app.
- `HKLiveWorkoutDelegate.workoutBuilder(_:didUpdateWorkoutZone:)` enables real-time coaching as the user moves between zones.
- Always query `builder.zoneConfiguration(for:)` before starting; supply a `setCustomZoneConfiguration` fallback when preferred zones are not set.
- Custom configurations are per-workout and not persisted; use them for proprietary zone models without altering the user's Health Settings.

---
_Source: WWDC26 Session 207 page (abstract, chapter summaries, code samples, and resource links)._
