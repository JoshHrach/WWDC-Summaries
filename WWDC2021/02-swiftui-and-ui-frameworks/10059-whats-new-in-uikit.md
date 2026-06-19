# What's New in UIKit
**WWDC21 · Session 10059** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10059/)

_Platforms:_ iOS 15, iPadOS 15, Mac Catalyst

## Overview
UIKit for iOS 15 and iPadOS 15 delivers productivity improvements for iPad multitasking and keyboard navigation, major UI refinements across bars, lists, and buttons, API enhancements to collection views and diffable data source, performance upgrades for image loading, and new security and privacy features. The release also marks full unification of system colors across Apple platforms and the debut of Swift concurrency annotations on UIKit APIs.

## Key Topics

### iPad Multitasking and Windowing
`UIWindowScene.ActivationAction` enables "Open in New Window" from a context menu with a single API call. The new center-scene multitasking mode opens content in its own centered `UIWindowScene`. Apps can also move a scene to Split View or the new window shelf.

### Keyboard Navigation and Pointer
`UIFocusSystem` now backs tvOS, CarPlay, iPadOS, and Mac Catalyst — arrow keys move focus between items, Tab moves between focus groups. Band selection is enabled by default in multi-select `UICollectionView`. Pointer accessories display secondary shapes around the cursor for context. The keyboard shortcut menu is redesigned with categories and search; requires migrating from `keyCommands` to `UIMenuBuilder`.

### Drag and Drop on iPhone
Inter-app Drag and Drop now works on iPhone (previously iPad-only). No API changes required.

### UI Refinements
**Bars**: `UIToolbar` and `UITabBar` remove background material when scrolled to bottom (`scrollEdgeAppearance`). `scrollEdgeAppearance` is now on all three bar types (was only on `UINavigationBar`). `setContentScrollView(_:for:)` added to `UIViewController` to manually specify the observed scroll view.

**List headers**: New `UIListContentConfiguration` header styles — `.plain` (seamless, pins with material on scroll), `.grouped`, `.prominentInsetGrouped`, `.extraProminentInsetGrouped`.

**Sheets**: New `.medium` detent for half-height sheets; optional non-dimming behind medium detent.

**Date Picker**: Wheels reintroduced; tap time label to use keyboard input.

### UIButton.Configuration
New `UIButton.Configuration` API: `.plain()`, `.gray()`, `.tinted()`, `.filled()` styles. Supports Dynamic Type, multi-line text, image placement (`.trailing`, `.leading`, etc.), corner style, button size. Pop-up and pull-down buttons natively in UIKit. `configurationUpdateHandler` closure for state-driven configuration updates (no subclassing needed). Context menus now support collapsible submenus (no API change required).

### SF Symbols Color Rendering
New `UIImage.SymbolConfiguration(hierarchicalColor:)` for Hierarchical rendering. Palette and Multicolor rendering modes added. `UIImage.withSymbolVariant(_:)` for programmatic variant selection (`.fill`, `.circle`, `.slash`, etc.) without string manipulation.

### UIContentSizeCategory Restrictions
`UITrait UIContentSizeCategory` can now be overridden on a view hierarchy to set a floor or ceiling for Dynamic Type scaling — useful to prevent oversized decorative headers.

### System Colors Unification
`UIColor.systemMint`, `UIColor.systemBrown` now available in UIKit everywhere. `UIColor.tintColor` — runtime-resolved color matching the current tint color in the hierarchy.

### TextKit 2
`UITextField` now uses TextKit 2 internally, improving layout for complex scripts (Kannada, etc.) with no adoption required.

### UIScene State Restoration
New APIs: `UITextView`/`UITextField` transient state save/restore; `UIScene` callback for post-storyboard-load state restoration; ability to extend app launch and delay UI activation for async model code.

### Siri "Share This" and NSSharingServicePicker
New `UIActivityItemsConfiguration` on `UIScene` to represent sharable content for Siri "Share This" and Mac Catalyst `NSSharingServicePickerToolbarItem`.

### Collection View and Diffable Data Source Enhancements
`UICollectionViewCell.configurationUpdateHandler` closure — configure cell appearance inline based on `UICellConfigurationState`, no subclass needed. Diffable data source: non-animated snapshot apply no longer discards existing cells. New `reconfigureItems(_:)` on `NSDiffableDataSourceSnapshot` — efficiently updates displayed content without replacing cells.

Cell prefetching improvements give cells up to two visual frames of preparation time automatically when building for iOS 15.

### Image Performance
`UIImage.byPreparingForDisplay()` async — decodes and prepares an image off the main queue. `UIImage.byPreparingThumbnail(ofSize:)` async — efficient thumbnail generation using system knowledge of image format.

### Swift Concurrency Annotations
UIKit APIs annotated with `@MainActor` — main-queue usage now enforced at compile time. Async-safe variants of image APIs added for use with Swift concurrency.

