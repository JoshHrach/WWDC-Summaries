# Build Custom Workouts with WorkoutKit
**WWDC23 · Session 10016** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10016/)

_Platforms:_ iOS 17, watchOS 10

## Overview
This session introduces **WorkoutKit** — a brand-new Swift framework in iOS 17 and watchOS 10 that lets third-party apps create, preview, and schedule structured workouts for the Workout app on Apple Watch. WorkoutKit covers all four workout composition types: goal-based, pacer, swim-bike-run, and custom (the focus of this session). Custom workouts are structured sequences of warmup, repeatable interval blocks, and cooldown steps, each with their own goal and optional alert.

Beyond building workout compositions, WorkoutKit provides a built-in preview UI for showing workout details to the user and a scheduling system that gives an app a dedicated section at the top of the Apple Watch Workout app, listing the app's synced upcoming workouts.

## Key Topics

### Custom Workout Structure
A `CustomWorkoutComposition` has three sections:
- **Warmup step** (`WarmupStep`) — one step at the start with a goal (open, time, or distance) and an optional alert.
- **Blocks** (`IntervalBlock` array) — ordered, repeatable interval blocks, each containing any number of `BlockStep` elements (`.work` or `.rest` type), each with their own goal and alert. Blocks specify an iteration count.
- **Cooldown step** (`CooldownStep`) — one step at the end.

### Goals
`WorkoutGoal` — wraps a specific goal:
- `.open` — user manually progresses (default for warmup/cooldown if unspecified).
- `.time(HKQuantity)` — time-based goal.
- `.distance(HKQuantity)` — distance-based goal (not supported for non-distance activities).

### Alerts
`WorkoutAlert` — combines a `WorkoutAlertType` and a `WorkoutAlertTargetType`:
- **Types**: current pace, cadence, power, heart rate.
- **Target types**: `.target(HKQuantity)` (single value) or `.range(HKQuantity, HKQuantity)` (lower/upper bound).
- Alert availability depends on activity type — e.g., pace alerts not available for elliptical; distance goals not valid for non-distance activities.

### Validation
- Creating a `CustomWorkoutComposition` with `try` triggers validation. WorkoutKit provides granular `WorkoutCompositionError` values indicating exactly which constraint is violated.
- Validation also runs on export and other API calls.

### Preview UI **[NEW]**
- `WorkoutComposition.presentPreview()` — async call that presents an out-of-process preview UI:
  - **iOS**: system sheet showing workout title, list of steps/blocks with goals and alerts, and an "Add to Watch" button to save the workout to the Workout app.
  - **watchOS**: launches the Workout app with the composition loaded, ready to start or save.

### Export and Import
- `WorkoutComposition.dataRepresentation(encoding:)` — export to `Data` in `.json` or `.binary` (recommended: binary is significantly smaller); file extension `.workout`.
- `WorkoutComposition(data:)` — import from data.

### Scheduling Workouts **[NEW]**
- `WorkoutPlan` — interface to the Workout app's scheduled workout list for your app:
  - `WorkoutPlan.authorizationState` — check current permission state.
  - `WorkoutPlan.requestAuthorization()` — prompt user; opt-in required.
  - `WorkoutPlan.current` — async property returning the current plan.
  - `WorkoutPlan.scheduledCompositions` — array of `ScheduledWorkoutComposition` objects.
  - `WorkoutPlan.save()` — sync pending changes to the Workout app.
- `ScheduledWorkoutComposition` — pairs a `WorkoutComposition` with a `scheduledDate: Date` and a `completionState`.
- Users see workouts in a dedicated app section at the top of Workout; up to 15 synced at once; visible ±7 days.
- Query completed workouts via `WorkoutPlan.current`; check `completionState` per composition.
- `HKWorkout` extension — `workoutComposition` property retrieves the associated `WorkoutComposition` from a completed HealthKit workout.

## APIs & Frameworks

