# Discover New Capabilities in the App Intents Framework
**WWDC26 · Session 345** · [Watch](https://developer.apple.com/videos/play/wwdc2026/345/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, visionOS

## Overview
This session introduces a broad set of new App Intents capabilities organized into three areas: entity enhancements, richer parameter types, and intent execution control. All examples are built on the Landmarks Travel Tracking sample app and illustrate how the new APIs make intents faster, more flexible, and more relevant across Siri, Shortcuts, Spotlight, Widgets, and Apple Intelligence.

Entity enhancements include `ValueRepresentation` for cross-app structured data sharing, `RelevantEntities` for proactive content surfacing, `EntityCollection` for large-scale parameter passing without full resolution, and `SyncableEntity` for stable cross-device identifiers in multi-device Siri conversations. Parameter type improvements extend native support to `Duration`, `PersonNameComponents`, and the new `@UnionValue` macro. On the execution side, `LongRunningIntent` breaks the 30-second ceiling and `ExecutionTargets` gives developers precise control over which process runs an intent.

## Key Topics

### Share Entities Across Apps with ValueRepresentation
The new `ValueRepresentation` transfer representation carries structured system types (like `PlaceDescriptor` from GeoToolbox) to other apps. This extends `Transferable` beyond file and data blobs — a landmark entity can flow directly into Maps for directions. Use a closure to transform, or a key-path if the entity already has the property.

### Register Relevant Entities with RelevantEntities
Spotlight indexing and donations can't surface content users have never touched. `RelevantEntities.shared.updateEntities(_:for:)` proactively registers entities with a context (e.g., running playlists when a workout starts). Entities can be removed by context, by specific entity set, or all at once.

### Handle Entities Efficiently with EntityCollection
Previously, resolving every entity in a large array before `perform()` ran was costly. The new `EntityCollection<T>` parameter type passes only identifiers to the intent body — a one-line parameter type change that made tagging 1,000 photos nearly instant. Use `.identifiers` in `perform()` instead of iterating full entities.

### Use Entities Across Devices with SyncableEntity
Siri conversations now continue across devices, but local IDs differ per device. `SyncableEntity` declares a stable ID (server UUID, CloudKit record ID). When you only have local IDs, `SyncableEntityIdentifier` pairs a local ID (used on-device) with a stable ID (used by the system).

### Richer Parameter Types
`@Parameter` now supports `Duration` (no custom time pickers needed) and `PersonNameComponents`, with full Siri, Shortcuts, and Widget support. The system handles picker presentation, disambiguation, and localization automatically.

### Union Value Parameters
`@UnionValue` enum lets a single parameter accept multiple entity types (e.g., a widget that shows either a `LandmarkCollectionEntity` or a `PhotoAlbumEntity`). The macro generates `typeDisplayRepresentation`, per-case display representations, and picker support — works in Shortcuts and Widgets.

### Extend Execution with LongRunningIntent
Intents normally have a 30-second budget. `LongRunningIntent` removes that cap, manages the background task lifecycle, shows progress as a Live Activity (built on `ProgressReportingIntent`), and supports background GPU access. Wrap work in `performBackgroundTask(_:onCancel:)` and report progress via `Progress`. Add `CancellableIntent` to clean up gracefully on cancellation.

### Target the Right Process with ExecutionTargets
When intents live in a shared Swift package, the system picks a host process by heuristics — not always the right one. `ExecutionTargets` overrides this via `allowedExecutionTargets`: `.main`, `.appIntentsExtension`, `.widgetKitExtension`, or any combination.

## APIs & Frameworks

### AppIntents
- `ValueRepresentation` — **[NEW]** `TransferRepresentation` that shares structured system types across apps
- `PlaceDescriptor` — system structured type (GeoToolbox); exportable via `ValueRepresentation`
- `IntentValueRepresentation` — export/import structured intent values via `Transferable`
- `RelevantEntities` — **[NEW]** `shared.updateEntities(_:for:)`, `removeAllEntities(for:)`, `removeEntities(_:from:)`, `removeAllEntities()` methods
- `AppEntityContext` — **[NEW]** context type for relevant entity registration; e.g., `.audio(.workout(activityType:))`
- `EntityCollection<T>` — **[NEW]** lazy parameter type; use `.identifiers` instead of full resolution
- `SyncableEntity` protocol — **[NEW]** declares stable cross-device entity IDs
- `SyncableEntityIdentifier<Local, Stable>` — **[NEW]** pairs a local ID with a stable ID
- `@UnionValue` macro — **[NEW]** generates union enum for multi-type parameters; `typeDisplayRepresentation`, `caseDisplayRepresentations`
- `LongRunningIntent` protocol — **[NEW]** removes 30-second execution cap; `performBackgroundTask(_:onCancel:)` method
- `CancellableIntent` protocol — **[NEW]** `onCancel(reason:)` cleanup callback
- `ProgressReportingIntent` protocol — `progress` property for Live Activity updates
- `ExecutionTargets` — **[NEW]** `allowedExecutionTargets` static property; `.main`, `.appIntentsExtension`, `.widgetKitExtension`
- `@Parameter` — extended to support `Duration`, `PersonNameComponents` natively **[NEW]**
- `Duration` — **[NEW]** native parameter type (no custom pickers needed)
- `PersonNameComponents` — **[NEW]** native parameter type
- `AppEntity` protocol — `id`, `displayRepresentation`, `defaultQuery`
- `Transferable` protocol — used alongside `AppEntity` for cross-app sharing
- `TransferRepresentation` — base protocol; `ValueRepresentation` is a new conformance
- `OpenIntent` protocol — `target` parameter
- `IntentResult` — `.result()` return
- `@Dependency` property wrapper

### GeoToolbox (new framework referenced)
- `PlaceDescriptor` — structured location type; `.coordinate(CLLocationCoordinate2D)` representation **[NEW]**

### Vision
- `GenerateImageFeaturePrintRequest` — used in Landmarks sample for feature print computation
- `FeaturePrintObservation` — `.distance(to:)` method for similarity

## Code Highlights

**ValueRepresentation — export a landmark as a PlaceDescriptor:**
```swift
struct LandmarkEntity: AppEntity, Transferable {
    static var transferRepresentation: some TransferRepresentation {
        ValueRepresentation(exporting: { entity in
            PlaceDescriptor(
                representations: [.coordinate(entity.landmark.locationCoordinate)],
                commonName: entity.landmark.name
            )
        })
    }
}
```

**EntityCollection — pass identifiers, skip full resolution:**
```swift
struct TagPhotosIntent: AppIntent {
    @Parameter var photos: EntityCollection<PhotoEntity>  // was: [PhotoEntity]
    func perform() async throws -> some IntentResult {
        modelData.tagPhotos(ids: photos.identifiers, tag: tag)
        return .result()
    }
}
```

**SyncableEntity with local + stable IDs:**
```swift
struct PhotoEntity: AppEntity, SyncableEntity {
    var id: SyncableEntityIdentifier<String, String>
    init(localID: String, stableID: String) {
        self.id = SyncableEntityIdentifier(local: localID, stable: stableID)
    }
}
```

**LongRunningIntent with cancellation:**
```swift
struct UploadPhotoIntent: LongRunningIntent, CancellableIntent {
    func perform() async throws -> some IntentResult & ProvidesDialog {
        let result = try await performBackgroundTask({
            for chunk in 1...chunks {
                try Task.checkCancellation()
                try await uploadChunk(chunk)
                progress.completedUnitCount = Int64(chunk)
            }
            return "Upload complete!"
        }, onCancel: { reason in cleanup(for: reason) })
        return .result(dialog: "\(result)")
    }
}
```

**ExecutionTargets — force main app for write operations:**
```swift
struct UpdateFavoriteIntent: AppIntent {
    static var allowedExecutionTargets: ExecutionTargets { .main }
}
```

## Takeaways
- `EntityCollection` is a one-line change that dramatically improves performance for intents operating on large sets — use it whenever you pass many entities as a parameter.
- `SyncableEntity` is required once your app's Siri conversations span devices; pair local and stable IDs with `SyncableEntityIdentifier` if you don't have server-side UUIDs.
- `LongRunningIntent` + `CancellableIntent` + `ProgressReportingIntent` together form the complete pattern for background work that needs to outlive the standard intent budget and report progress to the user.
- `@UnionValue` and the new native `Duration` / `PersonNameComponents` parameter types significantly reduce boilerplate for common intent parameter patterns.

---
_Source: WWDC26 Session 345 page (abstract, chapter summaries, code samples, and resource links)._
