# Modern cell configuration
**WWDC20 · Session 10027** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10027/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
iOS 14 introduces a declarative, composable approach to configuring `UICollectionViewCell` and `UITableViewCell` content and backgrounds. Instead of directly mutating legacy subviews (`textLabel`, `imageView`, etc.), you create lightweight value-type **configurations**, populate their properties, and assign them to the cell's `contentConfiguration` or `backgroundConfiguration` property. UIKit then handles rendering and state-driven updates efficiently.

Two primary configuration types ship in iOS 14: `UIListContentConfiguration` (image, text, secondary text layouts) and `UIBackgroundConfiguration` (fill color, visual effects, stroke, corners). Both can be asked to return updated versions of themselves for any `UICellConfigurationState`, enabling automatic or manual state-driven appearance changes.

## Key Topics

### Configurations: Core Concepts
- Configurations are **value types** — each mutation is local until assigned to the cell
- Always start with a **fresh configuration** (never try to mutate the cell's existing one); apply it in full each time
- Setting a configuration is **atomic** — UIKit diffs and updates views efficiently
- **Composable**: the same configuration code works for `UICollectionViewCell`, `UITableViewCell`, table view headers/footers, and standalone `UIView`s

### UIListContentConfiguration
- Replaces direct `imageView` / `textLabel` / `detailTextLabel` mutations (those properties are **deprecated** and will be removed)
- Properties: `image`, `text`, `secondaryText`, `imageProperties`, `textProperties`, `secondaryTextProperties`
- Layout properties: `directionalLayoutMargins`, `imageToTextPadding`, `textToSecondaryTextVerticalPadding`
- **Reserved layout size**: `imageProperties.reservedLayoutSize` — horizontally centers the image in a fixed width slot, aligning text across cells with differently sized images; UIKit applies a standard size for SF Symbols automatically
- Self-sizing: configurations are designed for flexible-height self-sizing cells; do not enforce a fixed height manually
- Special layout mode for accessibility text sizes: text wraps around the image to maximize available space — built in automatically
- Static factory methods: `.cell()`, `.subtitleCell()`, `.valueCell()`, `.sidebarCell()`, `.sidebarSubtitleCell()`, `.accompaniedSidebarCell()`, `.header()`, `.footer()`

### UIBackgroundConfiguration
- Properties: `backgroundColor`, `visualEffect` (blur), `strokeColor`, `strokeWidth`, `cornerRadius`, `customView`
- Applied automatically to list cells based on list/table style (plain, grouped, inset grouped, sidebar, sidebar plain)
- Static factory methods: `.listCell()`, `.listGroupedCell()`, `.listSidebarCell()`, `.listPlainCell()`, `.clear()`
- **Mutually exclusive** with `UIView.backgroundColor` and `backgroundView` — setting one resets the others

### Configuration State
- `UICellConfigurationState` — aggregates `UITraitCollection` + system booleans (`isHighlighted`, `isSelected`, `isDisabled`, `isFocused`, `isEditing`, `isSwiped`, `isExpanded`, drag-and-drop states) + custom key-value storage
- `UIViewConfigurationState` — same but without cell-specific states (used for headers/footers)
- `configuration.updated(for: state)` — returns a copy of the configuration with properties adjusted for that state (e.g., changed colors for selected state); original is unchanged; previously set properties remain "locked"

### updateConfiguration(using:)
- New `UICollectionViewCell` / `UITableViewCell` override **[NEW]** — called before first display and whenever state changes
- The canonical place to set both content and background configurations in a cell subclass
- Pattern: get fresh config → call `updated(for: state)` → set your content → apply customizations for state → assign to cell
- Call `setNeedsUpdateConfiguration()` to trigger a reconfiguration on demand

### Color Transformers
- `UIConfigurationColorTransformer` — a function `(UIColor) -> UIColor` applied to a configuration's color at render time
- Allows producing highlighted, grayscale, or dimmed variants from a single source color
- System configurations use preset transformers internally for their state appearances

### UIListContentView
- The rendering view backing `UIListContentConfiguration` — can be instantiated standalone (`UIListContentView(configuration:)`) and used inside a `UIStackView` or alongside custom views without being in a cell at all
- Combine with custom subviews for hybrid layouts (system list content + app-specific elements side by side)

### Custom Configurations
- Create a type conforming to `UIContentConfiguration` with a paired `UIContentView` class for fully custom cell layouts
- Custom configurations can implement `updated(for:)` to participate in the same automatic state-update pipeline

## APIs & Frameworks

- **UIKit**
  - `UIContentConfiguration` **[NEW]** — protocol; `makeContentView() -> UIView & UIContentView`, `updated(for:) -> Self`
  - `UIListContentConfiguration` **[NEW]** — system list content configuration; value type with image/text/layout properties
  - `UIListContentConfiguration.cell()` / `.subtitleCell()` / `.valueCell()` / `.sidebarCell()` / `.header()` / `.footer()` **[NEW]** — factory methods for standard styles
  - `UIListContentView` **[NEW]** — rendering view for `UIListContentConfiguration`; usable standalone
  - `UIBackgroundConfiguration` **[NEW]** — system background configuration; properties for fill, blur, stroke, corners
  - `UIBackgroundConfiguration.listCell()` / `.listGroupedCell()` / `.listSidebarCell()` / `.clear()` **[NEW]** — factory methods
  - `UIConfigurationColorTransformer` **[NEW]** — `typealias (UIColor) -> UIColor`; preset transformers available as static properties
  - `UIContentView` **[NEW]** — protocol that rendering views conform to; has `configuration: UIContentConfiguration`
  - `UIView.contentConfiguration: UIContentConfiguration?` **[NEW]** — assign to any view supporting configurations
  - `UIView.backgroundConfiguration: UIBackgroundConfiguration?` **[NEW]** — assign background configuration
  - `UICellConfigurationState` **[NEW]** — aggregated trait + state struct for cells
  - `UIViewConfigurationState` **[NEW]** — aggregated trait + state struct for non-cell views
  - `UICollectionViewCell.updateConfiguration(using:)` **[NEW]** — override to apply configurations for a given state
  - `UITableViewCell.updateConfiguration(using:)` **[NEW]** — same for table view cells
  - `UICollectionViewCell.setNeedsUpdateConfiguration()` **[NEW]** — requests a state update
  - `UICollectionViewCell.defaultContentConfiguration() -> UIListContentConfiguration` **[NEW]** — fresh config for cell's style
  - `UITableViewCell.defaultContentConfiguration() -> UIListContentConfiguration` **[NEW]** — same for table view cells

## Code Highlights

Basic content configuration (collection or table view cell):
```swift
var content = cell.defaultContentConfiguration()
content.image = UIImage(systemName: "star")
content.text = "Hello WWDC!"
cell.contentConfiguration = content
```

Customizing appearance per state in a cell subclass:
```swift
override func updateConfiguration(using state: UICellConfigurationState) {
    var content = self.defaultContentConfiguration().updated(for: state)
    content.image = self.item.icon
    content.text = self.item.title

    if state.isHighlighted || state.isSelected {
        content.imageProperties.tintColor = .white
        content.textProperties.color = .white
    }

    self.contentConfiguration = content
}
```

Requesting a specific style's default configurations:
```swift
var background = UIBackgroundConfiguration.listSidebarCell()
var content = UIListContentConfiguration.sidebarCell()
```

Using `UIListContentView` standalone:
```swift
var content = UIListContentConfiguration.cell()
// configure content...
let contentView = UIListContentView(configuration: content)
stackView.addArrangedSubview(contentView)
```

## Takeaways
- Always start with a **fresh configuration** on every configuration pass — never mutate the cell's existing configuration; UIKit handles efficient diffing internally.
- `updateConfiguration(using:)` is the single canonical place to apply configurations in a cell subclass; it fires automatically on state transitions and supports manual triggers via `setNeedsUpdateConfiguration()`.
- `UIBackgroundConfiguration` and `UIListContentConfiguration` are **mutually exclusive** with their legacy counterparts; do not mix old and new APIs on the same cell.
- `configuration.updated(for: state)` is the key method for producing state-responsive appearances without conditional branches everywhere; properties you explicitly set remain locked through state updates.
- `UIListContentView` can be used standalone in any `UIView` hierarchy — not just inside cells.

---
_Source: WWDC20 Session 10027 page (abstract, transcript, and code samples)._
