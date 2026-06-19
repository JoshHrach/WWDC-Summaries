# SwiftData: Dive into Inheritance and Schema Migration
**WWDC25 · Session 291** · [Watch](https://developer.apple.com/videos/play/wwdc2025/291/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26, watchOS 26

## Overview
This session covers two major SwiftData capabilities: model class inheritance and schema migration. Inheritance allows SwiftData model types to form class hierarchies, enabling polymorphic queries and shared stored properties. Schema migration explains how to version your SwiftData schema and migrate existing persistent stores as your model evolves — both through lightweight migration (automatic) and custom migration stages.

## Key Topics

### Model Class Inheritance
SwiftData models (classes annotated with `@Model`) can now participate in class inheritance. A base model class can define shared properties, and subclasses add specialized properties. SwiftData stores the hierarchy in a single table (single-table inheritance) or separate tables depending on configuration. Polymorphic queries (`FetchDescriptor<BaseModel>`) return all instances of the base type and all subclasses.

Key behaviors:
- Subclasses inherit all `@Attribute` and `@Relationship` declarations from their superclass
- Queries on the base type include results from all subclasses
- Queries on a specific subclass return only instances of that exact type
- `ModelContext.fetch(_:)` with a subclass type filter returns only matching subclass instances

### Discriminator / Type Identification
When fetching a heterogeneous collection of base-type results, SwiftData provides a way to determine the concrete type of each instance at runtime, allowing safe downcasting to the appropriate subclass.

### Versioned Schema
`VersionedSchema` is a protocol that groups a set of model types into a named, versioned schema definition. Each schema version is a distinct `VersionedSchema` conformance, enabling SwiftData to track which version of the schema the persistent store was last opened with.

### SchemaMigrationPlan
`SchemaMigrationPlan` is a protocol describing the ordered list of schema versions and migration stages needed to bring an older store up to the current schema. Conforming types list the `VersionedSchema` versions in migration order and provide migration stages between consecutive versions.

### Lightweight Migration
When the only changes between schema versions are additions of new optional properties or renaming (with `@Attribute(.renamed(from:))` annotation), SwiftData can perform lightweight migration automatically with no custom code.

### Custom Migration Stages
`MigrationStage.custom(fromVersion:toVersion:willMigrate:didMigrate:)` lets developers run arbitrary Swift code during a migration — for example, splitting a single string property into two separate fields, or backfilling computed values.

## APIs & Frameworks

**SwiftData**
- `@Model` class inheritance **[NEW]** — `@Model` classes can now subclass other `@Model` classes; full property and relationship inheritance
- Polymorphic `FetchDescriptor<T>` queries **[NEW]** — fetching a base type returns all subtype instances
- `VersionedSchema` protocol **[NEW in broader usage]** — groups model types into a named, versioned schema
- `SchemaMigrationPlan` protocol **[NEW in broader usage]** — ordered list of schema versions and migration stages
- `MigrationStage.lightweight(fromVersion:toVersion:)` **[NEW in broader usage]** — automatic migration for additive/rename-only changes
- `MigrationStage.custom(fromVersion:toVersion:willMigrate:didMigrate:)` **[NEW in broader usage]** — custom migration code between two schema versions
- `@Attribute(.renamed(from:))` — marks a property as a rename of a prior property; enables lightweight rename migration
- `ModelContainer(for:migrationPlan:)` — initializes a container with a migration plan applied on first open

## Code Highlights

```swift
// Base model class
@Model
class Animal {
    var name: String
    var birthDate: Date
}

// Subclass
@Model
class Dog: Animal {
    var breed: String
    var isGoodBoy: Bool
}

// Polymorphic fetch — returns Animal and Dog instances
let allAnimals = try context.fetch(FetchDescriptor<Animal>())

// Subtype-only fetch
let allDogs = try context.fetch(FetchDescriptor<Dog>())
```

```swift
// Define versioned schemas
enum SchemaV1: VersionedSchema {
    static var models: [any PersistentModel.Type] { [Animal.self] }
    static var versionIdentifier = Schema.Version(1, 0, 0)
}

enum SchemaV2: VersionedSchema {
    static var models: [any PersistentModel.Type] { [Animal.self, Dog.self] }
    static var versionIdentifier = Schema.Version(2, 0, 0)
}

// Migration plan
enum AnimalMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] { [SchemaV1.self, SchemaV2.self] }
    static var stages: [MigrationStage] {
        [MigrationStage.lightweight(fromVersion: SchemaV1.self, toVersion: SchemaV2.self)]
    }
}

// Use the plan when creating the container
let container = try ModelContainer(for: Animal.self, Dog.self,
                                   migrationPlan: AnimalMigrationPlan.self)
```

## Takeaways
- Use model class inheritance to model "is-a" relationships in SwiftData — avoid duplicating properties across unrelated model types when a shared base class makes semantic sense.
- Polymorphic queries on a base type automatically include all subclass instances; use subclass-specific `FetchDescriptor` types to filter to exact types.
- Always version your schema with `VersionedSchema` from the start so that future migrations have a clear baseline to migrate from.
- Prefer lightweight migration (additive properties, renames with `@Attribute(.renamed(from:))`) over custom stages — it requires no code and runs automatically.
- Use `MigrationStage.custom` only when data transformation is required (splitting, combining, or backfilling fields).

---
_Source: WWDC25 Session 291 page (abstract, chapter summaries, and resource links). Note: full transcript was not accessible; summary is based on available preview content and session abstract._
