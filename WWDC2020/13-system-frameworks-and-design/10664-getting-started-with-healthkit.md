# Getting Started with HealthKit
**WWDC20 · Session 10664** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10664/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7

## Overview
This introductory session walks through building a step-count app called SmoothWalker using HealthKit from scratch. It covers the four foundational pillars of any HealthKit integration: enabling the capability, checking platform availability, creating an `HKHealthStore`, requesting authorization, reading data with queries, and writing data.

The session provides a clear taxonomy of all HealthKit object types (quantity samples, category samples, workouts, characteristics) and their shared structure (type, value, start/end time, metadata). The bulk of practical content focuses on three query types — `HKStatisticsQuery`, `HKStatisticsCollectionQuery`, and `HKSampleQuery` — demonstrating live data, statistics computation, and real-time updates via the collection query's update handler.

## Key Topics

**HealthKit Setup**
- Enable "HealthKit" capability in Xcode target → Signing & Capabilities
- Check `HKHealthStore.isHealthDataAvailable()` before assuming the platform supports HealthKit (not available on all Apple platforms)
- Create a single `HKHealthStore` instance and reuse it across the app lifecycle — multiple instances are unnecessary

**Authorization and Privacy**
- Read and write permissions are requested separately; the user can grant each independently
- Requires two Info.plist keys: `NSHealthShareUsageDescription` (read) and `NSHealthUpdateUsageDescription` (write)
- Call `requestAuthorization(toShare:read:completion:)` in context (e.g., `viewWillAppear` or onboarding) — not at launch blindly
- Authorization sheet lets users toggle each data type individually
- HealthKit is the source of truth — call `requestAuthorization` every time before accessing data, as permissions can change outside the app
- Success in the completion handler means authorization was successfully requested, not that it was granted

**HealthKit Object Taxonomy**
- `HKQuantitySample` — numerical value + unit (distance, steps, heart rate, calories)
- `HKCategorySample` — value from a predefined set, no unit (menstrual flow, sleep stage)
- `HKWorkout` — summarizes multiple values across multiple units (calories, distance, duration)
- `HKObject` characteristics — static data: birthday, blood type (read-only from Health app)
- All `HKObject` subclasses share: UUID, source device, source bundle identifier, metadata dictionary
- All `HKSample` subclasses add: `startDate`, `endDate`

**Aggregation Styles**
- Cumulative types (steps, distance, energy burned) — sum is meaningful; de-duplication prevents double-counting across devices
- Discrete types (heart rate, temperature, UV exposure) — average, min, max are meaningful; sum is not

**Writing Data**
- Create `HKQuantityType` → create `HKQuantity(unit:doubleValue:)` → create `HKQuantitySample` → call `HKHealthStore.save(_:withCompletion:)`
- HealthKit handles unit conversion automatically (e.g., meters on Apple Watch → yards in another app)

**Reading Data with Queries**
- `HKStatisticsQuery` — single statistic (sum, average, min, max) over a date range; handles de-duplication across sources
- `HKStatisticsCollectionQuery` — statistics grouped by fixed time interval (daily, weekly); runs live with update handler for real-time UI updates
- `HKSampleQuery` — raw sample retrieval with predicate, sort descriptor, and limit; no aggregation
- Execute queries via `HKHealthStore.execute(_:)`; stop long-running queries via `HKHealthStore.stop(_:)`
- All query results arrive on a background thread — dispatch to main thread before updating UI

**Live Updates**
- Set `HKStatisticsCollectionQuery.statisticsUpdateHandler` before executing to receive future updates
- Call `HKHealthStore.stop(_:)` in `viewWillDisappear` (or equivalent) to end background observation
- Query runs indefinitely in the background until stopped

## APIs & Frameworks

### HealthKit
- `HKHealthStore` — central gateway for all HealthKit operations
  - `isHealthDataAvailable()` — class method; returns false on unsupported platforms
  - `requestAuthorization(toShare:read:completion:)` — requests permissions; completion receives `Bool` success + optional error
  - `execute(_:)` — executes any `HKQuery` subclass
  - `stop(_:)` — terminates a running query
  - `save(_:withCompletion:)` — saves an `HKObject` (sample, workout) to the store

- `HKQuantityType` — represents a quantitative data type
  - `HKQuantityType.quantityType(forIdentifier:)` — look up by `HKQuantityTypeIdentifier`
  - Common identifiers: `.stepCount`, `.distanceWalkingRunning`, `.heartRate`, `.activeEnergyBurned`

