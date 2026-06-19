# Evolve your Core Data schema
**WWDC22 · Session 10120** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10120/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
Core Data requires that the underlying SQLite store schema always matches the app's current managed object model. Any change to the model — adding an attribute, renaming a relationship, restructuring entity hierarchies — must be materialized in the store through a migration process. This session teaches developers how to use Core Data's built-in lightweight migration to handle the vast majority of schema changes automatically, with zero custom mapping code.

The session explains what operations are eligible for lightweight migration (attribute, relationship, and entity additions/removals/renames) and how to decompose complex, ineligible migrations into a chain of simple, lightweight-eligible steps. The final section covers the additional constraints that CloudKit imposes on schema changes, because the CloudKit Production environment is immutable once promoted.

## Key Topics

**What is schema migration** — When the compiled managed object model no longer matches the store's persisted schema, Core Data raises `NSPersistentStoreIncompatibleVersionHashError`. Migration materializes model changes into the underlying SQLite schema before the store can be opened.

**Lightweight migration** — Core Data infers a `NSMappingModel` automatically by comparing source and destination models. Eligible changes include: adding/removing/renaming attributes and relationships, changing relationship cardinality (to-one ↔ to-many, ordered ↔ unordered), adding/removing/renaming entities, and moving attributes within an entity hierarchy. Renaming uses a "renaming identifier" set in the Xcode Data Model Editor's property inspector.

**Enabling lightweight migration** — Set `NSMigratePersistentStoresAutomaticallyOption` and `NSInferMappingModelAutomaticallyOption` to `true` in the options dictionary passed to `NSPersistentStoreCoordinator.addPersistentStore`. `NSPersistentContainer` and `NSPersistentStoreDescription` set these options automatically.

**Staging complex migrations** — When a single migration step is ineligible for lightweight migration (e.g., switching a binary attribute from external storage to inline), decompose it into multiple intermediate model versions (A → A′ → A″ → B) where every individual step qualifies. Any app-specific data transformation executed between steps must be "restartable" in case the process is terminated mid-migration.

**Core Data + CloudKit schema constraints** — CloudKit's Production environment is immutable: only adding new fields to existing record types or adding new record types is supported. Lightweight migration only updates the local store file; developers must also run the schema initializer and promote changes via CloudKit Console. Strategies for cross-version compatibility: incremental field additions, entity versioning with a version attribute, or migrating to a new CloudKit container.

## APIs & Frameworks

### Core Data
- `NSManagedObjectModel` — compiled data model
- `NSPersistentStoreCoordinator` — manages persistent stores
- `NSPersistentStoreCoordinator.addPersistentStore(ofType:configurationName:at:options:)` — add store with migration options
- `NSMigratePersistentStoresAutomaticallyOption` — option key to trigger automatic migration
- `NSInferMappingModelAutomaticallyOption` — option key to infer mapping model automatically
- `NSMappingModel` — describes how to transform source model to destination model
- `NSMappingModel.inferredMappingModel(forSourceModel:destinationModel:)` — test whether lightweight migration is possible without executing it
- `NSPersistentContainer` — automatically applies lightweight migration options
- `NSPersistentStoreDescription` — automatically applies lightweight migration options
- `NSPersistentStoreIncompatibleVersionHashError` — error code raised when store schema mismatches the model
- `NSPersistentCloudKitContainer` — Core Data + CloudKit integration container
- `NSPersistentCloudKitContainerOptions` — configure a new CloudKit container for a store
- Renaming identifier — property inspector field in Xcode Data Model Editor used to rename attributes/relationships/entities across model versions

### CloudKit (schema interaction)
- CloudKit Development environment — schema can be freely modified
- CloudKit Production environment — record types and fields are immutable; only additions allowed
- CloudKit Console — tool to promote schema from Development to Production

## Code Highlights

Manually enabling lightweight migration:
```swift
import CoreData

let storeURL = NSURL.fileURL(withPath: "/path/to/store")
let momURL   = NSURL.fileURL(withPath: "/path/to/model")
guard let mom = NSManagedObjectModel(contentsOf: momURL) else {
    fatalError("Error initializing managed object model for URL: \(momURL)")
}
let coordinator = NSPersistentStoreCoordinator(managedObjectModel: mom)
let opts: [AnyHashable: Any] = [
    NSMigratePersistentStoresAutomaticallyOption: true,
    NSInferMappingModelAutomaticallyOption: true
]
try coordinator.addPersistentStore(ofType: NSSQLiteStoreType,
                                   configurationName: nil,
                                   at: storeURL,
                                   options: opts)
```

Using `NSPersistentContainer` (lightweight migration is automatic):
```swift
let container = NSPersistentContainer(name: "MyModel")
container.loadPersistentStores { _, error in
    if let error { fatalError("Failed to load store: \(error)") }
}
```

## Takeaways
- `NSPersistentContainer` and `NSPersistentStoreDescription` enable lightweight migration automatically — there is no need for manual option configuration in most apps.
- Complex schema changes that exceed lightweight migration's capabilities can be decomposed into a sequence of individually-eligible intermediate model versions; any interposed data transformation logic must be restartable.
- CloudKit Production schemas are immutable — plan for additive-only migrations, and always promote changes via the CloudKit Console in addition to migrating the local store.
- Use `NSMappingModel.inferredMappingModel(forSourceModel:destinationModel:)` to validate that a planned migration is lightweight-eligible before shipping it.

---
_Source: WWDC22 Session 10120 page (abstract, chapter summaries, code samples, and resource links)._
