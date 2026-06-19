# Synchronize Health Data with HealthKit
**WWDC20 · Session 10184** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10184/)

_Platforms:_ iOS 14, watchOS 7

## Overview
Health data flows across a user's entire device ecosystem—iPhone, Apple Watch, third-party apps, and external care-team servers. This session explains how to correctly monitor changes in the HealthKit database and how to keep an external data store (local or remote) in sync without introducing duplicates, spurious deletes, or stale data.

The key insight is that different data types demand different query strategies. High-frequency, cumulative data (step count) is best summarized with `HKStatisticsCollectionQuery` driven by an `HKAnchoredObjectQuery` that fires only when the data actually changes. Low-frequency, individually significant data (six-minute walk test results) should track each sample, making the `HKAnchoredObjectQuery` the primary tool for change detection without the statistics layer.

For external synchronization, HealthKit's built-in `HKMetadataKeySyncIdentifier` and `HKMetadataKeySyncVersion` metadata keys give every sample a stable identity across devices and versions. HealthKit enforces deduplication and version-ordering automatically: re-saving a sample with the same identifier but no version increment is silently ignored, while a higher version overwrites the previous sample atomically.

## Key Topics
- **Monitoring changes with `HKAnchoredObjectQuery`** — snapshot of new/deleted samples since a persisted anchor; replaces polling the full dataset
- **Combining queries** — use `HKAnchoredObjectQuery` to detect which dates changed, then drive `HKStatisticsCollectionQuery` only for those date ranges
- **Persisting the anchor** — store the anchor between app launches so subsequent queries return only incremental changes
- **Choosing the right query per data type** — cumulative/high-frequency data → statistics; low-frequency individual samples → anchored object query
- **Sync identifiers and versions** — `HKMetadataKeySyncIdentifier` / `HKMetadataKeySyncVersion` for conflict-free cross-device deduplication
- **Safe saves** — HealthKit silently ignores duplicate sync-identifier saves; higher version overwrites atomically
- **Safe deletes** — only delete samples your app saved; query first, then delete; prefer soft-delete / user-intent checks
- **New mobility data types** — `sixMinuteWalkTestDistance` and related types introduced in iOS 14 / watchOS 7

## APIs & Frameworks

**HealthKit**
- `HKHealthStore` — primary entry point; `execute(_:)`, `save(_:withCompletion:)`, `delete(_:withCompletion:)`
- `HKAnchoredObjectQuery` — **primary change-monitoring query**; returns added and deleted samples since anchor
  - `init(type:predicate:anchor:limit:resultsHandler:)` — initializer
  - `resultsHandler` / `updateHandler` — same block pattern for initial and live updates
  - `HKQueryAnchor` — opaque anchor persisted between launches **[persist and reuse]**
- `HKStatisticsCollectionQuery` — aggregated statistics over time intervals (anchor date, interval, options)
  - `options: .cumulativeSum` — sum over each interval bucket
- `HKQuantityType` — typed quantity sample type (e.g., `HKQuantityType.quantityType(forIdentifier: .stepCount)`)
- `HKQuantitySample` — individual health sample
  - `init(type:quantity:startDate:endDate:metadata:)` — includes metadata for sync
- `HKQuantity` — value + unit pair (`HKUnit.meter()`, `HKUnit.count()`, etc.)
- `HKObjectType.quantityType(forIdentifier:)` — retrieves quantity type by identifier
- `HKQuantityTypeIdentifier.stepCount` — step count identifier
- `HKQuantityTypeIdentifier.sixMinuteWalkTestDistance` **[NEW in iOS 14]** — distance covered in a six-minute walk test
- `HKMetadataKeySyncIdentifier` **[key]** — String metadata key; unique ID for a sample across devices
- `HKMetadataKeySyncVersion` **[key]** — Int metadata key; version number; higher version overwrites same-identifier sample
- `HKSampleQuery` — general-purpose query for fetching samples (used as a prerequisite before delete)
- `HKPredicate` — query predicate builder (date range, sample type, etc.)

**CareKit** (referenced)
- Open-source framework for building care-experience apps; recommended for rich health graph visualizations

## Code Highlights

Using `HKAnchoredObjectQuery` to detect step-count changes and then run statistics only for affected dates:
```swift
var persistedAnchor: HKQueryAnchor? = loadAnchor()

let handler: HKAnchoredObjectQueryUpdateHandler = { query, samples, deletedObjects, newAnchor, error in
    guard let samples = samples as? [HKQuantitySample] else { return }
    persistedAnchor = newAnchor
    saveAnchor(newAnchor)

    let predicate = predicateFromSamples(samples) // date range predicate
    fetchStatistics(with: predicate)              // run HKStatisticsCollectionQuery for those dates
}

let query = HKAnchoredObjectQuery(
    type: HKQuantityType.quantityType(forIdentifier: .stepCount)!,
    predicate: nil,
    anchor: persistedAnchor,
    limit: HKObjectQueryNoLimit,
    resultsHandler: handler)
query.updateHandler = handler
healthStore.execute(query)
```

Saving a six-minute walk test sample with sync metadata to prevent duplicates:
```swift
let metadata: [String: Any] = [
    HKMetadataKeySyncIdentifier: serverSample.syncIdentifier,
    HKMetadataKeySyncVersion: serverSample.syncVersion
]
let sample = HKQuantitySample(
    type: HKQuantityType.quantityType(forIdentifier: .sixMinuteWalkTestDistance)!,
    quantity: HKQuantity(unit: .meter(), doubleValue: serverSample.distanceMeters),
    start: serverSample.startDate,
    end: serverSample.endDate,
    metadata: metadata)
healthStore.save(sample) { success, error in ... }
```

## Takeaways
- Always monitor HealthKit changes with `HKAnchoredObjectQuery` rather than periodic full-dataset polls; persist the anchor across launches to retrieve only incremental changes.
- Combine `HKAnchoredObjectQuery` (change detection) with `HKStatisticsCollectionQuery` (aggregation) for cumulative data types like step count; query individual samples directly for low-frequency types like six-minute walk test.
- Use `HKMetadataKeySyncIdentifier` and `HKMetadataKeySyncVersion` on every sample written from an external source; HealthKit will deduplicate across all user devices and handle version ordering atomically.
- Only delete samples your app created, always reflecting genuine user intent—prefer querying first, then deleting the specific matched sample.

---
_Source: WWDC20 Session 10184 page (abstract, chapter summaries, code samples, and resource links)._
