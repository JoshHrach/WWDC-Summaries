# Build for iPad
**WWDC20 · Session 10105** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10105/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
iPadOS 14 introduces redesigned system apps (Mail, Notes, Home, Shortcuts) that take full advantage of the large iPad display with multi-column layouts and sidebar navigation. This session teaches developers how to adopt the same patterns using new UIKit APIs for `UISplitViewController`, `UICollectionView` lists with sidebar styling, and the new `UICollectionView.CellRegistration` and `UIContentConfiguration` APIs.

The session presents a complete case study of the Shortcuts app redesign: replacing a tab bar with a collapsible sidebar, implementing separate regular-width and compact-width layouts within a single `UISplitViewController`, and handling state restoration across size class transitions. It also covers reducing modality by reacting to user gestures (like scrolling or drawing) to automatically dismiss transient UI.

The core recommendation is to design two experiences — regular width (multi-column) and compact width (single-column) — using a single `UISplitViewController` as the root, rather than building separate iPad and iPhone apps.

## Key Topics

**New UISplitViewController API**
A new `init(style:)` initializer accepts `.doubleColumn` or `.tripleColumn`. Columns are named: `.primary`, `.supplementary` (three-column only), `.secondary`, and `.compact`. View controllers are set per-column with `setViewController(_:for:)`. The compact column provides a fully independent view hierarchy for compact-width environments. `showDetailViewController(_:sender:)` updates the secondary column dynamically on selection.

**Display Modes and Behaviors**
`preferredSplitBehavior` controls how columns appear: `.tile` (side-by-side), `.displace` (pushes secondary), `.overlay` (columns float over secondary). `preferredDisplayMode` locks the layout (e.g., `.oneBesideSecondary` for always-split). `hideColumn(_:)` and `showColumn(_:)` trigger animated transitions. `presentsWithGesture` (default `true`) enables swipe-from-edge; `showsSecondaryOnlyButton` adds a hide-all button.

**UICollectionView Sidebar Lists**
`UICollectionLayoutListConfiguration(appearance:)` with `.sidebar` or `.sidebarPlain` appearance creates list layouts. `UICollectionViewCompositionalLayout.list(using:)` builds the layout from the configuration. `UICollectionViewDiffableDataSource` manages data. The `.sidebarPlain` appearance (white background, separators) is recommended for content lists in supplementary columns.

**Cell Registration and Content Configuration**
`UICollectionView.CellRegistration<CellType, ItemType>` replaces string-based `register(_:forCellWithReuseIdentifier:)`. `cell.defaultContentConfiguration()` returns a `UIListContentConfiguration` pre-styled for the cell's list appearance. Setting `text`, `image`, and calling `cell.contentConfiguration = content` replaces custom cell subclasses.

**Modality Reduction**
iOS 14 automatically dismisses menus without requiring a dismiss tap — touching outside the menu dismisses it and passes the touch through (e.g., for scrolling). Developers should apply the same principle: watch for incoming user events (drawing, scrolling) and use them to dismiss transient UI like popovers or color pickers.

## APIs & Frameworks

### UIKit
- `UISplitViewController(style:)` **[NEW]** — initializer with `.doubleColumn` or `.tripleColumn` style
- `UISplitViewController.Style` **[NEW]** — `.doubleColumn`, `.tripleColumn`
- `UISplitViewController.Column` **[NEW]** — `.primary`, `.supplementary`, `.secondary`, `.compact`
- `UISplitViewController.setViewController(_:for:)` **[NEW]** — assigns a view controller to a column
- `UISplitViewController.showColumn(_:)` **[NEW]** — animates a column into view
- `UISplitViewController.hideColumn(_:)` **[NEW]** — animates a column out of view
- `UISplitViewController.preferredSplitBehavior` **[NEW]** — `.tile`, `.displace`, `.overlay`
- `UISplitViewController.presentsWithGesture` — enables/disables swipe gesture (default `true`)
- `UISplitViewController.showsSecondaryOnlyButton` **[NEW]** — adds a hide-all-but-secondary button
- `UISplitViewController.preferredDisplayMode` — e.g., `.oneBesideSecondary`
- `UISplitViewController.showDetailViewController(_:sender:)` — updates secondary column
- `UICollectionLayoutListConfiguration(appearance:)` **[NEW]** — list configuration with appearance
- `UICollectionLayoutListConfiguration.Appearance` **[NEW]** — `.sidebar`, `.sidebarPlain`, `.insetGrouped`, `.grouped`, `.plain`
- `UICollectionLayoutListConfiguration.headerMode` — `.none`, `.firstItemInSection`, `.supplementary`
- `NSCollectionLayoutSection.list(using:layoutEnvironment:)` **[NEW]** — creates a list section
- `UICollectionViewCompositionalLayout.list(using:)` **[NEW]** — creates a full list layout
- `UICollectionView.CellRegistration<CellType, ItemType>` **[NEW]** — type-safe cell registration with configuration closure
- `UICollectionView.dequeueConfiguredReusableCell(using:for:item:)` **[NEW]** — dequeues using a `CellRegistration`
- `UICollectionViewListCell` **[NEW]** — list-optimized cell with content configuration support
- `UICollectionViewListCell.defaultContentConfiguration()` **[NEW]** — returns pre-styled `UIListContentConfiguration`
- `UIContentConfiguration` protocol **[NEW]** — type-safe cell content description
- `UIListContentConfiguration` **[NEW]** — content configuration for list cells (text, image, secondary text, etc.)
- `UIContentConfigurationState` **[NEW]** — state-aware configuration updates
- `UICollectionViewDiffableDataSource<SectionType, ItemType>` — data source using snapshots
- `NSDiffableDataSourceSnapshot` — snapshot for data source updates

## Code Highlights

Two-column split view setup:
```swift
let splitVC = UISplitViewController(style: .doubleColumn)
splitVC.setViewController(sidebarVC, for: .primary)
splitVC.setViewController(detailVC, for: .secondary)
splitVC.setViewController(tabBarController, for: .compact)
splitVC.preferredSplitBehavior = .tile
```

Sidebar list collection view:
```swift
let config = UICollectionLayoutListConfiguration(appearance: .sidebar)
let layout = UICollectionViewCompositionalLayout.list(using: config)
let collectionView = UICollectionView(frame: frame, collectionViewLayout: layout)
```

Cell registration with content configuration:
```swift
let reg = UICollectionView.CellRegistration<UICollectionViewListCell, MyItem> { cell, _, item in
    var content = cell.defaultContentConfiguration()
    content.text = item.title
    content.image = item.image
    cell.contentConfiguration = content
}
let dataSource = UICollectionViewDiffableDataSource<Section, MyItem>(collectionView: cv) {
    cv, indexPath, item in
    cv.dequeueConfiguredReusableCell(using: reg, for: indexPath, item: item)
}
```

## Takeaways
- Use `UISplitViewController(style:)` as the root view controller for all iPadOS apps; set a separate `.compact` column view controller to provide a tailored iPhone layout without a separate code path.
- `UICollectionLayoutListConfiguration` with `.sidebar` or `.sidebarPlain` appearance plus `CellRegistration` and `defaultContentConfiguration()` replaces `UITableView` and custom cell subclasses for sidebar navigation lists.
- Manage state across size class transitions explicitly: when switching between regular and compact hierarchies, save and restore the current detail view controller using a protocol so users land in the right place.
- Reduce modality by using incoming user events (touch, scroll, draw) to automatically dismiss transient UI like popovers and color pickers — iOS 14 touch-through menu dismissal is provided automatically.

---
_Source: WWDC20 Session 10105 page (abstract, chapter summaries, code samples, and resource links)._