### WorkoutKit **[NEW — entire framework]**
- `CustomWorkoutComposition` — structured custom workout with warmup, blocks, cooldown **[NEW]**
- `GoalWorkoutComposition` — single-goal workout (distance, time, energy) **[NEW]**
- `PacerWorkoutComposition` — pace/speed-focused workout **[NEW]**
- `SwimBikeRunWorkoutComposition` — triathlon multisport workout **[NEW]**
- `WarmupStep` — warmup step with goal and optional alert **[NEW]**
- `CooldownStep` — cooldown step with goal and optional alert **[NEW]**
- `IntervalBlock` — repeatable block of `BlockStep` elements **[NEW]**
- `BlockStep` — single step within a block; type `.work` or `.rest` **[NEW]**
- `WorkoutGoal` — `.open`, `.time(HKQuantity)`, `.distance(HKQuantity)` **[NEW]**
- `WorkoutAlert` — alert combining type and target **[NEW]**
- `WorkoutAlertType` — `.currentPace`, `.cadence`, `.power`, `.heartRate` **[NEW]**
- `WorkoutAlertTargetType` — `.target(HKQuantity)`, `.range(HKQuantity, HKQuantity)` **[NEW]**
- `WorkoutComposition` — wrapper for performing operations on any composition type **[NEW]**
- `WorkoutComposition.presentPreview()` async — show preview UI (iOS sheet or watchOS Workout app) **[NEW]**
- `WorkoutComposition.dataRepresentation(encoding:)` — export to `.json` or `.binary` **[NEW]**
- `WorkoutComposition(data:)` — import from data **[NEW]**
- `WorkoutPlan` — scheduling interface to the Workout app **[NEW]**
- `WorkoutPlan.authorizationState` — current authorization state **[NEW]**
- `WorkoutPlan.requestAuthorization()` async — request scheduling permission **[NEW]**
- `WorkoutPlan.current` async — fetch current workout plan **[NEW]**
- `WorkoutPlan.scheduledCompositions` — array of scheduled compositions **[NEW]**
- `WorkoutPlan.save()` async — sync to Workout app **[NEW]**
- `ScheduledWorkoutComposition` — composition + scheduled date + completion state **[NEW]**
- `WorkoutCompositionError` — granular validation errors **[NEW]**
- `HKWorkout.workoutComposition` — retrieve composition from HealthKit workout **[NEW]**

### HealthKit (supporting)
- `HKQuantity` — wraps numeric values with units for goals and alerts (existing)
- `HKUnit` — unit of measure (e.g., `.mile()`, `.hour()`, `.watt()`) (existing)
- `HKWorkout` — HealthKit workout sample; extended with `workoutComposition` **[extended]**

## Code Highlights

Building a custom cycling workout with blocks and alerts:
```swift
import WorkoutKit
import HealthKit

// Warmup — open goal
let warmupStep = WarmupStep()

// Block 1: 2-mile work step with pace alert, 0.5-mile recovery with HR zone 1
let twoMileGoal = WorkoutGoal.distance(HKQuantity(unit: .mile(), doubleValue: 2))
let paceTarget = WorkoutAlertTargetType.target(HKQuantity(unit: .mile().unitDivided(by: .hour()), doubleValue: 10))
let paceAlert = WorkoutAlert(targetType: paceTarget, type: .currentPace)
let workStep1 = BlockStep(.work, goal: twoMileGoal, alert: paceAlert)

let halfMileGoal = WorkoutGoal.distance(HKQuantity(unit: .mile(), doubleValue: 0.5))
let hrAlert = WorkoutAlert(targetType: .target(HKQuantity(unit: .count().unitDivided(by: .minute()), doubleValue: 120)), type: .heartRate)
let recoveryStep1 = BlockStep(.rest, goal: halfMileGoal, alert: hrAlert)

let block1 = IntervalBlock(steps: [workStep1, recoveryStep1], iterations: 4)

// Cooldown — 5-minute goal
let fiveMinuteGoal = WorkoutGoal.time(HKQuantity(unit: .minute(), doubleValue: 5))
let cooldownStep = CooldownStep(goal: fiveMinuteGoal)

// Assemble
let composition = try CustomWorkoutComposition(
    activity: .cycling,
    location: .outdoor,
    displayName: "My Workout",
    warmup: warmupStep,
    blocks: [block1],
    cooldown: cooldownStep
)
```

Presenting preview and scheduling:
```swift
let workoutComposition = WorkoutComposition(composition)

// Preview (iOS: sheet; watchOS: launches Workout app)
await workoutComposition.presentPreview()

// Schedule
let plan = await WorkoutPlan.current
let scheduled = ScheduledWorkoutComposition(composition: workoutComposition, scheduledDate: .now)
plan.scheduledCompositions.append(scheduled)
await plan.save()
```

## Takeaways
- WorkoutKit is entirely new in watchOS 10 / iOS 17 — it's the first public API for creating and scheduling structured workouts in the Workout app.
- Custom workouts support four alert types (pace, cadence, power, heart rate) — but alert and goal availability is activity-type-specific; always handle validation errors.
- `presentPreview()` provides a ready-made UI for user review and one-tap "Add to Watch" — no custom UI needed for the preview flow.
- Scheduling gives your app a dedicated section in the Workout app; sync up to 15 workouts at a time and query completion state to keep your app in sync.

---
_Source: WWDC23 Session 10016 page (abstract, chapter summaries, code samples, and resource links)._
