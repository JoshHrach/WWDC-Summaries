# Advances in Diffable Data Sources
**WWDC20 · Session 10045** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10045/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
iOS 14 brings two significant additions to the Diffable Data Source API introduced in iOS 13: Section Snapshots and first-class Reordering Support. These changes expand Diffable Data Source's capabilities to support outline-style hierarchical UIs (common in iPadOS 14's sidebar-based layouts) and to automate the commit of user-initiated drag-to-reorder interactions.

Section Snapshots (`NSDiffableDataSourceSectionSnapshot`) allow data sources to be composed from per-section chunks of data, each independently managed. They also model hierarchical parent-child relationships required for expandable/collapsible outline views. Reordering support uses a new `reorderingHandlers` property on `UICollectionViewDiffableDataSource` to let the framework automatically update the snapshot while notifying the app via `NSDiffableDataSourceTransaction` so it can persist the new order to its backing store.

## Key Topics
- **`NSDiffableDataSourceSectionSnapshot`** **[NEW]** — A section-scoped snapshot type generic over `ItemIdentifierType`; no section identifier concept (applied to a specific section via `apply(_:to:)`); supports `append(_:to:)` with optional parent for hierarchical data.
- **Hierarchical / outline data** — Parent-child relationships created by supplying a parent to `append(_:to:)`; expansion state managed via `expand(_:)` / `collapse(_:)` / `isExpanded(_:)`; child-only sub-snapshots retrieved via `snapshot(of:includingParent:)`.
- **Composable data sources** — Multiple independent section snapshots applied to the same `UICollectionViewDiffableDataSource` via `apply(_:to:)` (one per section); the overall section order is still managed by the existing `NSDiffableDataSourceSnapshot`.
- **`UICollectionViewDiffableDataSource.sectionSnapshotHandlers`** **[NEW]** — A `SectionSnapshotHandlers<Item>` struct with optional closures: `shouldExpandItem`, `willExpandItem`, `shouldCollapseItem`, `willCollapseItem`, and `snapshotForExpandingParent` (for lazy loading of child content).
- **Cell Outline Disclosure Accessory** — Standard UIKit cell accessory that drives automatic section snapshot expand/collapse; the framework updates the section snapshot and re-applies it automatically.
- **Reordering support** **[NEW]** — `UICollectionViewDiffableDataSource.reorderingHandlers` (`ReorderingHandlers<Item>` struct): `canReorderItem`, `willReorder`, `didReorder` closures. Both `canReorderItem` and `didReorder` must be provided to enable the feature.
- **`NSDiffableDataSourceTransaction`** **[NEW]** — Passed to `didReorder`; contains `initialSnapshot`, `finalSnapshot`, `difference` (Swift `CollectionDifference<Item>`), and `sectionTransactions` (per-section detail).
- **`NSDiffableDataSourceSectionTransaction`** **[NEW]** — Per-section reordering detail: `sectionIdentifier`, `initialSnapshot`, `finalSnapshot`, `difference`.
- **Applying `CollectionDifference` to backing store** — Use `Array.applying(_:)` with the transaction's `difference` to update an array-backed source of truth in one line.

## APIs & Frameworks

### UIKit
- **`NSDiffableDataSourceSectionSnapshot<ItemIdentifierType>`** **[NEW]** — `append(_:to:)`, `insert(_:before:)`, `insert(_:after:)`, `delete(_:)`, `deleteAll()`, `expand(_:)`, `collapse(_:)`, `replace(childrenOf:using:)`, `insert(_:before:)` (snapshot overload), `insert(_:after:)` (snapshot overload), `isExpanded(_:)`, `isVisible(_:)`, `contains(_:)`, `level(of:)`, `index(of:)`, `parent(of:)`, `snapshot(of:includingParent:)`, `items`, `rootItems`, `visibleItems`
- **`UICollectionViewDiffableDataSource.apply(_:to:animatingDifferences:completion:)`** **[NEW]** — Applies a section snapshot to a specific section
- **`UICollectionViewDiffableDataSource.snapshot(for:)`** **[NEW]** — Returns the current section snapshot for a section
- **`UICollectionViewDiffableDataSource.sectionSnapshotHandlers`** **[NEW]** — `SectionSnapshotHandlers<Item>` struct
- **`UICollectionViewDiffableDataSource.SectionSnapshotHandlers`** **[NEW]** — `shouldExpandItem`, `willExpandItem`, `shouldCollapseItem`, `willCollapseItem`, `snapshotForExpandingParent`
- **`UICollectionViewDiffableDataSource.reorderingHandlers`** **[NEW]** — `ReorderingHandlers<Item>` struct
- **`UICollectionViewDiffableDataSource.ReorderingHandlers`** **[NEW]** — `canReorderItem`, `willReorder`, `didReorder`
- **`NSDiffableDataSourceTransaction<Section, Item>`** **[NEW]** — `initialSnapshot`, `finalSnapshot`, `difference`, `sectionTransactions`
- **`NSDiffableDataSourceSectionTransaction<Section, Item>`** **[NEW]** — `sectionIdentifier`, `initialSnapshot`, `finalSnapshot`, `difference`

### Swift Standard Library
- **`CollectionDifference<Item>`** — Returned in transactions; `Array.applying(_:)` can apply it directly to an array backing store

## Code Highlights

Build an outline with hierarchical section snapshot:
```swift
var sectionSnapshot = NSDiffableDataSourceSectionSnapshot<Item>()
sectionSnapshot.append(["Smileys", "Nature", "Food", "Activities"])
sectionSnapshot.append(["🥃", "🍎", "🍑"], to: "Food") // children of Food
dataSource.apply(sectionSnapshot, to: .suggested)
```

Prevent a parent from collapsing:
```swift
dataSource.sectionSnapshotHandlers.shouldCollapseItem = { item in
    return item != pinnedParent  // false = prevent collapse
}
```

Enable reordering and persist to backing store:
```swift
dataSource.reorderingHandlers.canReorderItem = { item in true }
dataSource.reorderingHandlers.didReorder = { [weak self] transaction in
    if let updated = self?.backingStore.applying(transaction.difference) {
        self?.backingStore = updated
    }
}
```

## Takeaways
- Section Snapshots enable per-section composition of `UICollectionViewDiffableDataSource` and are the only way to model hierarchical outline-style data for expandable/collapsible cells in iOS 14.
- Expansion state is part of the section snapshot; the framework automatically updates and re-applies the snapshot when the user taps a disclosure accessory.
- Reordering requires supplying both `canReorderItem` and `didReorder` closures; `didReorder` receives an `NSDiffableDataSourceTransaction` whose `difference` can be applied directly to an array backing store.
- Both section snapshots and the original full-collection snapshot can coexist: use the original snapshot to manage section order, and section snapshots to manage per-section item content.

---
_Source: WWDC20 Session 10045 page (abstract, chapter summaries, code samples, and resource links)._
