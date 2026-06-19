# Beyond Counting Steps
**WWDC20 · Session 10656** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10656/)

_Platforms:_ iOS 14, watchOS 7

## Overview
Apple introduces eight new Mobility Metrics in HealthKit for iOS 14 and watchOS 7 that go far beyond step counting to measure the quality, efficiency, and capacity of human movement — all captured passively as users go about their daily lives with iPhone or Apple Watch. Four metrics are iPhone-derived (walking speed, step length, double support time, walking asymmetry); four are Apple Watch-derived (stair ascent/descent speed, six-minute walk distance estimate, improved VO2Max).

The session explains the clinical significance of each metric, demonstrates how to query, aggregate, and visualize them using HealthKit and ResearchKit, and shows how to distinguish auto-predicted values from user-entered values in HealthKit samples. The running example — tracking ankle-injury recovery — illustrates a three-step developer pattern: query a HealthKit quantity type, aggregate using a domain-appropriate statistical method (e.g., 95th-percentile per day, seven-day rolling average), then combine multiple metrics to reveal the full picture.

## Key Topics
- **iPhone Mobility Metrics (new)** — Walking Speed, Step Length, Double Support Time (% time with both feet on ground; lower = better), Walking Asymmetry (% time one foot's steps are faster than the other's).
- **Apple Watch Mobility Metrics (new)** — Stair Ascent Speed, Stair Descent Speed, Six-Minute Walk Distance (estimated from a week of passive movement data), VO2Max (now usable at slow walking speeds, not just running).
- **Data collection model** — iPhone metrics capture continuous flat-ground walking bouts (not all walking); collected on the motion co-processor all day; no GPS calibration needed; retroactive — data exists before the app is installed.
- **HealthKit query pattern** — Request read permission with `HKHealthStore.requestAuthorization(toShare:read:)`; construct `HKSampleQuery` with a date predicate and sort descriptor; execute to receive `[HKQuantitySample]` in a completion handler.
- **Aggregation strategies** — Raw data contains many samples per day; filter by relevant walking bouts, then aggregate (e.g., 95th-percentile of daily speeds to estimate maximum walking capacity; seven-day rolling average to smooth noise; reset aggregation at injury date).
- **Source filtering** — Distinguish Apple-predicted samples from user-entered Health app values via `HKSample.sourceRevision.source.bundleIdentifier` and `HKSample.metadata[HKMetadataKeyWasUserEntered]`.
- **ResearchKit visualization** — `ORKLineGraphChartView` and `ORKDiscreteGraphChartView` accept `ORKValueRange` data points via a chart data source; useful for rendering per-day walking metrics.
- **Headphone Motion API (new)** — AirPods Pro now expose inertial sensor signals via the headphone motion API, adding a third wearable motion source alongside iPhone and Apple Watch.
- **CMPedometer / CMAltimeter** — Existing APIs for step count, cadence, distance, pace, flights of stairs, and altitude change complement the new HealthKit metrics.

## APIs & Frameworks

### HealthKit
- **`HKQuantityType.quantityType(forIdentifier:)`** — Retrieve a quantity type by identifier
- **`HKQuantityTypeIdentifier.walkingSpeed`** **[NEW]** — `HKQuantityTypeIdentifier`; meters per second or miles per hour
- **`HKQuantityTypeIdentifier.walkingStepLength`** **[NEW]** — `HKQuantityTypeIdentifier`; meters
- **`HKQuantityTypeIdentifier.walkingDoubleSupportPercentage`** **[NEW]** — `HKQuantityTypeIdentifier`; percent
- **`HKQuantityTypeIdentifier.walkingAsymmetryPercentage`** **[NEW]** — `HKQuantityTypeIdentifier`; percent
- **`HKQuantityTypeIdentifier.stairAscentSpeed`** **[NEW]** — `HKQuantityTypeIdentifier`; floors per second (Apple Watch)
- **`HKQuantityTypeIdentifier.stairDescentSpeed`** **[NEW]** — `HKQuantityTypeIdentifier`; floors per second (Apple Watch)
- **`HKQuantityTypeIdentifier.sixMinuteWalkTestDistance`** **[NEW]** — `HKQuantityTypeIdentifier`; meters (Apple Watch prediction or user-entered)
- **`HKQuantityTypeIdentifier.vo2Max`** — Existing; now supported at slower walking-level effort; not new but improved
- **`HKSampleQuery`** — `init(sampleType:predicate:limit:sortDescriptors:resultsHandler:)` — query walking samples over a date range
- **`HKQuery.predicateForSamples(withStart:end:options:)`** — Date range predicate for queries
- **`HKSample.sourceRevision`** — `HKSourceRevision`; `.source.bundleIdentifier` to detect Apple-sourced samples
- **`HKSample.metadata`** — `[String: Any]?`; check `HKMetadataKeyWasUserEntered` to distinguish user-entered from predicted values
- **`HKQuantitySample.quantity`** — `HKQuantity`; call `.doubleValue(for: HKUnit)` to extract numeric value in a chosen unit

