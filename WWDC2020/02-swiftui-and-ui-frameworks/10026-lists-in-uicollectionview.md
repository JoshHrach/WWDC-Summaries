# Lists in UICollectionView
**WWDC20 · Session 10026** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10026/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
iOS 14 brings UITableView-like list appearances directly to `UICollectionView` through a new `UICollectionLayoutListConfiguration` type. Built on top of the Compositional Layout system introduced in iOS 13, lists in `UICollectionView` are highly flexible: individual sections of a collection view can use different layouts (lists, grids, orthogonally scrolling sections) all within a single `UICollectionView` instance. Self-sizing is now the default for list sections.

The session also introduces `UICollectionViewListCell`, a new cell subclass that brings several table-view features — swipe actions, accessories, separator layout guides — into the collection view world with a more capable and declarative API. Two new sidebar appearances enable modern multi-column app designs on iPadOS 14.

## Key Topics

### UICollectionLayoutListConfiguration
- New type that wraps Compositional Layout to produce UITableView-like list sections
- Appearances: `.plain`, `.grouped`, `.insetGrouped` (familiar from UITableView), and new `.sidebar` / `.sidebarPlain` (for iPadOS 14 multi-column layouts)
- Controls separator visibility and header/footer configuration per section

### Simple vs. Per-Section Setup
- **Simple setup**: `UICollectionViewCompositionalLayout.list(using:)` — entire collection view uses a single list configuration
- **Per-section setup**: `NSCollectionLayoutSection.list(using:layoutEnvironment:)` inside the section-provider initializer — each section can independently use a list or any custom Compositional Layout

### Headers and Footers
- Must be explicitly enabled (unlike UITableView)
- **`.supplementary` mode**: registers a supplementary view; provide it via `supplementaryViewProvider` on the diffable data source or the UICollectionView delegate; returning `nil` will assert
- **`.firstItemInSection` mode** (headers only): the first data item in the section is rendered as a header cell; ideal for hierarchical data with section snapshots from Diffable Data Source

### Self-Sizing
- Self-sizing is the new default for list sections — no need to manually calculate cell heights
- Build cells with Auto Layout; collection view resolves sizes automatically
- Override `preferredLayoutAttributesFittingAttributes(_:)` on cell subclasses if manual sizing is needed

### UICollectionViewListCell
- New `UICollectionViewCell` subclass for use with list sections (can also be used in any section; any cell type can be used in list sections)
- Works seamlessly with content configurations and background configurations (see "Modern Cell Configuration")

### Separator Layout Guide
- Replaces the point-based `separatorInset` from UITableView with a layout-guide approach
- Constrain `separatorLayoutGuide.leadingAnchor` to the primary content's leading anchor; the system keeps the separator aligned automatically
- When using system-provided content configurations, this is handled automatically

### Swipe Actions
- Moved from UITableView data source to the **cell** itself — configured alongside cell content
- Set `cell.leadingSwipeActionsConfiguration` and/or `cell.trailingSwipeActionsConfiguration`
- Only supported when the cell is inside a list section (requires layout-cell communication)
- Override the configuration getter to create configs lazily (called only when a swipe begins)
- **Important**: never capture the index path in action handlers — capture the data model or a stable identifier instead

### Accessories
- Declarative array-based API: `cell.accessories = [...]`
- System automatically sorts accessories to correct sides and manages edit-mode visibility
- Built-in accessories: `.disclosureIndicator()`, `.delete()`, `.reorder()`, `.checkmark()`, `.detail()`, and the new `.outlineDisclosure()` **[NEW]**
- `.outlineDisclosure()` communicates with the diffable data source section snapshot API to expand/collapse children
- `displayed:` parameter controls when each accessory is visible (`.always`, `.whenEditing`, `.whenNotEditing`)
- Accessories can trigger functionality (e.g., `.reorder()` enters reorder mode; `.delete()` reveals trailing swipe actions)

## APIs & Frameworks

