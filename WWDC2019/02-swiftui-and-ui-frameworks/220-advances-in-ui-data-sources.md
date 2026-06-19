# Advances in UI Data Sources
**WWDC19 · Session 220** · [Watch](https://developer.apple.com/videos/play/wwdc2019/220/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, tvOS 13

## Overview
iOS 13 introduces `UICollectionViewDiffableDataSource` and `UITableViewDiffableDataSource` (plus `NSCollectionViewDiffableDataSource` on macOS) — a completely new data source mechanism that eliminates the error-prone `performBatchUpdates` model. Instead of manually computing index-path-based insertions, deletions, and moves, developers describe the complete desired state of a collection using an `NSDiffableDataSourceSnapshot` and call `apply(_:animatingDifferences:)`. The framework computes the diff automatically and performs the animated update.

The core insight is replacing the dual-truth problem (controller truth vs. UI truth) with a single source of truth: the snapshot. Crashes from mismatched batch updates ("invalid number of items in section") are eliminated by construction.

## Key Topics

### The Problem with Existing Data Sources
- `UICollectionViewDataSource` / `UITableViewDataSource` requires manually computing `performBatchUpdates` calls.
- Synchronization errors between the controller layer and UI layer cause the infamous `NSInternalInconsistencyException`.
- The common workaround — `reloadData()` — works but produces non-animated UI updates.

### NSDiffableDataSourceSnapshot **[NEW]**
- `NSDiffableDataSourceSnapshot<SectionIdentifierType, ItemIdentifierType>` — a value type (struct in Swift) that holds the complete desired UI state as ordered lists of section identifiers and item identifiers.
- Section and item identifier types must conform to `Hashable` — and must produce unique hash values per item to allow the framework to track identity across updates.
- Common patterns: single-section layouts use an enum with one case (automatically `Hashable`); model objects use a `UUID`-based `id` field for the hash.
- Snapshot operations:
  - `appendSections(_:)`, `appendItems(_:toSection:)` — build from scratch.
  - `insertItems(_:beforeItem:)` / `insertItems(_:afterItem:)` — insert relative to an existing identifier.
  - `moveItem(_:beforeItem:)` / `moveItem(_:afterItem:)` — reorder.
  - `deleteItems(_:)` / `deleteSections(_:)` — remove.
  - `itemIdentifiers(inSection:)`, `sectionIdentifiers`, `numberOfItems(inSection:)` — inspect.
- Ask the data source for its `snapshot()` to start from current UI state rather than empty.

### UICollectionViewDiffableDataSource / UITableViewDiffableDataSource **[NEW]**
- Initialized with the collection/table view and a cell provider closure — equivalent to `cellForItemAt indexPath:` but also receives the item identifier directly.
- Automatically registers itself as the collection/table view's data source — no additional wiring needed.
- `apply(_:animatingDifferences:completion:)` — compute diff from current state to snapshot state, then update the UI with automatic animations. Pass `animatingDifferences: false` for initial population.
- `itemIdentifier(for:)` — translate an `IndexPath` (received from delegate callbacks like `didSelectItemAt`) back to the item identifier in O(1).
- `indexPath(for:)` — translate an identifier to its current `IndexPath`.
- Never call `performBatchUpdates`, `insertItems`, `deleteItems`, etc. on the collection/table view when using a diffable data source — the framework will assert.

### Background Queue Apply **[NEW]**
- The diff computation is O(N) — linear in item count.
- For large data sets, `apply(_:animatingDifferences:)` can safely be called from a background queue.
- The framework detects the calling queue and performs the diff on that queue, then automatically switches to the main queue to apply the changes to the UI.
- Constraint: be consistent — always call `apply` from the same queue type (background or main) for a given data source; mixing queues causes assertions.

### NSCollectionViewDiffableDataSource (macOS) **[NEW]**
- API-identical to the iOS versions; available for `NSCollectionView`.
- Same snapshot type (`NSDiffableDataSourceSnapshot`) is shared across all platforms.

## APIs & Frameworks

**UIKit / AppKit** (all **[NEW]** iOS 13+, macOS 10.15+)

- `NSDiffableDataSourceSnapshot<SectionIdentifierType: Hashable, ItemIdentifierType: Hashable>` **[NEW]**
  - `appendSections(_:)`, `appendItems(_:)`, `appendItems(_:toSection:)`
  - `insertSections(_:beforeSection:)`, `insertSections(_:afterSection:)`
  - `insertItems(_:beforeItem:)`, `insertItems(_:afterItem:)`
  - `moveSection(_:beforeSection:)`, `moveSection(_:afterSection:)`
  - `moveItem(_:beforeItem:)`, `moveItem(_:afterItem:)`
  - `deleteSections(_:)`, `deleteItems(_:)`, `deleteAllItems()`
  - `reloadSections(_:)`, `reloadItems(_:)`
  - `numberOfSections: Int`, `numberOfItems: Int`, `numberOfItems(inSection:) -> Int`
  - `sectionIdentifiers: [SectionIdentifierType]`
  - `itemIdentifiers: [ItemIdentifierType]`, `itemIdentifiers(inSection:) -> [ItemIdentifierType]`
  - `sectionIdentifier(containingItem:) -> SectionIdentifierType?`
  - `indexOfItem(_:) -> Int?`, `indexOfSection(_:) -> Int?`

- `UICollectionViewDiffableDataSource<SectionIdentifierType: Hashable, ItemIdentifierType: Hashable>` **[NEW]**
  - `init(collectionView:cellProvider:)`
  - `apply(_:animatingDifferences:completion:)`
  - `snapshot() -> NSDiffableDataSourceSnapshot<...>`
  - `itemIdentifier(for:) -> ItemIdentifierType?` — O(1) IndexPath→identifier
  - `indexPath(for:) -> IndexPath?` — O(1) identifier→IndexPath

- `UITableViewDiffableDataSource<SectionIdentifierType: Hashable, ItemIdentifierType: Hashable>` **[NEW]**
  - Same interface as `UICollectionViewDiffableDataSource`; also supports:
  - `defaultRowAnimation: UITableView.RowAnimation` — controls insert/delete row animation style

- `NSCollectionViewDiffableDataSource<SectionIdentifierType: Hashable, ItemIdentifierType: Hashable>` **[NEW]** — macOS equivalent

## Code Highlights

```swift
// 1. Define identifier types (must be Hashable with unique hashes)
enum Section: Hashable { case main }

struct Mountain: Hashable {
    let identifier = UUID()  // stable unique identity
    let name: String
    let elevation: Int

    func hash(into hasher: inout Hasher) {
        hasher.combine(identifier)  // identity only — not name/elevation
    }
    static func == (lhs: Mountain, rhs: Mountain) -> Bool {
        lhs.identifier == rhs.identifier
    }
}

// 2. Create and configure the data source
class MountainViewController: UIViewController {
    var dataSource: UICollectionViewDiffableDataSource<Section, Mountain>!

    func configureDataSource() {
        dataSource = UICollectionViewDiffableDataSource<Section, Mountain>(
            collectionView: collectionView
        ) { collectionView, indexPath, mountain in
            // cell provider — mountain identifier is passed directly, no lookup needed
            let cell = collectionView.dequeueReusableCell(
                withReuseIdentifier: "MountainCell", for: indexPath)
            cell.textLabel?.text = mountain.name
            return cell
        }
    }

    // 3. Apply a snapshot to update the UI (called on search text change, model update, etc.)
    func performQuery(with filter: String?) {
        let mountains = mountainsController.filteredMountains(with: filter)

        var snapshot = NSDiffableDataSourceSnapshot<Section, Mountain>()
        snapshot.appendSections([.main])
        snapshot.appendItems(mountains)          // pass model values directly (Hashable)
        dataSource.apply(snapshot, animatingDifferences: true)
    }
}
```

```swift
// Multi-section example with heterogeneous items
enum WiFiSection: Hashable { case config, networks }

struct WiFiItem: Hashable {
    let identifier: UUID
    let network: WiFiNetwork?  // nil for config items (switch, current network)

    func hash(into hasher: inout Hasher) { hasher.combine(identifier) }
}

func updateUI(animated: Bool = true) {
    var snapshot = NSDiffableDataSourceSnapshot<WiFiSection, WiFiItem>()
    snapshot.appendSections([.config, .networks])
    snapshot.appendItems(configItems, toSection: .config)
    if wifiEnabled {
        snapshot.appendItems(networkItems, toSection: .networks)
    }
    dataSource.apply(snapshot, animatingDifferences: animated)
}
```

```swift
// Translate IndexPath back to identifier in delegate callback
func collectionView(_ collectionView: UICollectionView,
                    didSelectItemAt indexPath: IndexPath) {
    guard let mountain = dataSource.itemIdentifier(for: indexPath) else { return }
    showDetailView(for: mountain)  // use identifier directly, no array lookup
}
```

```swift
// Background queue apply for large data sets
DispatchQueue.global().async {
    var snapshot = NSDiffableDataSourceSnapshot<Section, Item>()
    snapshot.appendSections(allSections)
    snapshot.appendItems(allItems)
    // apply() detects background queue, diffs there, switches to main for UI update
    self.dataSource.apply(snapshot, animatingDifferences: true)
    // IMPORTANT: always call apply from the same queue — never mix main/background
}
```

## Takeaways
- `UICollectionViewDiffableDataSource` and `UITableViewDiffableDataSource` fully replace `performBatchUpdates` — the crash-prone synchronization problem is solved by construction: there is one truth (the snapshot), and the framework owns the diff.
- Item identifiers must be `Hashable` with unique, stable hashes that represent identity (not value) — use a `UUID` or stable database primary key, not mutable content fields.
- Passing native Swift model types (structs conforming to `Hashable`) as item identifiers eliminates the index-path-to-model lookup step in the cell provider and delegate callbacks.
- For large data sets, call `apply` from a background queue — the framework automatically diffs in the background and commits on the main queue with no additional API needed.
- The AirDrop extension in iOS 13's Share Sheet is an early internal adopter — UUID-identified Bonjour devices map directly to diffable data source items for seamless animated discovery updates.

---
_Source: WWDC19 Session 220 page (transcript, abstract, and resource links)._
