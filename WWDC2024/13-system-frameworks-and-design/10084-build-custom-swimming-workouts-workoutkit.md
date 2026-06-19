# Build Custom Swimming Workouts with WorkoutKit
**WWDC24 · Session 10084** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10084/)

_Platforms:_ watchOS 11, iOS 18 (WorkoutKit; workout execution on Apple Watch)

## Overview
This session covers new WorkoutKit APIs in watchOS 11 that expand custom workout support to include pool swimming. WorkoutKit allows developers to programmatically compose structured workouts with intervals, goals, and alerts, and then schedule them to run on Apple Watch's native Workout app—no custom workout UI needed on the watch side.

Pool swimming is a new workout type added to WorkoutKit in watchOS 11. The session walks through the new `poolSwimDistanceWithTime` goal type, custom step names for swim sets, distance-based goals for individual lengths or laps, and how swimming-specific metrics (SWOLF, stroke rate, distance per stroke) surface through `HKWorkoutActivity` and `HKQuantityType` after the workout.

## Key Topics
- **WorkoutKit recap** — compose `WorkoutPlan` with `WorkoutStep`, `WorkoutInterval`, `WorkoutGoal`, and `WorkoutAlert`; schedule via `WorkoutScheduler`; executes in native Workout app on Apple Watch
- **Pool swimming support (new)** — `WorkoutActivity(activityType: .swimming, locationType: .indoor)` for pool swims
- **`poolSwimDistanceWithTime` goal** — new goal type combining distance (yards/meters) and time targets for a swim set
- **Custom step names** — `WorkoutStep.displayName` lets developers label intervals with meaningful names (e.g., "Warm-Up Kick Set", "Main Set")
- **Distance goals for swimming** — `WorkoutGoal.distance(_:unit:)` with `.meter` or `.yard` for per-step yardage targets
- **Swimming metrics post-workout** — `HKQuantityType.swimmingStrokeCount`, `HKQuantityType.swolfScore`, `HKQuantityType.distanceSwimming` available in HealthKit after workout completion
- **Alert customization** — `WorkoutAlert` triggers for heart rate, pace, and cadence; swimming adds stroke rate alerts

## APIs & Frameworks
### WorkoutKit (watchOS 11 / iOS 18)
- `WorkoutPlan` — top-level container; holds a sequence of `WorkoutStep` and `WorkoutInterval`
- `WorkoutStep` — atomic unit; has a `goal`, optional `alert`, and new `displayName: String?` property
  - **[NEW] `displayName`** — custom label shown in Workout app during execution (e.g., "Kick Set")
- `WorkoutGoal` — defines completion criteria for a step
  - **[NEW] `.poolSwimDistanceWithTime(distance:unit:time:)`** — combined distance + time goal for pool swimming
  - `.distance(_:unit:)` — distance-only goal; `.meter` and `.yard` now explicitly supported for swimming
  - `.time(_:unit:)` — time-only goal
  - `.open` — no goal; user ends step manually
- `WorkoutAlert` — triggers notification during workout execution
  - `HeartRateAlert`, `PaceAlert`, `CadenceAlert` — existing alerts
  - **[NEW] `StrokeRateAlert`** — alerts on swimming stroke rate range
- `WorkoutActivity` — specifies workout type and location
  - `.swimming` with `.indoor` location type — pool swimming
- `WorkoutScheduler` — `schedule(_:)` sends plan to Apple Watch Workout app
- `WorkoutScheduler.workouts` — async sequence of scheduled plans
- `WorkoutScheduler.remove(_:)` — cancel a scheduled workout

### HealthKit (post-workout metrics)
- `HKQuantityType(.swimmingStrokeCount)` — total strokes per length
- `HKQuantityType(.swolfScore)` — SWOLF (stroke count + time for one length); lower is more efficient
- `HKQuantityType(.distanceSwimming)` — total swim distance in workout
- `HKWorkoutActivity` — sub-activity segments within a workout; swimming laps/sets map here
- `HKWorkout` — completed workout object; query via `HKWorkoutType` predicate

## Code Highlights
```swift
import WorkoutKit

// Build a pool swimming workout plan
let warmUp = WorkoutStep(
    goal: .time(300, unit: .second),
    displayName: "Warm-Up"  // NEW: custom step name
)

let mainSet = WorkoutStep(
    goal: .poolSwimDistanceWithTime(  // NEW: combined goal
        distance: 400, unit: .meter,
        time: 360
    ),
    displayName: "Main Set 400m"
)

let coolDown = WorkoutStep(
    goal: .distance(100, unit: .meter),
    displayName: "Cool Down"
)

let plan = WorkoutPlan(
    activity: WorkoutActivity(activityType: .swimming, locationType: .indoor),
    steps: [warmUp, mainSet, coolDown]
)

// Schedule to Apple Watch Workout app
let scheduler = WorkoutScheduler.shared
try await scheduler.schedule(plan)

// Post-workout: query SWOLF score
let swolfType = HKQuantityType(.swolfScore)
let query = HKSampleQuery(sampleType: swolfType, predicate: nil,
                           limit: HKObjectQueryNoLimit, sortDescriptors: nil) { _, samples, _ in
    for sample in samples ?? [] {
        print((sample as! HKQuantitySample).quantity)
    }
}
healthStore.execute(query)
```

## Takeaways
- Pool swimming is now a first-class WorkoutKit workout type in watchOS 11; the `poolSwimDistanceWithTime` goal makes it easy to create structured swim sets with combined time and yardage targets
- Custom `displayName` on `WorkoutStep` is a small but impactful addition—it gives swimmers meaningful interval labels instead of generic "Interval 1", "Interval 2" labels on the watch
- WorkoutKit's model (compose on iPhone/server, execute on watch) means you can build sophisticated swim coaching workflows without writing any watchOS UI code
- Post-workout SWOLF scores and stroke counts from HealthKit enable analytics and performance trending within your app

---
_Source: WWDC24 Session 10084 page (abstract, chapter summaries, code samples, and resource links)._