### Security and Privacy
**Location Button**: `CLLocationButton` — grants one-time current location access on tap, no prompts. Appearance is flexible but system-enforced for legibility. **Paste Privacy**: Paste banner suppressed when user uses standard paste UI (edit menu Paste button, Cmd-V). New standard paste menu items: `UIPasteControl`, selectors for `.paste`, `.pasteAndGo`, `.pasteAndSearch`, `.pasteAndMatchStyle`. Expanded pasteboard detection API covers all Data Detectors types without showing the notice. **UIEventAttribution** (iOS 14.5+): `UIEventAttributionView`, `UIEventAttribution` for privacy-preserving App-to-Web click measurement (PCM).

## APIs & Frameworks

- `UIWindowScene.ActivationAction` **[NEW]** — "Open in New Window" context menu action
- `UIWindowScene.ActivationConfiguration` **[NEW]** — configuration for a new scene activation
- `UIFocusSystem` — unified focus engine across tvOS/CarPlay/iPadOS/Mac Catalyst **[NEW on iPad]**
- `UIPointerAccessory` **[NEW]** — secondary shapes displayed around the pointer
- `UIMenuBuilder` — required for categorized keyboard shortcut menu
- `UITabBar.scrollEdgeAppearance` **[NEW on UITabBar]**
- `UIToolbar.scrollEdgeAppearance` **[NEW on UIToolbar]**
- `UIViewController.setContentScrollView(_:for:)` **[NEW]**
- `UIListContentConfiguration` header styles: `.prominentInsetGrouped`, `.extraProminentInsetGrouped` **[NEW]**
- `UISheetPresentationController.Detent.medium()` **[NEW]** — half-height sheet detent
- `UIButton.Configuration` **[NEW]** — `.plain()`, `.gray()`, `.tinted()`, `.filled()`
- `UIButton.configurationUpdateHandler` **[NEW]** — closure-based state-driven button configuration
- `UIImage.SymbolConfiguration(hierarchicalColor:)` **[NEW]** — Hierarchical SF Symbol rendering
- `UIImage.withSymbolVariant(_:)` **[NEW]** — programmatic symbol variant selection
- `UIImage.byPreparingForDisplay()` **[NEW]** — async image decode preparation
- `UIImage.byPreparingThumbnail(ofSize:)` **[NEW]** — async efficient thumbnail generation
- `UIColor.systemMint` **[NEW in UIKit]** — previously only other frameworks
- `UIColor.systemBrown` **[NEW in UIKit]**
- `UIColor.tintColor` **[NEW]** — runtime-resolved tint color
- `UICellConfigurationState` — used with `configurationUpdateHandler`
- `UICollectionViewCell.configurationUpdateHandler` **[NEW]** — closure for state-based cell config
- `NSDiffableDataSourceSnapshot.reconfigureItems(_:)` **[NEW]** — update cell content without replacing
- `CLLocationButton` **[NEW]** — one-time location access button
- `UIPasteControl` **[NEW]** — standard paste button that suppresses pasteboard notice
- `UIEventAttributionView` / `UIEventAttribution` — App-to-Web click measurement (PCM)
- `@MainActor` annotations on UIKit — compile-time main-queue enforcement

## Code Highlights

Open in New Window action:
```swift
let newSceneAction = UIWindowScene.ActivationAction { _ in
    let activity = NSUserActivity(activityType: "com.myapp.detailscene")
    return UIWindowScene.ActivationConfiguration(userActivity: activity)
}
```

UIButton.Configuration:
```swift
var config = UIButton.Configuration.tinted()
config.title = "Add to Cart"
config.image = UIImage(systemName: "cart.badge.plus")
config.imagePlacement = .trailing
config.buttonSize = .large
config.cornerStyle = .capsule
let button = UIButton(configuration: config)
```

Async image preparation:
```swift
Task {
    let prepared = await UIImage(contentsOfFile: path)?.byPreparingForDisplay()
    imageView.image = prepared
}
```

Cell configuration update handler:
```swift
cell.configurationUpdateHandler = { cell, state in
    var content = UIListContentConfiguration.cell().updated(for: state)
    content.text = "Hello"
    if state.isDisabled { content.textProperties.color = .systemGray }
    cell.contentConfiguration = content
}
```

## Takeaways

- `UIButton.Configuration` replaces fragile button subclassing with a composable, state-driven API that supports Dynamic Type, multi-line, and pop-up/pull-down menus natively.
- Async image APIs (`byPreparingForDisplay`, `byPreparingThumbnail`) and cell prefetching improvements directly address the most common scroll-performance regressions in UIKit apps.
- The Location Button and expanded paste privacy APIs reduce friction for users while giving developers clear, well-defined paths for privacy-preserving features.
- UIFocusSystem unification means iPadOS apps gain full keyboard navigation with minimal code when using standard UIKit components.

---
_Source: WWDC21 Session 10059 page (abstract, chapter summaries, code samples, and resource links)._
