# Create a Custom Data Store with SwiftData
**WWDC24 · Session 10138** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10138/)

_Platforms:_ iOS 18, iPadOS 18, macOS 15, tvOS 18, watchOS 11

## Overview
SwiftData's new DataStore protocol in iOS 18 lets developers replace the default Core Data / SQLite backing store with any persistence layer they choose — a JSON file, a remote database, an in-memory cache, or a custom binary format — while keeping the full SwiftData model, query, and predicate surface unchanged above it.

This session introduces the protocol requirements, explains the snapshot-based communication contract between SwiftData and the store, and walks through a complete implementation of a JSON-backed custom store. The result is a store that persists `@Model` objects to a flat JSON file on disk, handles fetch and save operations, and integrates transparently with `ModelContext` and `@Query`.

## Key Topics
- **`DataStore` protocol** — the new abstraction; conformers implement `fetch`, `save`, `erase`, and `identifier` requirements.
- **Snapshot model** — SwiftData communicates with the store through `DataStoreSnapshot` dictionaries rather than model instances, decoupling the store from the model graph.
- **`DataStoreSaveChanges`** — the object passed to `save(_:)` containing inserted, updated, and deleted snapshot collections.
- **`DefaultStore` and `ModelConfiguration`** — existing entry points remain unchanged; custom stores plug in via `ModelConfiguration(schema:, dataStoreClass:)`.
- **Predicate and sort descriptor translation** — stores receive `FetchDescriptor` with a `Predicate` tree and sort descriptors; the JSON store implements in-memory filtering and sorting.
- **History and conflict resolution** — advanced stores can implement `DataStoreHistory` for change tracking; the session focuses on the simpler stateless path.

## APIs & Frameworks

**SwiftData**
- **[NEW]** `DataStore` protocol — primary conformance point for custom stores
  - `fetch(_:) throws -> [DataStoreSnapshot]` — handle a `FetchDescriptor`
  - `save(_:) throws` — handle inserted/updated/deleted snapshot batches
  - `erase() throws` — wipe all persisted data
  - `identifier` — a `String` uniquely identifying this store instance
- **[NEW]** `DataStoreSnapshot` — dictionary-like type mapping `String` keys to `DataStoreValue`; represents a single model object's persisted state
- **[NEW]** `DataStoreValue` — enum of supported value kinds (int, double, string, data, url, bool, date, nil, relationship, etc.)
- **[NEW]** `DataStoreSaveChanges` — passed to `save(_:)`; has `.inserted`, `.updated`, `.deleted` arrays of `DataStoreSnapshot`
- **[NEW]** `DataStoreConfiguration` — base class for custom store configurations; subclass to carry store-specific options (e.g., file URL)
- `FetchDescriptor<T>` — unchanged; passed to `fetch`; carries `.predicate` and `.sortBy`
- `ModelConfiguration(schema:dataStoreClass:)` — **[NEW]** overload accepting a custom `DataStore.Type`
- `ModelContainer` — unchanged; accepts `ModelConfiguration` with custom store
- `ModelContext` — unchanged user-facing API; all CRUD flows through the store protocol transparently
- `@Query` — unchanged; works with custom stores via the same `FetchDescriptor` path
- `@Model` macro — unchanged; annotate model types as before

## Code Highlights
Minimal custom store skeleton:

```swift
final class JSONDataStore: DataStore {
    let configuration: JSONStoreConfiguration
    required init(_ configuration: JSONStoreConfiguration) throws {
        self.configuration = configuration
    }
    func fetch<T>(_ request: FetchDescriptor<T>) throws -> [DataStoreSnapshot] {
        // Load JSON, filter/sort, return snapshots
    }
    func save(_ changes: DataStoreSaveChanges) throws {
        // Apply insertions, updates, deletions; write JSON
    }
    func erase() throws { try FileManager.default.removeItem(at: configuration.url) }
    var identifier: String { configuration.url.absoluteString }
}
```

Wire the custom store into a `ModelContainer`:

```swift
let config = ModelConfiguration(schema: Schema([Trip.self]),
                                 dataStoreClass: JSONDataStore.self)
let container = try ModelContainer(for: Trip.self, configurations: config)
```

## Takeaways
- The `DataStore` protocol is the right solution when the default SQLite store is not appropriate — e.g., syncing with a server-owned format, embedding into an existing database, or unit-testing with an in-memory store.
- All SwiftData query, predicate, and relationship APIs remain fully functional; only the persistence layer changes.
- Implement `DataStoreHistory` if your store needs change tracking for sync or audit purposes; the base protocol does not require it.
- Start from the JSON store sample code linked in the session resources as a reference implementation before building a production store.

---
_Source: WWDC24 Session 10138 page (abstract, chapter summaries, code samples, and resource links)._
