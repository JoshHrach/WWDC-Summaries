# Showcase App Data in Spotlight
**WWDC21 · Session 10098** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10098/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session introduces `NSCoreDataCoreSpotlightDelegate`, which provides automatic Spotlight indexing of Core Data content with as little as two lines of code. The delegate monitors persistent history and asynchronously updates the Spotlight index whenever managed objects change, eliminating the need to manually write Core Spotlight indexing code.

The session covers three progressively detailed implementations: a minimal two-line setup, a customized subclass with explicit attribute sets, and a full-text search feature built on top of `CSSearchQuery` that reuses the Spotlight-indexed data inside the app. New in iOS 15 and macOS Monterey: a revised initializer, `startSpotlightIndexing`/`stopSpotlightIndexing` controls, an index update notification, and `deleteSpotlightIndex` for privacy-preserving index management without deleting user data.

## Key Topics

### Minimal Setup
- Mark entity attributes as "Index in Spotlight" in the Core Data model editor.
- Set a "Core Data Spotlight display name" on each entity (an `NSExpression` evaluated at indexing time — e.g., a key path like `Tag.name`).
- Prerequisites: store type must be `NSSQLiteStoreType` with `NSPersistentHistoryTrackingKey = true`.
- Create a delegate with the new initializer `NSCoreDataCoreSpotlightDelegate(forStoreWith:coordinator:)` **[NEW; old `forStoreWith:model:` deprecated]**.
- Call `startSpotlightIndexing()`. Done.

### Customizing the Delegate
- Subclass `NSCoreDataCoreSpotlightDelegate` and override:
  - `domainIdentifier()` — unique reverse-DNS string identifying the index domain.
  - `indexName()` — human-readable index name for multi-index scenarios.
  - `attributeSet(for:)` — return a fully configured `CSSearchableItemAttributeSet` per entity type; use `contentType` (`.image`, `.text`, etc.) and populate `displayName`, `thumbnailData`, `keywords`, `identifier`, etc.
- If a relationship is indexed, `attributeSet(for:)` must be overridden to define what aspect of the relationship is indexed.
- Concurrent access to `CSSearchableItemAttributeSet` properties has undefined behavior — modify on one thread at a time.

### Start/Stop Indexing
- `startSpotlightIndexing()` — activates indexing (must be called explicitly with new initializer).
- `stopSpotlightIndexing()` **[NEW]** — pauses indexing during CPU/disk-intensive operations.
- `isIndexingEnabled` property — query current state.

### Index Update Notifications (NEW)
- Subscribe to `NSCoreDataCoreSpotlightDelegate.indexDidUpdateNotification` **[NEW]**.
- Posted after `NSManagedObjectContext.save()` or batch operations complete indexing.
- `userInfo` contains `NSStoreUUIDKey` (store UUID string) and `NSPersistentHistoryTokenKey` (the history token processed).

### Privacy-Preserving Index Deletion (NEW)
- `deleteSpotlightIndex(completionHandler:)` **[NEW]** — removes all indexed content from Spotlight without deleting Core Data entities. Completion handler provides any `Error`.
- Replaces the previous approach of deleting the entire Core Data graph.

### Full-Text Search with CSSearchQuery
- Build a query string using `CSSearchableItemAttributeSet` property names (e.g., `(keywords == "term*"cwdt)` where `c`=case insensitive, `d`=diacritic insensitive, `w`=word-based).
- Create `CSSearchQuery(queryString:attributes:)`.
- Set `foundItemsHandler` (called multiple times with batches of `CSSearchableItem`).
- Set `completionHandler` (called once on finish or error).
- Call `query.start()`.
- Use returned `CSSearchableItem` identifiers to fetch corresponding `NSManagedObject` instances.

## APIs & Frameworks

- `CoreData` framework
- `NSCoreDataCoreSpotlightDelegate` class
- `NSCoreDataCoreSpotlightDelegate(forStoreWith:coordinator:)` **[NEW]** — designated initializer (replaces deprecated `forStoreWith:model:`)
- `NSCoreDataCoreSpotlightDelegate.startSpotlightIndexing()` — start asynchronous indexing
- `NSCoreDataCoreSpotlightDelegate.stopSpotlightIndexing()` **[NEW]**
- `NSCoreDataCoreSpotlightDelegate.isIndexingEnabled` property
- `NSCoreDataCoreSpotlightDelegate.deleteSpotlightIndex(completionHandler:)` **[NEW]**
- `NSCoreDataCoreSpotlightDelegate.indexDidUpdateNotification` **[NEW]** — `NSNotification.Name`
- `NSStoreUUIDKey` — userInfo key for store UUID in index update notification
- `NSPersistentHistoryTokenKey` — userInfo key for history token in index update notification
- `NSPersistentHistoryTrackingKey` — store option required for Spotlight delegate
- `NSSQLiteStoreType` — required store type
- `CoreSpotlight` framework
- `CSSearchableItemAttributeSet` — metadata container for a searchable item
- `CSSearchableItemAttributeSet(contentType:)` — initialize with `UTType` (`.image`, `.text`, etc.)
- `CSSearchableItemAttributeSet.displayName`
- `CSSearchableItemAttributeSet.thumbnailData`
- `CSSearchableItemAttributeSet.keywords`
- `CSSearchableItemAttributeSet.identifier`
- `CSSearchableItem` — result item from a Spotlight query
- `CSSearchQuery` — full-text search query object
- `CSSearchQuery(queryString:attributes:)` — initialize with query predicate and attribute names
- `CSSearchQuery.foundItemsHandler` — batch results callback
- `CSSearchQuery.completionHandler` — final callback
- `CSSearchQuery.start()` — execute the query
- `NSExpression` — used for Core Data Spotlight display name in model editor
- Xcode Core Data model editor — "Index in Spotlight" attribute checkbox, "Core Data Spotlight display name" field

## Code Highlights

Minimal two-line setup:
```swift
let spotlightDelegate = NSCoreDataCoreSpotlightDelegate(
    forStoreWith: description, coordinator: coordinator)
spotlightDelegate.startSpotlightIndexing()
```

Custom attribute set for an image entity:
```swift
override func attributeSet(for object: NSManagedObject) -> CSSearchableItemAttributeSet? {
    guard let photo = object as? Photo else { return nil }
    let set = CSSearchableItemAttributeSet(contentType: .image)
    set.identifier = photo.uniqueName
    set.displayName = photo.userSpecifiedName
    set.thumbnailData = photo.thumbnail?.data
    set.keywords = photo.tags?.compactMap { ($0 as? Tag)?.name }
    return set
}
```

Full-text search query:
```swift
let query = CSSearchQuery(
    queryString: "(keywords == \"\(escaped)*\"cwdt)",
    attributes: ["displayName", "keywords"])
query.foundItemsHandler = { items in self.spotlightFoundItems += items }
query.completionHandler = { _ in /* reload UI with found items */ }
query.start()
```

## Takeaways
- `NSCoreDataCoreSpotlightDelegate` reduces Spotlight indexing to two lines of code while handling change tracking automatically via persistent history.
- Use `deleteSpotlightIndex(completionHandler:)` for privacy-respecting index cleanup — it removes search results without touching user data.
- Override `attributeSet(for:)` in a subclass to gain full control over what is indexed and how results appear, especially important for image content (thumbnail) and keyword-based search.
- The same `CSSearchQuery` API that Spotlight uses can power full-text search within your app, creating a unified index with no duplication.

---
_Source: WWDC21 Session 10098 page (abstract, chapter summaries, code samples, and resource links)._
