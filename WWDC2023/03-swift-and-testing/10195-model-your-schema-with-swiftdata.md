# Model Your Schema with SwiftData
**WWDC23 · Session 10195** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10195/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
This session covers advanced SwiftData schema authoring and schema evolution. It builds directly on the basics from "Meet SwiftData" to show how schema macros (`@Attribute`, `@Relationship`, `@Transient`) can fine-tune persistence behavior, and how to handle schema changes across app releases using `VersionedSchema` and `SchemaMigrationPlan`. The running example is the SampleTrips app, a trip-planning app that evolves through three schema versions.

## Key Topics

### @Attribute Schema Macro

**Uniqueness constraint:**
- `@Attribute(.unique)` enforces a uniqueness constraint on a property in the backing store
- Eligible types: primitive value types (Int, String, UUID, etc.) and to-one relationships
- Uniqueness conflicts use **upsert** semantics: if an insert collides with an existing record, the existing record is updated to the latest values instead of causing an error

**Preserving data on property rename:**
- `@Attribute(originalName: "old_name")` maps an existing stored column name to a renamed Swift property
- Without `originalName`, SwiftData sees a rename as adding a new property and dropping the old one, causing data loss
- Adding `originalName` makes the rename a **lightweight migration** — no custom code required

**Other @Attribute options:**
- `.externalStorage` – stores large binary data (e.g., images) outside the main database file
- Transformable support – allows storing types that implement `ValueTransformer`

### @Relationship Schema Macro

**Implicit inverses:**
- SwiftData automatically discovers and sets inverse relationships between `@Model` types without any annotation
- Default delete rule on implicit inverses: **nullify** (related objects are set to nil when the owner is deleted)

**Cascade delete rule:**
- `@Relationship(.cascade)` deletes the related objects when the owning object is deleted
- Apply to `bucketList` and `livingAccommodation` on Trip so they are deleted alongside the trip

**Other @Relationship options:**
- `originalName:` – preserves the stored relationship name across a rename (same as @Attribute)
- `minimumModelCount:` / `maximumModelCount:` – specifies cardinality constraints on to-many relationships

### @Transient Macro
- `@Transient` excludes a property from persistence entirely — the value is never written to or read from the store
- The property must have a default value (for correct initialization when objects are fetched from the store, where the transient value is not present)
- Use for derived/computed values, UI state, view counters, cached calculations

### Schema Evolution with VersionedSchema and SchemaMigrationPlan

**When migrations occur:**
- Adding or removing a property, renaming a property (without `originalName`), changing a constraint or delete rule, and other model changes all trigger a migration when a new app version opens the existing store

**VersionedSchema:**
- Protocol conforming type (typically an `enum`) that captures a complete, frozen snapshot of the schema at a specific app release
- Declares `static var models: [any PersistentModel.Type]` listing all model types in that version
- Each version's `@Model` classes are nested inside the `VersionedSchema` enum to namespace them

**SchemaMigrationPlan:**
- Protocol conforming type that declares the total ordered sequence of `VersionedSchema` versions (`static var schemas: [any VersionedSchema.Type]`) and the migration stages between consecutive versions (`static var stages: [MigrationStage]`)
- SwiftData applies stages in order; users can upgrade from any earlier version to the latest

**Migration stages — two types:**

1. `MigrationStage.lightweight(fromVersion:toVersion:)` – no code required; SwiftData handles the migration automatically. Eligible changes include: adding `originalName` for a rename, changing delete rules, adding non-unique attributes with defaults, and similar non-destructive schema changes.

2. `MigrationStage.custom(fromVersion:toVersion:willMigrate:didMigrate:)` – provides `willMigrate` and `didMigrate` closures that receive a `ModelContext`. Use when data transformation is needed before or after the schema change. Example: deduplicate Trip records before applying the `.unique` constraint on the `name` property.

**Configuring migrations on ModelContainer:**
- Pass `migrationPlan:` to `ModelContainer(for:migrationPlan:)` to register the plan
- SwiftData automatically detects the current store version and runs the necessary stages

## APIs & Frameworks

