# What's New in HealthKit
**WWDC22 · Session 10005** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10005/)

_Platforms:_ iOS 16, iPadOS 16, watchOS 9

## Overview
HealthKit gains four major areas of improvement in 2022: detailed sleep stage tracking, enhanced workout APIs for multi-sport and interval workouts, Swift async query support, and new vision prescription storage with per-prescription privacy controls.

Sleep tracking now captures three distinct stages (REM, core, deep) automatically via Apple Watch, with the existing `asleep` case deprecated in favor of the more explicit `asleepUnspecified`. The workout API is significantly richer, introducing `HKWorkoutActivity` for representing distinct activities within a single workout (e.g., swim-bike-run triathlons or interval training).

Vision prescriptions can now be stored directly in HealthKit as `HKGlassesPrescription` or `HKContactsPrescription` objects, with optional PDF/image attachments of the physical prescription. A new per-object authorization model ensures users explicitly grant access per prescription — not blanket access to all vision data.

## Key Topics

### Sleep Stages
Apple Watch automatically tracks REM, core, and deep sleep stages, stored as `HKCategoryTypeIdentifier.sleepAnalysis` samples with new enum values. `asleep` is deprecated in favor of `asleepUnspecified`. New predicate helper `predicateForSamples(with:)` makes per-stage queries easy. Use `.allAsleepValues` when querying for all sleep data.

### Enhanced Workout API
`HKWorkoutActivity` represents a distinct activity segment within a workout, with its own configuration, event list, and statistics. Activities cannot overlap and need not be contiguous. `addWorkoutActivity(_:to:)` on `HKWorkoutBuilder` adds activities; `beginNewActivity` / `endCurrentActivity` manage live sessions on Apple Watch. New query predicates allow filtering workouts by per-activity statistics (e.g., average heart rate above a threshold).

### New Running and Swimming Metrics
Automatically collected on Apple Watch Series 6, SE, and newer: running stride length, running power (watts), running vertical oscillation. For swimming: SWOLF score (strokes + seconds per lap), calculated per lap and segment event.

### Swift Async Query Support (iOS 15.4+)
Every `HKQuery` subclass now has a matching `HKQueryDescriptor`. `result(for:)` returns a single async result; `results(for:)` returns an `AsyncSequence` for live updates. Breaking out of the loop stops the query automatically.

### Cardio Recovery (Heart Rate Recovery)
New `HKQuantityTypeIdentifier.heartRateRecoveryOneMinute` tracks how quickly heart rate drops after exercise. Context metadata includes recovery test type, activity type, duration, and max heart rate. Automatically saved by `HKLiveWorkoutBuilder` on Apple Watch.

### Vision Prescriptions
New `HKVisionPrescription` base type with `HKGlassesPrescription` and `HKContactsPrescription` subclasses. Created from `HKGlassesLensSpecification` / `HKContactsLensSpecification` objects. Optional image/PDF attachments via `HKAttachmentStore`. New per-object authorization: users grant access per prescription individually via `requestPerObjectReadAuthorization`.

## APIs & Frameworks

**HealthKit**

_Sleep_
- `HKCategoryTypeIdentifier.sleepAnalysis` — sleep category type
- `HKCategoryValueSleepAnalysis.asleepREM` **[NEW]** — REM sleep stage
- `HKCategoryValueSleepAnalysis.asleepCore` **[NEW]** — core sleep (AASM stages 1–2)
- `HKCategoryValueSleepAnalysis.asleepDeep` **[NEW]** — deep sleep (AASM stage 3)
- `HKCategoryValueSleepAnalysis.asleepUnspecified` **[NEW]** — replaces deprecated `.asleep`
- `HKCategoryValueSleepAnalysis.allAsleepValues` **[NEW]** — predicate set for all sleep stages

