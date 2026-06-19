# Make Blazing Fast Lists and Collection Views
**WWDC21 · Session 10252** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10252/)

_Platforms:_ iOS 15, iPadOS 15, tvOS 15

## Overview
This session covers three pillars of high-performance `UICollectionView` and `UITableView` in iOS 15: building on solid foundations (diffable data source with stable identifiers, cell registrations created once), understanding the cell lifecycle to enable automatic cell prefetching, and a new set of image preparation APIs that move PNG/JPEG decode off the main thread. Adopting these techniques eliminates commit hitches — the most common cause of visible stuttering during scrolling.

## Key Topics

**Diffable Data Source with Stable Identifiers**
Store model identifiers (not model objects) in `NSDiffableDataSourceSnapshot`. When a model property changes, its identifier remains the same so the snapshot diff is minimal. In iOS 15, calling `apply(_:animatingDifferences: false)` no longer calls `reloadData` internally — it applies only the diff, avoiding cell recreation for unchanged items.

**Cell Registrations (Create Once)**
`UICollectionView.CellRegistration` consolidates cell configuration in one place. The collection view maintains one reuse queue per registration instance, so registrations must be created outside the cell provider and reused. Creating a new registration inside the cell provider prevents any cell reuse.

**Cell Lifecycle and Commit Hitches**
A cell's life has two phases: Preparation (dequeue from reuse pool or init, `prepareForReuse`, configuration handler runs, layout sizing) and Display (`willDisplay`, on-screen). A commit hitch occurs when the preparation phase of a new cell takes too long and the frame deadline is missed, especially on ProMotion displays (120 Hz, less time per frame).

**Automatic Cell Prefetching (iOS 15)**
Building against the iOS 15 SDK enables a new prefetching mechanism in both `UICollectionView` and `UITableView`. When a commit finishes early (no new cell needed that frame), the system uses the spare time to prepare the next cell before it is required. This effectively doubles the available preparation time without any developer code changes. Key implication for the prefetching lifecycle: a prefetched cell may never be displayed (if the user reverses scroll direction), and a cell can cycle through Display → Off-screen → Display multiple times before returning to the reuse pool.

**`NSDiffableDataSourceSnapshot.reconfigureItems(_:)` (iOS 15)**
Use `reconfigureItems` instead of `reloadItems` when updating the content of existing visible cells asynchronously (e.g., after a network image download). `reconfigureItems` reruns the cell's registration configuration handler on the existing cell object — avoiding the full dequeue/init path of `reloadItems`. Capture the item identifier (not the cell object) in async completion handlers, and call `reconfigureItems` to trigger a UI update via the data source.

**Image Preparation APIs (iOS 15)**
All images must be decoded from their compressed format (PNG, JPEG, HEIC) to raw bitmap before the render server can display them. By default this decode happens synchronously on the main thread when an image is set on an image view, causing hitches for large images. New APIs in iOS 15 allow decode to happen off-thread:
- `UIImage.prepareForDisplay(completionHandler:)` — async, runs on an internal UIKit serial queue
- `UIImage.prepareForDisplay()` — sync, callable on any background thread
- `UIImage.prepareThumbnail(of:completionHandler:)` — async, decodes and scales to a target size **[NEW]**

Prepared images hold raw pixel data and should be cached sparingly (high memory cost). Use the original compressed image for disk storage; cache only the prepared form in memory.

**Data Source Prefetching Integration**
Implement `UICollectionViewDataSourcePrefetching` (`prefetchItemsAt`/`cancelPrefetchingForItemsAt`) to kick off network downloads before cells become visible. Combine with `prepareForDisplay` in the download completion to have images fully decoded and ready before the cell is needed.

## APIs & Frameworks

- **UIKit** — `UICollectionView`, `UITableView`
- `NSDiffableDataSourceSnapshot.reconfigureItems(_ identifiers:)` **[NEW]** — rerun config handler on existing cell
- `UICollectionView.CellRegistration<Cell, Identifier>` — cell config handler; create once, reuse
- `UICollectionViewDiffableDataSource` — `apply(_:animatingDifferences:)` now diffs-only (no `reloadData`) when `animatingDifferences: false` **[UPDATED]**
- `UICollectionViewDataSourcePrefetching` — `collectionView(_:prefetchItemsAt:)`, `collectionView(_:cancelPrefetchingForItemsAt:)`
- Automatic cell prefetching in `UICollectionView` and `UITableView` **[NEW]** — enabled by building with iOS 15 SDK; no API required
- `UIImage.prepareForDisplay(completionHandler: (UIImage?) -> Void)` **[NEW]** — async off-main decode
- `UIImage.prepareForDisplay() -> UIImage?` **[NEW]** — sync decode, any thread
- `UIImage.prepareThumbnail(of size: CGSize, completionHandler: (UIImage?) -> Void)` **[NEW]** — async decode + scale
- `UICollectionViewCell.prepareForReuse()` — existing lifecycle hook

## Code Highlights

Diffable data source using stable item identifiers:
```swift
var dataSource: UICollectionViewDiffableDataSource<Section, DestinationPost.ID>

func setInitialData() {
    var snapshot = NSDiffableDataSourceSnapshot<Section, DestinationPost.ID>()
    snapshot.appendSections([.main])
    snapshot.appendItems(postStore.allPosts.map { $0.id })
    dataSource.apply(snapshot, animatingDifferences: false) // diffs-only in iOS 15
}
```

Cell registration (created once, outside provider):
```swift
let cellRegistration = UICollectionView.CellRegistration<DestinationPostCell, DestinationPost.ID> { cell, indexPath, postID in
    let post = self.postsStore.fetchByID(postID)
    let asset = self.assetsStore.fetchByID(post.assetID)
    cell.titleView.text = post.region
    cell.imageView.image = asset.image
}
```

Async update via `reconfigureItems` (correct pattern):
```swift
// In the registration handler:
if asset.isPlaceholder {
    assetsStore.downloadAsset(post.assetID) { _ in
        self.setPostNeedsUpdate(id: post.id) // triggers reconfigureItems
    }
}

// Trigger reconfiguration:
func setPostNeedsUpdate(id: DestinationPost.ID) {
    var snapshot = dataSource.snapshot()
    snapshot.reconfigureItems([id])
    dataSource.apply(snapshot, animatingDifferences: true)
}
```

Image preparation (off-main decode):
```swift
imageView.image = placeholderImage
fullImage.prepareForDisplay { preparedImage in
    DispatchQueue.main.async { self.imageView.image = preparedImage }
}
```

Thumbnail preparation (off-main decode + scale):
```swift
profileImage.prepareThumbnail(of: avatarView.bounds.size) { thumb in
    DispatchQueue.main.async { self.avatarView.image = thumb }
}
```

## Takeaways

- Simply rebuilding against the iOS 15 SDK unlocks automatic cell prefetching in both `UICollectionView` and `UITableView`, eliminating most hitches with zero code changes.
- Use `reconfigureItems` (not `reloadItems`) to update the content of existing visible cells — it reruns the registration handler on the same cell without a dequeue/init cycle.
- Move image decode off the main thread using `UIImage.prepareForDisplay(completionHandler:)` or `prepareThumbnail(of:completionHandler:)`; cache prepared images in memory (not on disk) and pair them with `UICollectionViewDataSourcePrefetching` for best results.
- Store model identifiers (not model objects) in diffable data source snapshots; `apply(_:animatingDifferences: false)` in iOS 15 is now a true diff, not a `reloadData`.

---
_Source: WWDC21 Session 10252 page (abstract, full transcript, and code samples)._
