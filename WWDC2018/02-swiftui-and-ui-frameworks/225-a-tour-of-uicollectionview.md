# A Tour of UICollectionView
**WWDC18 · Session 225** · [Watch](https://developer.apple.com/videos/play/wwdc2018/225/)

_Platforms:_ iOS 12, tvOS 12

## Overview
UICollectionView is a flexible, powerful view component that abstracts visual layout from content. This session provides an end-to-end walkthrough of building a real app, covering layouts (both flow-based and custom), data sources, delegates, and animated batch updates.

The session demonstrates two key design patterns: using `UICollectionViewFlowLayout` with customization for a multi-column friends list, and building a fully custom `UICollectionViewLayout` subclass for a mosaic photo feed. Performance optimization of custom layouts is also covered, replacing a linear O(n) filter with a binary search in `layoutAttributesForElements(in:)`.

Animated batch updates are explored in depth, including how to correctly sequence data source mutations alongside CollectionView update calls to avoid assertion failures. The session provides concrete rules for decomposing moves and ordering deletes/inserts to keep data source and UI in sync.

## Key Topics

### Three Core Concepts
- **Layout** — Abstracts visual arrangement from content; defined by `UICollectionViewLayout` and subclasses. Immutable during scrolling; changed via invalidation.
- **Data Source** — Provides content (what is displayed): number of sections, items per section, cell for index path.
- **Delegate** — Optional; extends `UIScrollViewDelegate` with highlighting, selection, and visibility callbacks.

### UICollectionViewFlowLayout
- Line-based layout system; automatically wraps items to next line when a line fills.
- Key properties: `minimumLineSpacing`, `minimumInteritemSpacing`, `itemSize`, `sectionInset`, `sectionInsetReference`.
- Customized in the `prepare()` override, which is called on every invalidation (rotation, resize).
- Multi-column adaptation calculated from available width and minimum column width.

### Custom UICollectionViewLayout
- Four required override points: `collectionViewContentSize`, `layoutAttributesForElements(in:)`, `layoutAttributesForItem(at:)`, `prepare()`.
- `shouldInvalidateLayout(forBoundsChange:)` — called on every bounds change; return `true` only when bounds size changes, not on scroll origin changes.
- Cache layout attributes in `prepare()` to avoid recomputing in high-frequency methods.
- Binary search optimization for sorted attribute arrays to make `layoutAttributesForElements(in:)` O(log n).

### Batch Updates and Animations
- `performBatchUpdates(_:completion:)` — apply multiple insert/delete/move/reload operations simultaneously with animation.
- Data source mutations and CollectionView update calls must both occur inside the batch updates closure.
- Index path semantics: delete/reload index paths refer to the **before** state; insert index paths refer to the **after** state; move has both.
- `reload` decomposes into a delete + insert; conflicts arise if a reload and a move share the same source index path.
- Safe ordering rules: decompose moves into delete+insert pairs; process all deletes in **descending** index path order; process all inserts in **ascending** index path order.

## APIs & Frameworks

**UIKit — UICollectionView**
- `UICollectionView` **[core]**
- `UICollectionViewDataSource` protocol — `numberOfSections(in:)`, `collectionView(_:numberOfItemsInSection:)`, `collectionView(_:cellForItemAt:)`
- `UICollectionViewDelegate` protocol — `collectionView(_:didSelectItemAt:)`, `collectionView(_:willDisplay:forItemAt:)`, `collectionView(_:didEndDisplaying:forItemAt:)`
- `UICollectionView.performBatchUpdates(_:completion:)`
- `UICollectionView.reloadItems(at:)`
- `UICollectionView.deleteItems(at:)`
- `UICollectionView.insertItems(at:)`
- `UICollectionView.moveItem(at:to:)`
- `UICollectionView.reloadData()`
- `UICollectionView.dequeueReusableCell(withReuseIdentifier:for:)`
- `UICollectionView.register(_:forCellWithReuseIdentifier:)`

**UIKit — UICollectionViewLayout**
- `UICollectionViewLayout` (abstract base class)
- `UICollectionViewLayoutAttributes` — `frame`, `bounds`, `center`, `transform`, `alpha`, `zIndex`
- `UICollectionViewLayout.prepare()`
- `UICollectionViewLayout.collectionViewContentSize` (computed property)
- `UICollectionViewLayout.layoutAttributesForElements(in:)` → `[UICollectionViewLayoutAttributes]?`
- `UICollectionViewLayout.layoutAttributesForItem(at:)` → `UICollectionViewLayoutAttributes?`
- `UICollectionViewLayout.shouldInvalidateLayout(forBoundsChange:)` → `Bool`
- `UICollectionViewLayout.invalidateLayout()`
- `UICollectionViewLayout.invalidationContext(forBoundsChange:)`

**UIKit — UICollectionViewFlowLayout**
- `UICollectionViewFlowLayout` (concrete subclass of `UICollectionViewLayout`)
- `UICollectionViewFlowLayout.itemSize`
- `UICollectionViewFlowLayout.minimumLineSpacing`
- `UICollectionViewFlowLayout.minimumInteritemSpacing`
- `UICollectionViewFlowLayout.sectionInset`
- `UICollectionViewFlowLayout.sectionInsetReference` — `.fromSafeArea`
- `UICollectionViewDelegateFlowLayout` protocol

**UIKit — UIScrollView (inherited)**
- `UIScrollView.contentSize`
- `UIScrollView.contentInset`
- `UIView.autoresizingMask`

## Code Highlights

Adaptive item sizing in a `UICollectionViewFlowLayout` subclass `prepare()`:
```swift
override func prepare() {
    super.prepare()
    guard let cv = collectionView else { return }
    let availableWidth = cv.bounds.inset(by: cv.layoutMargins).width
    let minColumnWidth: CGFloat = 300
    let maxNumColumns = Int(availableWidth / minColumnWidth)
    let cellWidth = (availableWidth / CGFloat(maxNumColumns)).rounded(.down)
    itemSize = CGSize(width: cellWidth, height: 70)
    sectionInset = UIEdgeInsets(top: minimumInteritemSpacing, left: 0, bottom: 0, right: 0)
    sectionInsetReference = .fromSafeArea
}
```

Correct batch update sequencing (decompose move, process deletes descending, inserts ascending):
```swift
UIView.performWithoutAnimation {
    collectionView.performBatchUpdates({
        // Reload (non-animated)
        people[lastIndex] = updatedPerson
        collectionView.reloadItems(at: [IndexPath(item: lastIndex, section: 0)])
    })
}
collectionView.performBatchUpdates({
    // Delete at index 3, delete at index 2 (descending)
    let movedPerson = people.remove(at: 3)
    people.remove(at: 2)
    // Insert at index 0 (ascending)
    people.insert(movedPerson, at: 0)

    collectionView.deleteItems(at: [IndexPath(item: 3, section: 0)])
    collectionView.deleteItems(at: [IndexPath(item: 2, section: 0)])
    collectionView.moveItem(at: IndexPath(item: 3, section: 0),
                            to: IndexPath(item: 0, section: 0))
})
```

## Takeaways
- Start with `UICollectionViewFlowLayout` and subclass it for simple customization; drop to a full `UICollectionViewLayout` subclass only when the design is fundamentally not line-based.
- Cache layout attributes in `prepare()` and use binary search in `layoutAttributesForElements(in:)` to maintain smooth scrolling performance at scale.
- Always apply data source mutations inside `performBatchUpdates` alongside CollectionView calls; mismatched index paths cause runtime assertion failures.
- Decompose reloads and moves before batch-updating to avoid index-path conflicts; process deletes descending and inserts ascending.

---
_Source: WWDC18 Session 225 page (abstract, full transcript, and resource links)._