_Workouts_
- `HKWorkoutActivity` **[NEW]** — represents a discrete activity within a workout
- `HKWorkoutBuilder.addWorkoutActivity(_:to:)` **[NEW]** — add activity to workout
- `HKWorkoutSession.beginNewActivity(configuration:date:metadata:)` **[NEW]** — start live activity on Watch
- `HKWorkoutSession.endCurrentActivity(on:)` **[NEW]** — end current live activity
- `HKWorkout.workoutActivities` **[NEW]** — array of all activities in the workout
- `HKWorkout.statistics(for:)` **[NEW]** — get statistics by quantity type (replaces deprecated properties)
- `HKWorkoutActivity.statistics(for:)` **[NEW]** — per-activity statistics
- `HKWorkoutActivityType.transition` **[NEW]** — activity type for transitions in multi-sport workouts
- `HKWorkoutActivityType.swimBikeRun` — multi-sport workout configuration type
- `predicateForWorkoutActivities(with:operatorType:quantityType:)` **[NEW]** — query workouts by activity statistics

_New Metrics_
- `HKQuantityTypeIdentifier.runningStrideLength` **[NEW]** — running stride length
- `HKQuantityTypeIdentifier.runningPower` **[NEW]** — running power in watts
- `HKQuantityTypeIdentifier.runningVerticalOscillation` **[NEW]** — vertical oscillation
- `HKQuantityTypeIdentifier.heartRateRecoveryOneMinute` **[NEW]** — cardio recovery metric
- SWOLF score for swimming **[NEW]** — per lap/segment for Watch swimming workouts

_Swift Async Queries_
- `HKQueryDescriptor` protocol — base for async query descriptors
- `HKStatisticsCollectionQueryDescriptor` — async stats collection query
- `result(for:)` — single async result method on query descriptors
- `results(for:)` — `AsyncSequence`-returning method for live updates

_Vision Prescriptions_
- `HKVisionPrescription` **[NEW]** — base class for vision prescriptions
- `HKGlassesPrescription` **[NEW]** — glasses prescription subclass
- `HKContactsPrescription` **[NEW]** — contacts prescription subclass
- `HKGlassesLensSpecification` **[NEW]** — per-eye glasses prescription data
- `HKContactsLensSpecification` **[NEW]** — per-eye contacts prescription data
- `HKObjectType.visionPrescriptionType()` **[NEW]** — vision prescription type
- `HKAttachmentStore` **[NEW]** — store and read file attachments (image/PDF)
- `HKAttachment` **[NEW]** — represents a file attachment on a health sample
- `HKHealthStore.requestPerObjectReadAuthorization(toRead:predicate:completion:)` **[NEW]** — per-object authorization

## Code Highlights

```swift
// Query for REM sleep samples
let sleepPredicate = HKQuery.predicateForSamples(withValue: .asleepREM)
let queryPredicate = HKSamplePredicate.categorySample(type: sleepType,
                                                       predicate: sleepPredicate)
let descriptor = HKSampleQueryDescriptor(predicates: [queryPredicate], sortDescriptors: [])
let samples = try await descriptor.result(for: healthStore)

// Weekly calorie totals with live updates
let descriptor = HKStatisticsCollectionQueryDescriptor(
    predicate: caloriesPredicate,
    options: .cumulativeSum,
    anchorDate: thisSunday,
    intervalComponents: DateComponents(weekOfYear: 1))
for try await collection in descriptor.results(for: healthStore) {
    // handle update
}

// Save heart rate recovery sample
let sample = HKQuantitySample(
    type: .heartRateRecoveryOneMinute,
    quantity: HKQuantity(unit: .count().unitDivided(by: .minute()), doubleValue: 50),
    start: startDate, end: endDate,
    metadata: [
        HKMetadataKeyHeartRateRecoveryTestType: HKHeartRateRecoveryTestType.maxExercise.rawValue,
        HKMetadataKeyHeartRateRecoveryActivityType: HKWorkoutActivityType.swimBikeRun.rawValue
    ])
```

## Takeaways

- Sleep stage data (REM, core, deep) is now available in HealthKit; update existing sleep queries to use `.allAsleepValues` to avoid missing stage data.
- `HKWorkoutActivity` enables rich multi-sport (swim-bike-run) and interval workout tracking with per-activity statistics; deprecated `totalSwimmingStrokeCount` and similar workout properties should be replaced with `statistics(for:)`.
- Vision prescriptions with image/PDF attachments can be stored in HealthKit; the new per-object authorization model gives users fine-grained control.
- Swift async query descriptors (`result(for:)` / `results(for:)`) provide a clean, concise alternative to closure-based HealthKit queries from iOS 15.4+.

---
_Source: WWDC22 Session 10005 page (abstract, chapter summaries, code samples, and resource links)._
