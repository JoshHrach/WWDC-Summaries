# Meet the UIKit Button System
**WWDC21 · Session 10064** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10064/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12 (Mac Catalyst)

## Overview
iOS 15 overhauls `UIButton` with a new configuration-based API — `UIButtonConfiguration` — that provides four built-in styles (plain, gray, tinted, filled), automatic Dynamic Type support, multiline text, activity indicators, image placement control, and semantic customization via a centralized `configurationUpdateHandler`. The session also introduces two new button behaviors: **toggle buttons** (preserve selected state automatically) and **pop-up buttons** (single-selection menus). All these improvements automatically translate to native Mac Catalyst buttons without extra code.

## Key Topics

**UIButtonConfiguration — Four Built-in Styles**
- `.plain()` — classic borderless button (existing behavior)
- `.gray()` — gray background tint **[NEW]**
- `.tinted()` — app tint color for background/foreground **[NEW]**
- `.filled()` — solid fill in app tint color **[NEW]**

Assigning any `UIButtonConfiguration` to `UIButton.configuration` enables the new system. Existing `setTitle(_:for:)` / `setImage(_:for:)` calls still work and are applied on top of the configuration, making incremental adoption easy.

**Configuration Properties**
Configurations expose fine-grained layout and appearance control: `title`, `subtitle`, `image`, `imagePlacement` (.leading/.trailing/.top/.bottom), `imagePadding`, `titlePadding`, `contentInsets`, `buttonSize` (.small/.medium/.large/mini), `cornerStyle`, `baseBackgroundColor`, `baseForegroundColor`, `showsActivityIndicator`. The `background` property is a `UIBackgroundConfiguration` for precise background color, stroke, and corner radius control.

**configurationUpdateHandler**
A closure assigned to `UIButton.configurationUpdateHandler` is called whenever the button needs an update (state changes, explicit `setNeedsUpdateConfiguration()` calls). Inside, read button state, modify the configuration, and assign it back. This is the primary mechanism for state-driven button updates — replacing overrides and KVO.

`setNeedsUpdateConfiguration()` triggers the handler on demand. Call it from property `didSet` observers when non-UIButton state (e.g., a model property) changes that should affect button appearance.

**Toggle Buttons**
Setting `changesSelectionAsPrimaryAction = true` on any `UIButton` makes it a toggle button: each tap flips `isSelected`. The selected state persists. Use `UIButtonConfiguration` to customize the on/off visual states. Also works on `UIBarButtonItem` via the new `UIBarButtonItem.isSelected` property.

**Pop-Up Buttons**
Combine `showsMenuAsPrimaryAction = true` with `changesSelectionAsPrimaryAction = true` to create a pop-up button: tapping presents a `UIMenu` and the selected item's title/image become the button's title/image. Exactly one item is selected at all times. Access current selection via `button.menu?.selectedElements.first`. Set a default selection by marking one `UIAction` with `state: .on` when building the menu.

**UIMenu Enhancements**
- `UIMenu(options: .singleSelection, ...)` — makes a submenu behave as a single-selection group within a pull-down button.
- Menu items support `subtitle` for richer clarity.
- Submenu navigation improved; `selectedElements` traverses the full subtree.

**Mac Catalyst Automatic Conversion**
On Mac Catalyst, UIKit-styled buttons (including toggle and pop-up) automatically convert to native macOS controls — bezeled buttons with Mac-style pull-down and pop-up indicators. For prominent buttons that should retain iOS styling on Mac (e.g., a full-width Checkout button), set `UIButton.behaviorStyle = .pad` to override automatic conversion.

## APIs & Frameworks

### UIButtonConfiguration (UIKit) **[NEW]**
- `UIButton.Configuration.plain()` / `.gray()` / `.tinted()` / `.filled()` — factory methods **[NEW]**
- `UIButton.configuration: UIButton.Configuration?` — assign to enable new button system **[NEW]**
- `UIButton.Configuration.title: String?`
- `UIButton.Configuration.subtitle: String?` **[NEW]**
- `UIButton.Configuration.image: UIImage?`
- `UIButton.Configuration.imagePlacement: NSDirectionalRectEdge` — `.leading`, `.trailing`, `.top`, `.bottom` **[NEW]**
- `UIButton.Configuration.imagePadding: CGFloat` **[NEW]**
- `UIButton.Configuration.titlePadding: CGFloat` **[NEW]**
- `UIButton.Configuration.contentInsets: NSDirectionalEdgeInsets` **[NEW]**
- `UIButton.Configuration.buttonSize: UIButton.Configuration.Size` — `.mini`, `.small`, `.medium`, `.large` **[NEW]**
- `UIButton.Configuration.cornerStyle: UIButton.Configuration.CornerStyle` **[NEW]**
- `UIButton.Configuration.baseBackgroundColor: UIColor?` **[NEW]**
- `UIButton.Configuration.baseForegroundColor: UIColor?` **[NEW]**
- `UIButton.Configuration.showsActivityIndicator: Bool` **[NEW]**
- `UIButton.Configuration.background: UIBackgroundConfiguration` **[NEW]**
  - `UIBackgroundConfiguration.backgroundColor: UIColor?`
