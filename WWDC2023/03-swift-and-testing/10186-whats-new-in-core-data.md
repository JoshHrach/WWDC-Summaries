# What's New in Core Data
**WWDC23 · Session 10186** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10186/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10

## Overview
This session introduces three major additions to Core Data in iOS 17 / macOS Sonoma: **composite attributes**, **staged migration**, and **deferred migration**. Together these capabilities enable more expressive data models, reliable migration through previously impossible schema changes, and responsive apps that defer expensive migration work to background time.

Composite attributes replace transformable types as the idiomatic way to store structured custom data — they are natively queryable via `NSPredicate`, nestable within each other, and require no custom transformer code. Staged migration solves the long-standing problem of multi-step schema evolution that exceeds lightweight migration's capabilities by letting developers decompose complex changes into a series of lightweight-eligible steps. Deferred migration allows select lightweight migration cleanup tasks (index rebuilds, column drops after table copies) to be postponed and completed via a background task.

## Key Topics

### Composite Attributes
- `NSCompositeAttributeDescription` **[NEW]** — a new attribute type (`NSCompositeAttributeType`) that encapsulates multiple named sub-attributes of any built-in Core Data type (String, Float, Int, Data, etc.).
- Sub-attributes are defined as `NSAttributeDescription` instances stored in the `elements` array of the composite; composites may be nested within each other.
- Elements cannot be `NSRelationshipDescription` — only attribute descriptions are valid; invalid elements raise `NSInvalidArgumentException`.
- In managed object subclasses, composite attributes surface as `[String: Any]?` dictionaries keyed by sub-attribute name (e.g., `colorScheme["primary"]`).
- Fully predicatable: `NSPredicate(format: "colorScheme.primary == %@", color)` uses the namespaced keypath.
- Replaces transformable attributes for structured types — no transformer code, no manual serialization, and prevents relationship faulting when embedded data would otherwise require a separate relationship fetch.
- Xcode's Core Data model editor is updated to define and manage composite attributes visually.

### Staged Migration
- Designed for model changes that exceed lightweight migration (e.g., denormalizing an entity, splitting attributes into a new entity while preserving data).
- Workflow: decompose the non-lightweight change into a sequence of individually lightweight-eligible model versions, then describe the total ordering to Core Data.
- `NSStagedMigrationManager` **[NEW]** — encapsulates the ordered list of stages, manages the migration event loop, and provides access to the migrating store via `NSPersistentContainer`.
- `NSLightweightMigrationStage` **[NEW]** — describes one or more model versions that required no decomposition and are lightweight-eligible. All lightweight model versions must be represented here.
- `NSCustomMigrationStage` **[NEW]** — describes a single decomposed step between a source and destination `NSManagedObjectModelReference`; provides `willMigrateHandler` and `didMigrateHandler` closures where custom code runs.
- `NSManagedObjectModelReference` **[NEW]** — a lazy promise for an `NSManagedObjectModel`; initialized with a model name, bundle, and mandatory `versionChecksum` (obtain from `NSManagedObjectModel.versionChecksum` or Xcode build log "Compile data model" output).
- Add the manager to store options via `NSPersistentStoreStagedMigrationManagerOptionKey`.
- Inside `willMigrateHandler`, use generic `NSManagedObject`/`NSFetchRequestResult` types rather than generated subclasses, as the subclass shape may not match the intermediate model version.

### Deferred Migration
- Defers schema cleanup tasks (index rebuilds, column drops after table copies) that do not need to block app launch.
- Opt in by setting `NSPersistentStoreDeferredLightweightMigrationOptionKey: true` in store options alongside standard lightweight migration options.
- The app opens and uses the latest schema immediately; only background cleanup is deferred.
- Check store metadata for `NSPersistentStoreDeferredLightweightMigrationOptionKey == true` to detect pending deferred work.
- Complete deferred work by calling `NSPersistentStoreCoordinator.finishDeferredLightweightMigration()`.
- Compatible with `BGProcessingTask` for scheduling deferred work when the device is idle.
- SQLite store type only; runtime compatibility back to macOS Big Sur / iOS 14.
- Staged migration and deferred migration can be combined.

## APIs & Frameworks

