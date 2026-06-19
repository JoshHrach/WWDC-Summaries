# Advances in Collection View Layout
**WWDC19 · Session 215** · [Watch](https://developer.apple.com/videos/play/wwdc2019/215/)

_Platforms:_ iOS 13, iPadOS 13, tvOS 13, macOS Catalina 10.15 (Mac Catalyst)

## Overview
iOS 13 introduces `UICollectionViewCompositionalLayout` — a completely new approach to collection view layout that replaces the complex, delegation-heavy `UICollectionViewFlowLayout` subclassing model with a declarative, composable API. The new layout is built around four core types arranged in a hierarchy: `NSCollectionLayoutItem`, `NSCollectionLayoutGroup`, `NSCollectionLayoutSection`, and `UICollectionViewCompositionalLayout`. Each layer composes the layers below it, enabling sophisticated layouts — including orthogonal-scrolling sections, pinned headers, badge decorations, and per-section scroll behavior — with far less code than was previously possible.

The session motivates the new API by walking through real-world layouts drawn from App Store, App Store Today tab, and Music, demonstrating how layouts that previously required hundreds of lines and significant subclassing can be expressed in ~20 lines of compositional layout code. A key design principle is size expressibility: dimensions are expressed as fractional proportions of the container, absolute points, or estimated values — not as computed pixel values in delegate methods.

## Key Topics

### The Compositional Layout Hierarchy
- **`NSCollectionLayoutSize`** — specifies width and height independently using `NSCollectionLayoutDimension`.
  - `.fractionalWidth(_:)` — fraction of the container's width
  - `.fractionalHeight(_:)` — fraction of the container's height
  - `.absolute(_:)` — fixed point value
  - `.estimated(_:)` — self-sizing; layout starts with the estimate and adjusts
- **`NSCollectionLayoutItem`** — a single cell, described by its size. Optional `contentInsets` and `edgeSpacing`.
- **`NSCollectionLayoutGroup`** — arranges items:
  - `.horizontal(layoutSize:subitems:)` — items laid out left-to-right
  - `.vertical(layoutSize:subitems:)` — items stacked top-to-bottom
  - `.custom(layoutSize:itemProvider:)` — fully custom placement closure
  - Groups can be nested: a group can contain other groups as subitems.
- **`NSCollectionLayoutSection`** — wraps a group; controls orthogonal scrolling, spacing, supplementary items, decoration items, and content insets.
- **`UICollectionViewCompositionalLayout`** — top-level layout object; initialized with a section provider closure `(Int, NSCollectionLayoutEnvironment) -> NSCollectionLayoutSection?` for per-section layouts, or a single fixed section.

### Supplementary Items & Decoration Items
- **`NSCollectionLayoutBoundarySupplementaryItem`** — headers and footers for sections; can be pinned to the visible bounds (`pinToVisibleBounds = true`) for sticky headers.
- **`NSCollectionLayoutSupplementaryItem`** — anchored to an item or group (e.g., badges); positioned via `NSCollectionLayoutAnchor` (edges, fractional offsets).
- **`NSCollectionLayoutDecorationItem`** — background decoration behind a section's content; registered on the layout object itself (`layout.register(_:forDecorationViewOfKind:)`).

### Orthogonal Scrolling Sections **[NEW]**
- `NSCollectionLayoutSection.orthogonalScrollingBehavior` — enables horizontal scrolling within a vertically scrolling collection view (or vice versa), section by section.
- Scroll behaviors: `.continuous`, `.continuousGroupLeadingBoundary`, `.paging`, `.groupPaging`, `.groupPagingCentered`, `.none`.
- `NSCollectionLayoutSection.visibleItemsInvalidationHandler` — closure called during orthogonal scroll for per-frame custom effects (e.g., parallax, scale transforms).

### Environment-Aware Layouts
- The section provider closure receives `NSCollectionLayoutEnvironment` which exposes `container.effectiveContentSize` — the collection view's current size accounting for safe area and trait changes.
- Layouts automatically adapt when the device rotates or the collection view changes size without any additional code.

### Explicit Layout Spacing
- `NSCollectionLayoutGroup.interItemSpacing` — spacing between items in a group: `.fixed(_:)` or `.flexible(_:)` (minimum spacing, expands to fill).
- `NSCollectionLayoutSection.interGroupSpacing` — spacing between groups in a section.
- `NSCollectionLayoutItem.contentInsets` — insets applied per item (replaces minimumInteritemSpacing / minimumLineSpacing delegation).
- `NSCollectionLayoutSection.contentInsets` — insets applied to the entire section.

## APIs & Frameworks

**UIKit — Compositional Layout** (all **[NEW]** iOS 13+)
- `UICollectionViewCompositionalLayout` **[NEW]** — `init(section:)`, `init(sectionProvider:)`, `init(sectionProvider:configuration:)`
- `UICollectionViewCompositionalLayoutSectionProvider` **[NEW]** — `(Int, NSCollectionLayoutEnvironment) -> NSCollectionLayoutSection?`
- `UICollectionViewCompositionalLayoutConfiguration` **[NEW]** — `scrollDirection`, `interSectionSpacing`, `boundarySupplementaryItems`
- `NSCollectionLayoutEnvironment` **[NEW]** — `container: NSCollectionLayoutContainer`
- `NSCollectionLayoutContainer` **[NEW]** — `contentSize`, `effectiveContentSize`
- `NSCollectionLayoutDimension` **[NEW]** — `.fractionalWidth(_:)`, `.fractionalHeight(_:)`, `.absolute(_:)`, `.estimated(_:)`
- `NSCollectionLayoutSize` **[NEW]** — `init(widthDimension:heightDimension:)`
- `NSCollectionLayoutItem` **[NEW]** — `init(layoutSize:)`, `init(layoutSize:supplementaryItems:)`, `contentInsets`, `edgeSpacing`
- `NSCollectionLayoutGroup` **[NEW]** — `.horizontal(layoutSize:subitems:)`, `.vertical(layoutSize:subitems:)`, `.custom(layoutSize:itemProvider:)`, `interItemSpacing`, `supplementaryItems`, `edgeSpacing`
- `NSCollectionLayoutSection` **[NEW]** — `init(group:)`, `orthogonalScrollingBehavior`, `interGroupSpacing`, `contentInsets`, `boundarySupplementaryItems`, `decorationItems`, `visibleItemsInvalidationHandler`
- `UICollectionLayoutSectionOrthogonalScrollingBehavior` **[NEW]** — `.none`, `.continuous`, `.continuousGroupLeadingBoundary`, `.paging`, `.groupPaging`, `.groupPagingCentered`
- `NSCollectionLayoutBoundarySupplementaryItem` **[NEW]** — `init(layoutSize:elementKind:alignment:)`, `pinToVisibleBounds`, `extendsBoundary`
- `NSCollectionLayoutSupplementaryItem` **[NEW]** — `init(layoutSize:elementKind:containerAnchor:)`, `init(layoutSize:elementKind:containerAnchor:itemAnchor:)`
- `NSCollectionLayoutDecorationItem` **[NEW]** — `.background(elementKind:)`, `contentInsets`, `zIndex`
- `NSCollectionLayoutAnchor` **[NEW]** — `.init(edges:)`, `.init(edges:absoluteOffset:)`, `.init(edges:fractionalOffset:)`
- `NSCollectionLayoutGroupCustomItem` **[NEW]** — `init(frame:)` — used in custom group item provider closure
- `NSCollectionLayoutEdgeSpacing` **[NEW]** — `init(leading:top:trailing:bottom:)` with `.fixed(_:)` / `.flexible(_:)`
- `NSCollectionLayoutSpacing` **[NEW]** — `.fixed(_:)`, `.flexible(_:)`
- `NSCollectionLayoutVisibleItem` **[NEW]** — protocol exposing frame, alpha, transform, zIndex for use in `visibleItemsInvalidationHandler`

## Code Highlights

```swift
// Simple 3-column grid
func createLayout() -> UICollectionViewLayout {
    let itemSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1.0/3.0),
        heightDimension: .fractionalHeight(1.0))
    let item = NSCollectionLayoutItem(layoutSize: itemSize)
    item.contentInsets = NSDirectionalEdgeInsets(top: 5, leading: 5, bottom: 5, trailing: 5)

    let groupSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1.0),
        heightDimension: .fractionalWidth(1.0/3.0))
    let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize,
                                                   subitems: [item])

    let section = NSCollectionLayoutSection(group: group)
    return UICollectionViewCompositionalLayout(section: section)
}

// Per-section layout with orthogonal scrolling
func createMultiSectionLayout() -> UICollectionViewLayout {
    return UICollectionViewCompositionalLayout { sectionIndex, environment in
        if sectionIndex == 0 {
            // Horizontally scrolling section
            let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.4),
                                                  heightDimension: .fractionalHeight(1.0))
            let item = NSCollectionLayoutItem(layoutSize: itemSize)
            let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.85),
                                                   heightDimension: .absolute(200))
            let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize,
                                                           subitems: [item])
            let section = NSCollectionLayoutSection(group: group)
            section.orthogonalScrollingBehavior = .groupPagingCentered
            return section
        } else {
            // Standard vertical list section
            let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                                  heightDimension: .estimated(44))
            let item = NSCollectionLayoutItem(layoutSize: itemSize)
            let group = NSCollectionLayoutGroup.vertical(
                layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                                   heightDimension: .estimated(44)),
                subitems: [item])
            return NSCollectionLayoutSection(group: group)
        }
    }
}

// Pinned section header
let headerSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                        heightDimension: .estimated(44))
let header = NSCollectionLayoutBoundarySupplementaryItem(
    layoutSize: headerSize,
    elementKind: UICollectionView.elementKindSectionHeader,
    alignment: .top)
header.pinToVisibleBounds = true
section.boundarySupplementaryItems = [header]
```

## Takeaways
- `UICollectionViewCompositionalLayout` replaces bespoke `UICollectionViewLayout` subclasses and complex `UICollectionViewFlowLayout` delegation for the vast majority of collection view use cases.
- The fractional dimension system (`fractionalWidth`, `fractionalHeight`) means layouts automatically adapt to all device sizes and orientations with zero extra code.
- Orthogonal scrolling sections (`orthogonalScrollingBehavior`) are a first-class feature — implementing horizontal carousels inside a vertical scroll view no longer requires nested scroll views or custom hit testing.
- Nested groups allow arbitrarily complex item arrangements within a single section without subclassing.

---
_Source: WWDC19 Session 215 page (abstract, chapter summaries, code samples, and resource links)._
