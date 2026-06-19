# Exploring New Data Representations in HealthKit
**WWDC19 · Session 218** · [Watch](https://developer.apple.com/videos/play/wwdc2019/218/)

_Platforms:_ iOS 13, watchOS 6

## Overview
iOS 13 significantly expands HealthKit's data model with new efficient storage mechanisms for high-frequency quantity data, beat-to-beat heart rate capture, new heart alert category types, and an entirely new hearing health domain. The session covers the new quantity series architecture, the heartbeat series sample type, and APIs for audiogram and audio exposure data.

The quantity series approach solves a longstanding problem: storing many readings from a single measurement session (like a heart rate sensor during a game) previously required either a single averaged sample (losing resolution) or many redundant individual samples (wasting storage). Series samples provide both a single summarizing object and full per-reading resolution. A parallel design for heartbeat timestamps gives apps access to beat-to-beat intervals without storing HK quantities at all.

Hearing health is an entirely new health domain in HealthKit, supporting audiogram test results, headphone and environmental audio exposure, and audio exposure alerts from Apple Watch.

## Key Topics

**Quantity Series**
- New `HKQuantitySeriesSampleBuilder` writes high-frequency readings incrementally and finalizes into a single `HKQuantitySample` backed by all inserted values
- `HKQuantitySeriesSampleQuery` updated: now accepts a quantity type + NSPredicate instead of a single sample, returning all backing quantities with full date intervals and optional parent sample reference
- `HKStatisticsCollectionQuery` updated to support all new aggregation styles and automatically include backing quantity data from series

**Aggregation Style Updates**
- `HKQuantityAggregationStyle.discrete` deprecated
- New `.discreteArithmetic` (simple mean), `.discreteTemporallyWeighted` (time-weighted, for heart rate), `.discreteEquivalentContinuousLevel` (for audio exposure)
- New concrete subclasses: `HKCumulativeQuantitySample` (with `sum`) and `HKDiscreteQuantitySample` (with `average`, `minimum`, `maximum`, `mostRecentQuantity`)
- `HKQuantitySample` is now abstract; all instances are one of the two subclasses

**Heartbeat Series (Beat-to-Beat)**
- `HKHeartbeatSeriesSample` — stores timestamps of individual heartbeats (not quantities)
- `HKHeartbeatSeriesBuilder` — incrementally adds beats with `precededByGap` flag for data gaps
- `HKHeartbeatSeriesQuery` — enumerates beat timestamps for a given sample

**Heart Alert Category Types (New)**
- Three new `HKCategoryTypeIdentifier` values saved by Apple Watch when alerts fire: `.highHeartRate`, `.lowHeartRate`, `.irregularHeartRhythm`

**Hearing Health (New Domain)**
- `HKAudiogramSample` — stores an array of `HKAudiogramSensitivityPoint` values representing a pure-tone audiogram
- `HKAudiogramSensitivityPoint` — frequency + left/right ear sensitivity quantities in dBHL
- New `HKUnit`: `.hertz()`, `.decibelHearingLevel()`
- `HKQuantityTypeIdentifier.headphoneAudioExposure` **[NEW]** — read-write, tracks headphone audio exposure
- `HKQuantityTypeIdentifier.environmentalAudioExposure` **[NEW]** — read-write, tracked by Apple Watch
- `HKCategoryTypeIdentifier.audioExposureEvent` **[NEW]** — saved when environmental audio exposure exceeds threshold

## APIs & Frameworks

**HealthKit**

_Quantity Series_
- `HKQuantitySeriesSampleBuilder` **[NEW]** — `init(healthStore:quantityType:startDate:device:)`; `func insert(_:at:) `, `func finishSeries(metadata:endDate:completionHandler:)`
- `HKCumulativeQuantitySample` **[NEW]** — `var sum: HKQuantity`
- `HKDiscreteQuantitySample` **[NEW]** — `var average: HKQuantity`, `var minimum: HKQuantity`, `var maximum: HKQuantity`, `var mostRecentQuantity: HKQuantity`, `var mostRecentQuantityDateInterval: DateInterval`
- `HKQuantityAggregationStyle.discreteArithmetic` **[NEW]**
- `HKQuantityAggregationStyle.discreteTemporallyWeighted` **[NEW]**
- `HKQuantityAggregationStyle.discreteEquivalentContinuousLevel` **[NEW]**
- `HKQuantityAggregationStyle.discrete` — deprecated
- `HKQuantitySeriesSampleQuery` — updated: `init(quantityType:predicate:quantityHandler:)`; handler now receives `DateInterval` and optional `HKQuantitySample`
- `HKStatisticsCollectionQuery` — updated for new aggregation styles and series backing data

_Heartbeat Series_
- `HKHeartbeatSeriesSample` **[NEW]** — top-level sample for beat-to-beat data
- `HKHeartbeatSeriesBuilder` **[NEW]** — `init(healthStore:device:startDate:)`; `func addHeartbeat(atTimeInterval:precededByGap:completion:)`; `func finishSeries(metadata:completionHandler:)`
- `HKHeartbeatSeriesQuery` **[NEW]** — `init(heartbeatSeries:dataHandler:)`; enumerates `(timeInterval, precededByGap, done)`
- `HKQuantityTypeIdentifier.heartRateVariabilitySDNN` (existing, iOS 11)

_Heart Alerts_
- `HKCategoryTypeIdentifier.highHeartRate` **[NEW]**
- `HKCategoryTypeIdentifier.lowHeartRate` **[NEW]**
- `HKCategoryTypeIdentifier.irregularHeartRhythm` **[NEW]**

_Hearing Health_
- `HKAudiogramSample` **[NEW]** — `init(sensitivityPoints:startDate:endDate:metadata:)`
- `HKAudiogramSensitivityPoint` **[NEW]** — `init(frequency:leftEarSensitivity:rightEarSensitivity:)`
- `HKUnit.hertz()` **[NEW]**
- `HKUnit.decibelHearingLevel()` **[NEW]**
- `HKQuantityTypeIdentifier.headphoneAudioExposure` **[NEW]**
- `HKQuantityTypeIdentifier.environmentalAudioExposure` **[NEW]**
- `HKCategoryTypeIdentifier.audioExposureEvent` **[NEW]**

_Predicate support_
- New predicate keypaths for `HKCumulativeQuantitySample.sum` and `HKDiscreteQuantitySample` statistics properties

## Code Highlights

Creating a quantity series:
```swift
let builder = HKQuantitySeriesSampleBuilder(healthStore: healthStore,
                                             quantityType: heartRateType,
                                             startDate: Date(),
                                             device: device)
// For each sensor reading:
builder.insert(quantity, at: dateInterval)
// At end:
builder.finishSeries(metadata: metadata, endDate: endDate) { sample, error in ... }
```

Creating an audiogram sample:
```swift
let freq = HKQuantity(unit: .hertz(), doubleValue: 125)
let leftSens = HKQuantity(unit: .decibelHearingLevel(), doubleValue: 11)
let rightSens = HKQuantity(unit: .decibelHearingLevel(), doubleValue: 31)
let point = try HKAudiogramSensitivityPoint(frequency: freq,
                                            leftEarSensitivity: leftSens,
                                            rightEarSensitivity: rightSens)
let audiogram = HKAudiogramSample(sensitivityPoints: [point],
                                  startDate: start, endDate: end, metadata: nil)
```

## Takeaways
- Use `HKQuantitySeriesSampleBuilder` for any high-frequency sensor data to preserve resolution without redundant storage overhead.
- `HKQuantitySample` is now abstract; check for `HKCumulativeQuantitySample` or `HKDiscreteQuantitySample` when reading samples.
- Heartbeat series provide true beat-to-beat data separate from computed HRV — request both `heartbeatSeries` and `heartRateVariabilitySDNN` authorizations together.
- Hearing health is a new opportunity: audiogram apps and audio exposure tracking are now first-class HealthKit citizens in iOS 13.

---
_Source: WWDC19 Session 218 page (abstract, chapter summaries, code samples, and resource links)._