### Core Motion
- **`CMPedometer`** — Existing; step count, cadence, distance, pace, floors
- **`CMAltimeter`** — Existing; altitude change
- **`CMHeadphoneMotionManager`** **[NEW]** — Inertial sensor data from AirPods Pro

### ResearchKit
- **`ORKDiscreteGraphChartView`** — Existing; renders per-day range bars; used for plotting daily walking speed spread
- **`ORKLineGraphChartView`** — Existing; line graph; used for aggregated trend lines

## Code Highlights

Request HealthKit authorization and query walking speed:
```swift
let walkingSpeedType = HKQuantityType.quantityType(forIdentifier: .walkingSpeed)!
healthStore.requestAuthorization(toShare: nil, read: [walkingSpeedType]) { _, _ in }

let predicate = HKQuery.predicateForSamples(
    withStart: thirtyDaysBeforeInjury, end: sixtyDaysAfterInjury, options: .strictStartDate)
let query = HKSampleQuery(
    sampleType: walkingSpeedType,
    predicate: predicate,
    limit: HKObjectQueryNoLimit,
    sortDescriptors: [NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: true)]
) { _, samples, _ in
    guard let samples = samples as? [HKQuantitySample] else { return }
    // process samples
}
healthStore.execute(query)
```

Compute 95th-percentile daily walking speed:
```swift
let speeds = daySamples.map { $0.quantity.doubleValue(for: .milePerHour()) }.sorted()
let p95Index = Int(Double(speeds.count) * 0.95)
let maxSpeed = speeds[min(p95Index, speeds.count - 1)]
```

Filter Apple Watch–predicted six-minute walk vs. user-entered:
```swift
let isApple = sample.sourceRevision.source.bundleIdentifier.hasPrefix("com.apple")
let isUserEntered = (sample.metadata?[HKMetadataKeyWasUserEntered] as? Bool) == true
```

## Takeaways
- The eight new Mobility Metrics in HealthKit provide clinically meaningful, passively collected walking data — accessible retroactively without app installation — that goes far beyond step counting.
- Query `HKQuantityTypeIdentifier.walkingSpeed`, `.walkingDoubleSupportPercentage`, `.sixMinuteWalkTestDistance`, and related identifiers just like any other HealthKit quantity type; the framework handles sensor fusion and bout detection automatically.
- Raw mobility data is noisy; aggregate with domain-appropriate statistics (95th percentile for maximum capacity, seven-day rolling average for trend) and reset at meaningful events (e.g., injury date) to reveal clinically meaningful changes.
- Combine multiple metrics to create a holistic picture — walking speed reveals magnitude of change, double support time reveals compensatory balance strategies, and six-minute walk distance provides a standardized clinical benchmark.
- Use `sourceRevision.source.bundleIdentifier` and `HKMetadataKeyWasUserEntered` to layer both Apple-predicted and clinician-measured data on the same chart for a complete recovery timeline.

---
_Source: WWDC20 Session 10656 page (abstract and transcript)._