- **UIKit**
  - `UICollectionLayoutListConfiguration` **[NEW]** — configures a list section; `appearance:` sets the style
  - `UICollectionLayoutListConfiguration.Appearance` **[NEW]** — `.plain`, `.grouped`, `.insetGrouped`, `.sidebar`, `.sidebarPlain`
  - `UICollectionLayoutListConfiguration.headerMode` **[NEW]** — `.none`, `.supplementary`, `.firstItemInSection`
  - `UICollectionLayoutListConfiguration.footerMode` **[NEW]** — `.none`, `.supplementary`
  - `UICollectionLayoutListConfiguration.showsSeparators` **[NEW]** — controls separator visibility
  - `UICollectionViewCompositionalLayout.list(using:)` **[NEW]** — creates a full-collection-view list layout from a configuration
  - `NSCollectionLayoutSection.list(using:layoutEnvironment:)` **[NEW]** — creates a single list section for use in the section-provider initializer
  - `UICollectionViewListCell` **[NEW]** — `UICollectionViewCell` subclass for list sections
  - `UICollectionViewListCell.separatorLayoutGuide` **[NEW]** — layout guide for aligning separators to primary content
  - `UICollectionViewListCell.leadingSwipeActionsConfiguration` **[NEW]** — swipe actions property on the cell
  - `UICollectionViewListCell.trailingSwipeActionsConfiguration` **[NEW]** — swipe actions property on the cell
  - `UICollectionViewListCell.accessories` **[NEW]** — array of `UICellAccessory` values
  - `UICellAccessory` **[NEW]** — value type for cell accessories
  - `UICellAccessory.disclosureIndicator(displayed:options:)` **[NEW]**
  - `UICellAccessory.delete(displayed:options:actionHandler:)` **[NEW]**
  - `UICellAccessory.reorder(displayed:options:)` **[NEW]**
  - `UICellAccessory.checkmark(displayed:options:)` **[NEW]**
  - `UICellAccessory.detail(displayed:options:actionHandler:)` **[NEW]**
  - `UICellAccessory.outlineDisclosure(displayed:options:actionHandler:)` **[NEW]**
  - `UICellAccessory.DisplayedState` **[NEW]** — `.always`, `.whenEditing`, `.whenNotEditing`
  - `UICollectionView.CellRegistration` **[NEW in iOS 14]** — type-safe cell registration (companion API)
  - `UISwipeActionsConfiguration` — unchanged; used for configuring swipe action sets
  - `UIContextualAction` — unchanged; individual swipe action

## Code Highlights

Simple full-collection-view list:
```swift
let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
let layout = UICollectionViewCompositionalLayout.list(using: configuration)
```

Per-section setup mixing a custom grid and a list:
```swift
let layout = UICollectionViewCompositionalLayout { [weak self] sectionIndex, layoutEnvironment in
    guard let self else { return nil }
    if sectionIndex == 0 {
        return self.makeGridSection() // custom Compositional Layout section
    }
    let config = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
    return NSCollectionLayoutSection.list(using: config, layoutEnvironment: layoutEnvironment)
}
```

Configuring swipe actions and accessories on a list cell:
```swift
let reg = UICollectionView.CellRegistration<UICollectionViewListCell, Model> { cell, _, item in
    let markFavorite = UIContextualAction(style: .normal, title: "Favorite") {
        [weak self] _, _, completion in
        self?.markFavorite(with: item.identifier) // use stable ID, not index path
        completion(true)
    }
    cell.leadingSwipeActionsConfiguration = UISwipeActionsConfiguration(actions: [markFavorite])
    cell.accessories = [
        .disclosureIndicator(displayed: .whenNotEditing),
        .delete()
    ]
}
```

Separator layout guide alignment:
```swift
// After configuring your cell's layout with Auto Layout:
cell.separatorLayoutGuide.leadingAnchor.constraint(
    equalTo: primaryLabel.leadingAnchor
).isActive = true
```

## Takeaways
- `UICollectionLayoutListConfiguration` is the single new type needed to achieve UITableView-like list appearances inside `UICollectionView`, with the full power of Compositional Layout for per-section customization and mixed layouts.
- Swipe actions and accessories are now configured on `UICollectionViewListCell` directly, not through data source callbacks; the declarative accessories API handles edit-mode visibility and side placement automatically.
- Never capture index paths in swipe action handlers — index paths are not stable identifiers; capture the model object or a stable ID instead.
- `.sidebar` and `.sidebarPlain` appearances are new in iOS 14 and designed specifically for iPadOS 14 multi-column sidebar interfaces.

---
_Source: WWDC20 Session 10026 page (abstract, transcript, code samples, and resource links)._