- `HKQuantity` — stores a value + unit pair
  - `init(unit:doubleValue:)` — create a quantity
  - Common units: `HKUnit.count()`, `HKUnit.meter()`, `HKUnit.kilocalorie()`
  - `doubleValue(for:)` — retrieve value in a specified unit (with automatic conversion)

- `HKQuantitySample` — a time-stamped quantity measurement
  - `init(type:quantity:start:end:)` — primary initializer
  - `init(type:quantity:start:end:metadata:)` — with optional metadata dictionary

- `HKCategoryType` / `HKCategorySample` — categorical health data
  - `HKCategoryType.categoryType(forIdentifier:)` — look up by `HKCategoryTypeIdentifier`

- `HKStatisticsQuery` — single-range statistics on quantity samples
  - `init(quantityType:quantitySamplePredicate:options:completionHandler:)` — construct query
  - `options`: `HKStatisticsOptions` — `.cumulativeSum`, `.discreteAverage`, `.discreteMin`, `.discreteMax`, `.separateBySource`
  - Completion receives `HKStatistics?` with `.sumQuantity()`, `.averageQuantity()`, `.minimumQuantity()`, `.maximumQuantity()`

- `HKStatisticsCollectionQuery` — statistics grouped over fixed intervals
  - `init(quantityType:quantitySamplePredicate:options:anchorDate:intervalComponents:)` — construct query
  - `.initialResultsHandler` — called once with initial `HKStatisticsCollection`
  - `.statisticsUpdateHandler` — called on each future update with new statistics
  - `HKStatisticsCollection.enumerateStatistics(from:to:with:)` — iterate over time-bucketed statistics
  - `HKStatisticsCollection.statistics(for:)` — retrieve `HKStatistics` for a specific date

- `HKSampleQuery` — raw sample retrieval
  - `init(sampleType:predicate:limit:sortDescriptors:resultsHandler:)` — construct query
  - `HKObjectQueryNoLimit` — constant for unlimited results
  - Sort by `HKSampleSortIdentifierEndDate` or `HKSampleSortIdentifierStartDate`

- `HKQuery.predicateForSamples(withStart:end:options:)` — class method for common date-range predicate

## Code Highlights

Setup and authorization:
```swift
import HealthKit

let healthStore = HKHealthStore()

func setUpHealthKit() {
    guard HKHealthStore.isHealthDataAvailable() else { return }
    let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount)!
    healthStore.requestAuthorization(toShare: [stepType], read: [stepType]) { success, error in
        if success { self.calculateDailyStepCount() }
    }
}
```

Writing a quantity sample:
```swift
let distanceType = HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning)!
let quantity = HKQuantity(unit: .meter(), doubleValue: 628)
let sample = HKQuantitySample(type: distanceType, quantity: quantity,
                              start: startDate, end: endDate)
healthStore.save(sample) { success, error in
    // handle result
}
```

HKStatisticsCollectionQuery with live updates:
```swift
let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount)!
let oneWeekAgo = Calendar.current.date(byAdding: .day, value: -7, to: Date())!
let predicate = HKQuery.predicateForSamples(withStart: oneWeekAgo, end: Date())
let anchor = Calendar.current.startOfDay(for: oneWeekAgo)
var components = DateComponents(); components.day = 1

let query = HKStatisticsCollectionQuery(
    quantityType: stepType,
    quantitySamplePredicate: predicate,
    options: .cumulativeSum,
    anchorDate: anchor,
    intervalComponents: components
)
query.initialResultsHandler = { _, collection, _ in
    collection?.enumerateStatistics(from: oneWeekAgo, to: Date()) { stats, _ in
        let steps = stats.sumQuantity()?.doubleValue(for: .count()) ?? 0
        DispatchQueue.main.async { /* update UI */ }
    }
}
query.statisticsUpdateHandler = { _, _, collection, _ in
    collection?.enumerateStatistics(from: oneWeekAgo, to: Date()) { stats, _ in
        DispatchQueue.main.async { /* update UI */ }
    }
}
healthStore.execute(query)
// Stop in viewWillDisappear:
healthStore.stop(query)
```

## Takeaways
- Always call `HKHealthStore.isHealthDataAvailable()` before proceeding — HealthKit is not supported on all Apple platforms.
- Request authorization every time you intend to access health data; request only what is needed for the current feature; always request in context so users understand why.
- Use `HKStatisticsCollectionQuery` with an update handler for any time-series charts — it replaces dozens of individual `HKStatisticsQuery` calls and automatically keeps the UI current as new data arrives.
- Dispatch query results back to the main thread before updating UI; stop long-running queries in `viewWillDisappear` to avoid background resource usage.

---
_Source: WWDC20 Session 10664 page (abstract, transcript, and resource links)._