- `UIButton.configurationUpdateHandler: UIButton.ConfigurationUpdateHandler?` — `((UIButton) -> Void)?` **[NEW]**
- `UIButton.setNeedsUpdateConfiguration()` — trigger update from external state change **[NEW]**

### Toggle Button **[NEW]**
- `UIButton.changesSelectionAsPrimaryAction: Bool` — set to `true` for toggle behavior **[NEW]**
- `UIButton.isSelected` — tracks toggle state
- `UIBarButtonItem.isSelected: Bool` — new property for bar button toggle state **[NEW]**

### Pop-Up Button **[NEW]**
- `UIButton.showsMenuAsPrimaryAction: Bool` — present menu on tap
- `UIButton.changesSelectionAsPrimaryAction: Bool` — combined with above = pop-up button **[NEW]**
- `UIMenu.selectedElements: [UIMenuElement]` — current selection; always exactly one for pop-up **[NEW]**

### UIMenu Enhancements **[NEW]**
- `UIMenu(title:options:children:)` with `options: .singleSelection` — single-selection submenu **[NEW]**
- `UIMenuElement.subtitle: String?` — item-level subtitle **[NEW]**

### Mac Catalyst **[NEW]**
- `UIButton.behaviorStyle: UIBehaviorStyle` — `.automatic` (default, converts to Mac), `.pad` (keep iOS look) **[NEW]**

## Code Highlights

Filled button with configuration:
```swift
let signInButton = UIButton(type: .system)
signInButton.configuration = .filled()
signInButton.setTitle("Sign In", for: [])
```

Button with image, subtitle, and state-driven update handler:
```swift
var config = UIButton.Configuration.tinted()
config.title = "Add to Cart"
config.image = UIImage(systemName: "cart.badge.plus")
config.imagePlacement = .trailing
let addToCartButton = UIButton(configuration: config, primaryAction: addAction)

addToCartButton.configurationUpdateHandler = { [unowned self] button in
    var config = button.configuration
    config?.image = button.isHighlighted
        ? UIImage(systemName: "cart.fill.badge.plus")
        : UIImage(systemName: "cart.badge.plus")
    config?.subtitle = self.itemQuantityDescription
    button.configuration = config
}

private var itemQuantityDescription: String? {
    didSet { addToCartButton.setNeedsUpdateConfiguration() }
}
```

Toggle button:
```swift
let button = UIButton(primaryAction: UIAction(title: "In Stock Only") { _ in toggleStock() })
button.changesSelectionAsPrimaryAction = true
button.isSelected = showingOnlyInStock()
```

Pop-up button:
```swift
let colorClosure = { (action: UIAction) in updateColor(action.title) }
let button = UIButton(primaryAction: nil)
button.menu = UIMenu(children: [
    UIAction(title: "Bondi Blue", handler: colorClosure),
    UIAction(title: "Flower Power", state: .on, handler: colorClosure)
])
button.showsMenuAsPrimaryAction = true
button.changesSelectionAsPrimaryAction = true

// Read selection:
let selectedTitle = button.menu?.selectedElements.first?.title

// Set selection programmatically:
(button.menu?.children[selectedIndex] as? UIAction)?.state = .on
```

Single-selection submenu in a pull-down bar button:
```swift
let sortMenu = UIMenu(title: "Sort By", options: .singleSelection, children: [
    UIAction(title: "Title", handler: sortClosure),
    UIAction(title: "Date", handler: sortClosure),
])
let topMenu = UIMenu(children: [UIAction(title: "Refresh", handler: refreshClosure), sortMenu])
let barButton = UIBarButtonItem(primaryAction: nil, menu: topMenu)
```

## Takeaways
- `UIButtonConfiguration` centralizes all button appearance in one object; `configurationUpdateHandler` replaces subclasses and state observers for dynamic button updates.
- Toggle and pop-up buttons replace many uses of `UISegmentedControl`, `UISwitch`, and custom state-tracking code with simpler, semantically richer alternatives.
- All new button functionality translates automatically to native macOS controls on Mac Catalyst; use `behaviorStyle = .pad` only for deliberately prominent iOS-style buttons on Mac.
- `UIMenu.singleSelection` option makes any submenu in a pull-down behave like a pop-up menu, enabling hierarchical single-selection without managing state manually.

---
_Source: WWDC21 Session 10064 page (abstract, chapter summaries, code samples, and resource links)._