- **SwiftData** **[NEW]** – Swift-native persistence framework
- `@Model` macro **[NEW]** – designates a class as a SwiftData model type
- `@Attribute(.unique)` **[NEW]** – uniqueness constraint; upsert behavior on conflict
- `@Attribute(originalName:)` **[NEW]** – maps a renamed Swift property to its stored column name; enables lightweight rename migration
- `@Attribute(.externalStorage)` **[NEW]** – stores large binary data externally
- `@Relationship(.cascade)` **[NEW]** – cascade delete rule on a relationship
- `@Relationship(originalName:)` **[NEW]** – rename a relationship without data loss
- `@Relationship(minimumModelCount:maximumModelCount:)` **[NEW]** – relationship cardinality constraints
- `@Transient` macro **[NEW]** – excludes a property from persistence; requires a default value
- `VersionedSchema` protocol **[NEW]** – captures a frozen schema snapshot; contains nested `@Model` class definitions and `static var models` list
- `SchemaMigrationPlan` protocol **[NEW]** – declares ordered schema versions and migration stages; `static var schemas` and `static var stages`
- `MigrationStage.lightweight(fromVersion:toVersion:)` **[NEW]** – no-code automatic migration
- `MigrationStage.custom(fromVersion:toVersion:willMigrate:didMigrate:)` **[NEW]** – code-driven migration with `ModelContext` access
- `ModelContainer(for:migrationPlan:)` **[NEW]** – registers the migration plan; runs migrations automatically on store open
- `FetchDescriptor<T>` **[NEW]** – used in `willMigrate` closure to fetch old-version objects for transformation
- `ModelContext.fetch(_:)` **[NEW]** – fetches objects from within a migration closure
- `ModelContext.save()` – commits changes made during a migration stage

## Code Highlights

Full evolved @Model with all schema macros:
```swift
@Model final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    @Attribute(originalName: "start_date") var startDate: Date
    @Attribute(originalName: "end_date") var endDate: Date

    @Relationship(.cascade) var bucketList: [BucketListItem]? = []
    @Relationship(.cascade) var livingAccommodation: LivingAccommodation?

    @Transient var tripViews: Int = 0
}
```

Three versioned schema snapshots:
```swift
enum SampleTripsSchemaV1: VersionedSchema {
    static var models: [any PersistentModel.Type] {
        [Trip.self, BucketListItem.self, LivingAccommodation.self]
    }
    @Model final class Trip {
        var name: String
        var destination: String
        var start_date: Date
        var end_date: Date
        var bucketList: [BucketListItem]? = []
        var livingAccommodation: LivingAccommodation?
    }
}

enum SampleTripsSchemaV2: VersionedSchema {
    static var models: [any PersistentModel.Type] { [Trip.self, /*...*/ ] }
    @Model final class Trip {
        @Attribute(.unique) var name: String
        // ... start_date, end_date still use old names
    }
}

enum SampleTripsSchemaV3: VersionedSchema {
    static var models: [any PersistentModel.Type] { [Trip.self, /*...*/ ] }
    @Model final class Trip {
        @Attribute(.unique) var name: String
        @Attribute(originalName: "start_date") var startDate: Date
        @Attribute(originalName: "end_date") var endDate: Date
    }
}
```

Migration plan with one custom and one lightweight stage:
```swift
enum SampleTripsMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] {
        [SampleTripsSchemaV1.self, SampleTripsSchemaV2.self, SampleTripsSchemaV3.self]
    }

    static var stages: [MigrationStage] { [migrateV1toV2, migrateV2toV3] }

    static let migrateV1toV2 = MigrationStage.custom(
        fromVersion: SampleTripsSchemaV1.self,
        toVersion: SampleTripsSchemaV2.self,
        willMigrate: { context in
            let trips = try? context.fetch(FetchDescriptor<SampleTripsSchemaV1.Trip>())
            // Deduplicate trips before unique constraint is enforced
            try? context.save()
        },
        didMigrate: nil
    )

    static let migrateV2toV3 = MigrationStage.lightweight(
        fromVersion: SampleTripsSchemaV2.self,
        toVersion: SampleTripsSchemaV3.self
    )
}
```

Register the migration plan on ModelContainer:
```swift
struct TripsApp: App {
    let container = ModelContainer(
        for: Trip.self,
        migrationPlan: SampleTripsMigrationPlan.self
    )

    var body: some Scene {
        WindowGroup { ContentView() }
            .modelContainer(container)
    }
}
```

## Takeaways
- Always use `@Attribute(originalName:)` when renaming a property — without it, SwiftData treats the rename as a new property and drops the old column, silently losing all stored data.
- Uniqueness constraints require a custom migration stage when the existing data may contain duplicates; use the `willMigrate` closure to deduplicate before the unique constraint is enforced.
- `@Transient` properties must have default values — when a model object is fetched from the store, the transient property is initialized to its default since no stored value exists.
- Every schema change that ships to users should be captured in a `VersionedSchema`; failing to track schema versions makes it impossible to migrate users who skipped intermediate app releases.

---
_Source: WWDC23 Session 10195 page (abstract, chapter summaries, code samples, and transcript)._
