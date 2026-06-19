# Advances in UICollectionView
**WWDC20 · Session 10097** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10097/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
iOS 14 introduces significant advances across all three UICollectionView API categories — Data, Layout, and Presentation — building on the modern foundation established in iOS 13. This session provides an overview of all three areas, pointing to companion sessions for deeper dives on each.

On the **data** side, Diffable Data Source gains Section Snapshots (for per-section and hierarchical/outline data) and first-class reordering support. On the **layout** side, Compositional Layout gains Lists — a built-in way to create UITableView-like sections directly in a UICollectionView, with swipe actions and standard cell appearances. On the **presentation** side, Cell Registrations replace the `register`/`dequeue` pair, and new content configurations provide standardized, lightweight cell layout descriptors for images and text.

These three feature areas together enable building the sidebar-based, outline-expanding, list-rich UIs that characterize iPad and macOS apps in 2020.

## Key Topics
- **Section Snapshots** **[NEW]** — `NSDiffableDataSourceSectionSnapshot<Item>` for per-section data; supports `append(_:to:)` for hierarchical parent-child relationships; enables outline-style expand/collapse UIs.
- **Reordering Support** **[NEW]** — `UICollectionViewDiffableDataSource.reorderingHandlers` with `canReorderItem` and `didReorder` closures; `NSDiffableDataSourceTransaction` supplies `CollectionDifference` for backing store updates.
- **Compositional Layout Lists** **[NEW]** — `UICollectionLayoutListConfiguration` with appearance options (`.insetGrouped`, `.grouped`, `.plain`, `.sidebarPlain`, `.sidebar`); `UICollectionViewCompositionalLayout.list(using:)` creates a full list layout in two lines; also mixable per-section with other compositional layout sections.
- **`UICollectionViewListCell`** **[NEW]** — Concrete `UICollectionViewCell` subclass for list sections; supports accessories (disclosure indicator, checkmark, etc.), swipe actions, and separator management.
- **Cell Registrations** **[NEW]** — `UICollectionView.CellRegistration<CellType, ItemType>` — generic, closure-based cell setup that eliminates the `register` + reuse identifier pattern; used with `dequeueConfiguredReusableCell(using:for:item:)`.
- **Content Configurations** **[NEW]** — `UIListContentConfiguration.cell()`, `.valueCell()`, `.subtitleCell()` — lightweight value types describing cell content (image, text, secondary text) without managing layout directly; assigned to `cell.contentConfiguration`.
- **Background Configurations** **[NEW]** — `UIBackgroundConfiguration` — value types for cell background styling (color, corner radius, border, shadow); assigned to `cell.backgroundConfiguration`.
- **Sidebar appearance** — New list appearance style (`.sidebar`, `.sidebarPlain`) for iPadOS sidebar UIs; used in many system apps on iPadOS 14.

## APIs & Frameworks

### UIKit
- **`NSDiffableDataSourceSectionSnapshot<Item>`** **[NEW]** — Section-scoped snapshot; `append(_:to:)`, `expand(_:)`, `collapse(_:)`, `isExpanded(_:)`, `snapshot(of:includingParent:)`
- **`UICollectionViewDiffableDataSource.apply(_:to:animatingDifferences:completion:)`** **[NEW]** — Apply section snapshot to a section
- **`UICollectionViewDiffableDataSource.reorderingHandlers`** **[NEW]** — `ReorderingHandlers`: `canReorderItem`, `willReorder`, `didReorder`
- **`NSDiffableDataSourceTransaction`** **[NEW]** — `initialSnapshot`, `finalSnapshot`, `difference`, `sectionTransactions`
- **`UICollectionLayoutListConfiguration`** **[NEW]** — `init(appearance:)`; `UICollectionLayoutListConfiguration.Appearance`: `.plain`, `.grouped`, `.insetGrouped`, `.sidebar`, `.sidebarPlain`; `headerMode`, `footerMode`, `leadingSwipeActionsConfigurationProvider`, `trailingSwipeActionsConfigurationProvider`
- **`UICollectionViewCompositionalLayout.list(using:)`** **[NEW]** — Class method returning a list-style compositional layout
- **`UICollectionViewListCell`** **[NEW]** — `UICollectionViewCell` subclass; `accessories: [UICellAccessory]`, `separatorLayoutGuide`
- **`UICellAccessory`** **[NEW]** — `.disclosureIndicator()`, `.checkmark()`, `.outlineDisclosure()`, `.delete()`, `.insert()`, `.label(text:)`, `.customView(configuration:)`, etc.
- **`UICollectionView.CellRegistration<Cell, Item>`** **[NEW]** — `init(handler:)` where handler is `(Cell, IndexPath, Item) -> Void`; used with `dequeueConfiguredReusableCell(using:for:item:)`
- **`UICollectionView.dequeueConfiguredReusableCell(using:for:item:)`** **[NEW]** — Takes a `CellRegistration`, eliminates reuse identifier
- **`UIListContentConfiguration`** **[NEW]** — `cell()`, `valueCell()`, `subtitleCell()`, `plainHeader()`, `plainFooter()`, `groupedHeader()`, `groupedFooter()`, `sidebarCell()`, `sidebarSubtitleCell()`, `accompanyingTextItem()`; `image`, `text`, `secondaryText`, `imageProperties`, `textProperties`
- **`UIBackgroundConfiguration`** **[NEW]** — `listCell()`, `listPlainCell()`, `listGroupedCell()`, `listSidebarCell()`, `listAccompanyingTextItem()`, `clear()`; `backgroundColor`, `cornerRadius`, `strokeColor`, `strokeWidth`, `backgroundInsets`
- **`UIListContentView`** **[NEW]** — View subclass for rendering a `UIListContentConfiguration`

## Code Highlights

List layout with inset grouped appearance (2 lines):
```swift
let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
let layout = UICollectionViewCompositionalLayout.list(using: configuration)
```

Cell registration eliminates register + reuse identifier:
```swift
let reg = UICollectionView.CellRegistration<MyCell, ViewModel> { cell, indexPath, model in
    // configure cell
}
let dataSource = UICollectionViewDiffableDataSource<Section, Item>(collectionView: cv) {
    cv, indexPath, item in
    return cv.dequeueConfiguredReusableCell(using: reg, for: indexPath, item: item)
}
```

Content configuration for a standard list cell:
```swift
var config = UIListContentConfiguration.cell()
config.image = UIImage(systemName: "hammer")
config.text = "Ready. Set. Code."
cell.contentConfiguration = config
```

## Takeaways
- iOS 14 modernizes all three UICollectionView layers: Section Snapshots + Reordering (Data), Compositional Layout Lists (Layout), and Cell Registrations + Content Configurations (Presentation).
- Compositional Layout Lists can replicate UITableView behavior inside a UICollectionView in two lines, with swipe actions, accessories, and sidebar appearances included.
- Cell Registrations replace the `register` + reuse identifier pattern entirely; the closure receives a fully dequeued cell ready to configure.
- Content configurations are lightweight value types — assign one to `cell.contentConfiguration` and UIKit handles all layout and performance optimization.

---
_Source: WWDC20 Session 10097 page (abstract, chapter summaries, code samples, and resource links)._
