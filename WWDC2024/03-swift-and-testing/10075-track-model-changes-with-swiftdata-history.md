# Track Model Changes with SwiftData History
**WWDC24 · Session 10075** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10075/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11

## Overview
SwiftData History is a new API that lets apps track all modifications to their data store over time, in the order they occurred. Rather than querying what currently exists, history allows apps to query what changed—which models were inserted, updated, or deleted—since a previous point in time, identified by a token. This enables use cases like syncing changes with a remote server after coming back online, reflecting changes made by an app extension (e.g., a widget) in the main app's UI, and efficiently computing UI state changes without full re-fetches.

The session uses the SampleTrips app to demonstrate the full workflow: marking model attributes to preserve values on deletion (tombstones), fetching history transactions filtered by token and author, processing the typed insert/update/delete change records, persisting the latest token to UserDefaults, and updating SwiftUI state when the scene becomes active.

For developers building custom data stores, the session also covers the requirements to implement the `HistoryProviding` protocol and create matching custom transaction, change, and token types.

## Key Topics

### Fundamentals: Transactions, Changes, and Tokens
Each `ModelContext.save()` creates a `HistoryTransaction` that groups all changes made in that save. Changes within a transaction preserve insertion order and represent individual model instances that were inserted (`HistoryInsert`), updated (`HistoryUpdate`), or deleted (`HistoryDelete`). All three change types are parameterized by a `PersistentModel` type, enabling KeyPath access to properties.

A `HistoryToken` acts as a bookmark in the transaction stream. Tokens are store-specific; expired tokens (from deleted history) throw `historyTokenExpired` and should be discarded.

### Preserving Values on Deletion (Tombstones)
Deleted models lose all data. Use `@Attribute(.preserveValueOnDeletion)` on specific properties to retain their values in the delete change record (a "tombstone"). This is essential for maintaining identifiers needed to correlate deleted records with upstream systems.

### Fetching and Processing Changes
Create a `HistoryDescriptor<DefaultHistoryTransaction>` with a predicate filtering by `transaction.token > lastToken` and optionally by `transaction.author`. Call `ModelContext.fetchHistory(_:)` to get `[DefaultHistoryTransaction]`. Iterate transactions → changes, check change type with a `switch`, and access `change.changedPersistentIdentifier` to re-fetch related models.

### Tracking Transaction Authors
In a widget or app extension, set `ModelContext.author = "com.example.widget"` before saving to tag changes with an author string. In the main app, filter history by that author string to find only the extension's changes.

### Custom Store Support
Custom data stores can support history by implementing `HistoryProviding` and defining custom `HistoryTransaction`, `HistoryInsert`, `HistoryUpdate`, `HistoryDelete`, and `HistoryToken` types. The store must define transaction and change boundaries, handle tombstone storage for deleted values, and manage history record lifetime (when to delete old history).

## APIs & Frameworks

- `SwiftData` framework
- `HistoryTransaction` — protocol; represents a group of changes in a single save
- `DefaultHistoryTransaction` **[NEW]** — concrete transaction type for the default store; conforms to `HistoryTransaction`
- `DefaultHistoryTransaction.token` — the `DefaultHistoryToken` for this transaction
- `DefaultHistoryTransaction.author` — string identifying which process authored the changes
- `DefaultHistoryTransaction.changes` — ordered collection of changes in the transaction
- `HistoryChange` — protocol; base for insert/update/delete
- `DefaultHistoryInsert<T: PersistentModel>` **[NEW]** — insert change for a specific model type in the default store
- `DefaultHistoryUpdate<T: PersistentModel>` **[NEW]** — update change for a specific model type
- `DefaultHistoryDelete<T: PersistentModel>` **[NEW]** — delete change; may carry tombstone values
- `HistoryChange.changedPersistentIdentifier` — `PersistentIdentifier` of the changed model
- `HistoryToken` — protocol; bookmark in the history stream
- `DefaultHistoryToken` **[NEW]** — concrete token for the default store; `Codable` and `Comparable`
- `HistoryDescriptor<T: HistoryTransaction>` **[NEW]** — configures a history fetch; has `predicate` property
- `ModelContext.fetchHistory(_:)` **[NEW]** — fetches matching history transactions
- `ModelContext.deleteHistory(_:)` — deletes history records (removes expired tokens)
- `@Attribute(.preserveValueOnDeletion)` **[NEW]** — modifier preserving property values in delete tombstones
- `ModelContext.author` **[NEW]** — string property identifying the context's author for history tracking
- `HistoryProviding` **[NEW]** — protocol for custom data stores to implement history support
- `PersistentModel` — base protocol for SwiftData model classes
- `PersistentIdentifier` — stable cross-process identifier for a model instance
- `#Unique<T>([\.property])` — macro declaring uniqueness constraints on a model
- `FetchDescriptor<T>` — existing descriptor for querying current model state

## Code Highlights

Mark attributes for tombstone preservation on deletion:
```swift
@Model class Trip {
    @Attribute(.preserveValueOnDeletion)
    var name: String
    @Attribute(.preserveValueOnDeletion)
    var startDate: Date
    @Attribute(.preserveValueOnDeletion)
    var endDate: Date
}
```

Fetch history transactions filtered by token and author:
```swift
var historyDescriptor = HistoryDescriptor<DefaultHistoryTransaction>()
if let token {
    historyDescriptor.predicate = #Predicate { transaction in
        (transaction.token > token) && (transaction.author == author)
    }
}
let transactions = try taskContext.fetchHistory(historyDescriptor)
```

Process changes by type:
```swift
for transaction in transactions {
    for change in transaction.changes {
        let modelID = change.changedPersistentIdentifier
        switch change {
        case .insert(_ as DefaultHistoryInsert<LivingAccommodation>):
            resultTrips.insert(matchedTrip)
        case .update(_ as DefaultHistoryUpdate<LivingAccommodation>):
            resultTrips.update(with: matchedTrip)
        case .delete(_ as DefaultHistoryDelete<LivingAccommodation>):
            resultTrips.remove(matchedTrip)
        default: break
        }
    }
}
```

Persist token to UserDefaults across launches:
```swift
let tokenData = UserDefaults.standard.data(forKey: "historyToken")
var historyToken = tokenData.flatMap { try? JSONDecoder().decode(DefaultHistoryToken.self, from: $0) }
// ... process history ...
if let newToken, let encoded = try? JSONEncoder().encode(newToken) {
    UserDefaults.standard.set(encoded, forKey: "historyToken")
}
```

## Takeaways

- SwiftData History's `HistoryDescriptor` + `ModelContext.fetchHistory(_:)` lets apps efficiently discover exactly which model instances changed since a saved token, eliminating the need for full re-fetches or manual diffing.
- `@Attribute(.preserveValueOnDeletion)` creates tombstone values in delete change records, preserving key identifiers needed to correlate deletions with remote systems or other processes.
- Set `ModelContext.author` in extensions (widgets, background tasks) so the main app can filter history to only the changes that originated outside it.
- `DefaultHistoryToken` is `Codable`—serialize it to UserDefaults or any persistent store to resume history processing across app launches or process boundaries.

---
_Source: WWDC24 Session 10075 page (abstract, chapter summaries, code samples, and resource links)._