- `CoreData` framework
- `NSCompositeAttributeDescription` **[NEW]** — new attribute description for structured composite types
- `NSCompositeAttributeType` **[NEW]** — attribute type constant for composite attributes
- `NSCompositeAttributeDescription.elements` **[NEW]** — array of `NSAttributeDescription` or nested `NSCompositeAttributeDescription`
- `NSStagedMigrationManager` **[NEW]** — manages staged migration event loop
- `NSStagedMigrationManager(migrationStages:)` **[NEW]** — initializer accepting ordered array of stages
- `NSStagedMigrationManager.container` **[NEW]** — provides `NSPersistentContainer` during migration
- `NSLightweightMigrationStage` **[NEW]** — stage descriptor for lightweight-eligible model versions
- `NSCustomMigrationStage` **[NEW]** — stage descriptor for custom/decomposed migration steps
- `NSCustomMigrationStage.willMigrateHandler` **[NEW]** — closure invoked before the migration step executes
- `NSCustomMigrationStage.didMigrateHandler` **[NEW]** — closure invoked after the migration step completes
- `NSManagedObjectModelReference` **[NEW]** — lazy model reference with checksum validation
- `NSManagedObjectModelReference(modelName:in:versionChecksum:)` **[NEW]**
- `NSManagedObjectModel.versionChecksum` **[NEW]** — returns the model's version checksum string
- `NSPersistentStoreStagedMigrationManagerOptionKey` **[NEW]** — store option key to attach `NSStagedMigrationManager`
- `NSPersistentStoreDeferredLightweightMigrationOptionKey` **[NEW]** — store option key to enable deferred migration
- `NSPersistentStoreCoordinator.finishDeferredLightweightMigration()` **[NEW]** — completes pending deferred migration work
- `NSFetchRequest` — existing; now fully supports composite attribute namespaced keypaths in `NSPredicate`
- `NSMigratePersistentStoresAutomaticallyOption` — existing lightweight migration option
- `NSInferMappingModelAutomaticallyOption` — existing lightweight migration option
- `NSMappingModel.inferredMappingModel(forSourceModel:destinationModel:)` — existing; returns nil when changes are not lightweight-eligible

## Code Highlights

```swift
// Composite attribute: declare in managed object subclass
@NSManaged public var colorScheme: [String: Any]?

// Set composite attribute using dictionary notation
aircraft.colorScheme = [
    "primary": primaryColor.rawValue,
    "secondary": secondaryColor.rawValue,
    "tertiary": tertiaryColor.rawValue
]

// Predicate using namespaced keypath
fetchRequest.predicate = NSPredicate(format: "colorScheme.primary == %@", color)
```

```swift
// Staged migration setup
let v1Ref = NSManagedObjectModelReference(modelName: "modelV1", in: .main,
    versionChecksum: "kk8XL4OkE7gYLFHTrH6W+EhTw8w14uq1klkVRPiuiAk=")
let v2Ref = NSManagedObjectModelReference(modelName: "modelV2", in: .main,
    versionChecksum: "PA0Gbxs46liWKg7/aZMCBtu9vVIF6MlskbhhjrCd7ms=")
let v3Ref = NSManagedObjectModelReference(modelName: "modelV3", in: .main,
    versionChecksum: "iWKg7bxs46g7liWkk8XL4OkE7gYL/FHTrH6WF23Jhhs=")

let lightweightStage = NSLightweightMigrationStage([v1Ref.versionChecksum])
lightweightStage.label = "V1 to V2: Add flightData attribute"

let customStage = NSCustomMigrationStage(migratingFrom: v2Ref, to: v3Ref)
customStage.label = "V2 to V3: Denormalize model with FlightData entity"
customStage.willMigrateHandler = { migrationManager, currentStage in
    guard let container = migrationManager.container else { return }
    let context = container.newBackgroundContext()
    try context.performAndWait {
        let fetchRequest = NSFetchRequest<NSFetchRequestResult>(entityName: "Aircraft")
        fetchRequest.predicate = NSPredicate(format: "flightData != nil")
        let results = try context.fetch(fetchRequest) as! [NSManagedObject]
        for airplane in results {
            let fdEntity = NSEntityDescription.insertNewObject(
                forEntityName: "FlightData", into: context)
            fdEntity.setValue(airplane.value(forKey: "flightData"), forKey: "data")
            fdEntity.setValue(airplane, forKey: "aircraft")
            airplane.setValue(nil, forKey: "flightData")
        }
        try context.save()
    }
}

let manager = NSStagedMigrationManager([lightweightStage, customStage])
storeDescription.setOption(manager, forKey: NSPersistentStoreStagedMigrationManagerOptionKey)
```

```swift
// Deferred migration opt-in
let options: [String: Any] = [
    NSPersistentStoreDeferredLightweightMigrationOptionKey: true,
    NSMigratePersistentStoresAutomaticallyOption: true,
    NSInferMappingModelAutomaticallyOption: true
]
let store = try coordinator.addPersistentStore(
    ofType: NSSQLiteStoreType, at: storeURL, options: options)

// Complete deferred work (e.g., in a BGProcessingTask)
let metadata = coordinator.metadata(for: store)
if metadata[NSPersistentStoreDeferredLightweightMigrationOptionKey] == true {
    coordinator.finishDeferredLightweightMigration()
}
```

## Takeaways
- Composite attributes are the new standard for structured custom types in Core Data — they replace transformable attributes with a natively queryable, no-code-transformer approach that also improves fetch performance by eliminating cross-relationship faulting.
- Staged migration removes the last major obstacle to using lightweight migration exclusively: decompose any complex schema evolution into a sequence of lightweight-eligible steps, with `NSCustomMigrationStage` handlers for data preservation logic.
- Deferred migration keeps app launch fast even during large migrations by deferring schema cleanup to background time via `BGProcessingTask`.
- All three features compose: a single store can use staged migration (with deferred migration enabled) for maximum flexibility and performance.

---
_Source: WWDC23 Session 10186 page (abstract, chapters, transcript, and code samples)._
