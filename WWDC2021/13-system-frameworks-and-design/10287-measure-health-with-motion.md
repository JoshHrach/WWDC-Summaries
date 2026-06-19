# Measure Health with Motion
**WWDC21 · Session 10287** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10287/)

_Platforms:_ iOS 15, watchOS 8

## Overview
This session introduces two new health metrics that leverage iPhone and Apple Watch motion sensing: Walking Steadiness (new in iOS 15) and an updated Six-Minute Walk distance metric with a new recalibration API (iOS 15/watchOS 8). Together, these metrics provide continuous, passive, objective measures of walking quality and endurance without requiring dedicated equipment or clinical assessments.

Walking Steadiness captures the quality of an individual's gait and can predict fall risk over the next 12 months. The Six-Minute Walk distance metric estimates walking endurance and, with the new recalibration API, can now accurately capture acute changes in health following events like surgery. Both metrics integrate with HealthKit and are available in the Health app.

The session is presented from the perspective of building a virtual physical therapy clinic app, demonstrating how health and motion data can be used to monitor patient recovery remotely, flag at-risk patients, and track trends over time.

## Key Topics

**Six-Minute Walk Distance Recalibration**
The existing Six-Minute Walk metric uses a historical measurement window to build an accurate estimate. A new `recalibrateEstimates` API lets apps mark a recalibration date (e.g., a surgery date) so subsequent estimates use only post-event data, accurately capturing reduced mobility following an acute event. The recalibration effect is temporary and the metric reverts to normal windowing once the recalibration date is sufficiently in the past.

**Apple Walking Steadiness**
A new metric written weekly to HealthKit representing the quality of a user's gait. The score ranges from 0 to 1 (expressed as a percent) and maps to three classifications: OK, Low, and Very Low. Low and Very Low classifications trigger built-in Health app notifications, which apps can observe via HealthKit to prompt follow-up care.

**Custom Trend Detection**
Apps can query six weeks of steadiness scores and calculate a best-fit slope to detect declining steadiness in patients who have not yet crossed into a higher-risk classification, enabling proactive outreach.

**Best Practices for Health Data Collection**
The session covers privacy requirements (encryption, transparency, user control), data quality prerequisites (height/weight/age set in Health app, consistent phone carry behavior), and how to prompt users to enable Walking Steadiness notifications.

## APIs & Frameworks

### HealthKit

**Six-Minute Walk Distance**
- `HKQuantityTypeIdentifier.sixMinuteWalkTestDistance` — existing quantity type for walking endurance
- `HKHealthStore.recalibrateEstimates(sampleType:date:completionHandler:)` — recalibrates estimates from a given date forward **[NEW]**
- `HKSampleType.allowsRecalibrationForEstimates` — property to check if a type supports recalibration **[NEW]**
- `HKMetadataKeyEarliestDateUsedForEstimate` / "EarliestDateUsedForEstimate" metadata key — indicates start of measurement window for an estimate **[NEW]**

**Walking Steadiness**
- `HKQuantityTypeIdentifier.appleWalkingSteadiness` — new read-only quantity type, written weekly **[NEW]**
  - Unit: `HKUnit.percentUnit()`, range 0–1
- `HKAppleWalkingSteadinessClassification` — new enum for fall-risk classification **[NEW]**
  - `.ok` — healthy gait, low fall risk
  - `.low` — compromised mobility, elevated fall risk
  - `.veryLow` — significantly compromised mobility, high fall risk
  - `HKAppleWalkingSteadinessClassification(for:)` — convenience initializer converting a quantity to classification **[NEW]**
- `HKCategoryTypeIdentifier.appleWalkingSteadinessEvent` — new read-only category type for steadiness notifications **[NEW]**
  - `.initialLow` — first notification when user enters Low classification
  - `.initialVeryLow` — first notification when user enters Very Low classification
  - `.repeatLow` — repeat notification after ~3 months in Low
  - `.repeatVeryLow` — repeat notification after ~3 months in Very Low

**HealthKit Query Types Used**
- `HKSampleQuery` — used to retrieve most recent steadiness score and batch of historical scores
- `HKObserverQuery` — used to receive background notifications when steadiness events are saved
- `HKHealthStore.requestAuthorization(toShare:read:)` — authorization for reading steadiness types
- `HKHealthStore.enableBackgroundDelivery(for:frequency:withCompletion:)` — implied for background steadiness event delivery

### Core Motion
- `CMFallDetectionManager` — referenced in resources for fall detection (not demoed in session)

## Code Highlights

Recalibrate Six-Minute Walk estimates after surgery:
```swift
let sixMinuteWalkType = HKSampleType.quantityType(forIdentifier: .sixMinuteWalkTestDistance)!
if sixMinuteWalkType.allowsRecalibrationForEstimates {
    healthStore.recalibrateEstimates(sampleType: sixMinuteWalkType, date: surgeryDate) { success, error in
        // handle error
    }
}
```

Convert steadiness score to classification:
```swift
let recentClassification = HKAppleWalkingSteadinessClassification(for: walkingSteadiness.quantity)
```

Observe steadiness notification events in the background:
```swift
let notificationType = HKCategoryType.categoryType(forIdentifier: .appleWalkingSteadinessEvent)!
let query = HKObserverQuery(sampleType: notificationType, predicate: nil) { query, completionHandler, error in
    promptCheckupForNotification()
    completionHandler()
}
healthStore.execute(query)
```

## Takeaways
- `HKAppleWalkingSteadiness` and its classification enum give apps a passive, device-level fall-risk metric without requiring any specialized hardware or in-clinic testing.
- The new `recalibrateEstimates` API on `HKHealthStore` significantly improves the Six-Minute Walk metric's usefulness for tracking post-surgical recovery by anchoring the estimate window to a known event date.
- Apps should combine score queries, classification, and event observation to build a tiered monitoring strategy: classify current risk, observe system notifications, and detect declining trends before classification thresholds are crossed.
- All health data handling should follow strong privacy practices: encrypt data taken outside HealthKit, be transparent about collection, and ensure users can view and delete their data.

---
_Source: WWDC21 Session 10287 page (abstract, chapter summaries, code samples, and resource links)._
