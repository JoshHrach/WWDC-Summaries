# What's New in Mac Catalyst
**WWDC21 · Session 10052** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10052/)

_Platforms:_ macOS Monterey 12, iOS 15, iPadOS 15

## Overview
This session covers the latest UIKit and system APIs added to Mac Catalyst in macOS Monterey. The updates span four main areas: new button types (pop-up, toggle, pull-down), tooltip support, printing integration, and miscellaneous window/cursor APIs. The goal is to make Mac Catalyst apps feel increasingly native on macOS without requiring large rewrites.

The demo app "Trip Planner" illustrates each new API in context: scene subtitles that update as the user navigates, image tooltips, expansion tooltips on truncated labels, and a rich set of buttons using `UIButtonConfiguration` combined with behavioral style overrides. A responder-chain–based printing approach is shown to handle complex selection logic without forcing objects into the first responder chain artificially.

The session also highlights `UIBehavioralStyle`, which lets apps running in the Mac idiom selectively opt specific controls back into their iPad layout behavior — useful for large, panel-style toggle buttons that need to fill their frame.

## Key Topics

### New Button Types
- `changesSelectionAsPrimaryAction` **[NEW]**: boolean that causes a button's title and appearance to track its menu selection — combined with `showsMenuAsPrimaryAction = true`, produces a pop-up button
- Four button configurations: standard push button, toggle button (state tracking), pull-down menu, pop-up button (both properties true)
- Menu trigger behavior differs between idioms: Mac idiom triggers menu on long press for non-primary-action buttons; iPad idiom uses right-click

### ToolTips
- `UIToolTipInteraction(defaultToolTip:)` **[NEW]**: attach free-form tooltip text to any `UIView` via `addInteraction(_:)`
- `UIControl.toolTip` **[NEW]**: convenience property on `UIControl` for single-line tooltip assignment
- `UILabel.showsExpansionTextWhenTruncated` **[NEW]**: when `true`, hovering over a truncated label shows the full text in a floating tooltip

### Printing
- `UIApplicationSupportsPrintCommand` (Info.plist key) **[NEW]**: set to `true` to have the system automatically add "Print" and "Export as PDF" menu items; also adds "Print" to iPadOS shortcuts overlay
- `UIResponder.printContent(_:)` **[NEW]**: action method; implement on any responder to handle print; use `canPerformAction(_:withSender:)` to conditionally opt out; use `target(forAction:withSender:)` to route to the correct handler without forcing `becomeFirstResponder`

### Window and Scene Enhancements
- `UIScene.subtitle` **[NEW]**: string property displayed below the window title in the title bar; updates dynamically to reflect navigation state
- `UIApplicationSupportsTabbedSceneCollection` (Info.plist key): set to `false` to disable window tabbing and remove default tab menu items for document-based apps

### Behavioral Style
- `UIBehavioralStyle` **[NEW]**: enum (`.pad`, `.mac`) controlling whether a control uses iPad or macOS appearance/layout behavior
- `UIButton.preferredBehavioralStyle` / `UISlider.preferredBehavioralStyle` **[NEW]** (read/write): override at the control level
- `UIButton.behavioralStyle` / `UISlider.behavioralStyle` (read-only): resolved value
- Setting `.pad` on a `UIButton` in a Mac Catalyst app makes it stretch to fill its frame like on iPad

### Cursor APIs
- `UIPointerLockState` **[NEW for Mac Catalyst]**: hides and locks the cursor within the window (useful for games); temporarily unlocks on app switch and relocks on re-focus
- `UIPointerShape.beam(preferredLength:axis:)`: horizontal or vertical I-beam cursor shapes, mapping to canonical `NSCursor` shapes on macOS
- `UIPointerStyle.hidden`: hide the cursor entirely when needed

### Shortcuts / Intents on macOS
- With macOS Monterey, Shortcuts is available on Mac; custom Siri Intents from iOS apps now available on Mac
- Both in-app intent handling and Intents extensions are supported; re-enable previously disabled Intents extensions in Mac Catalyst targets

## APIs & Frameworks

- `UIKit` framework
- `UIButton.changesSelectionAsPrimaryAction` **[NEW]**
- `UIButton.showsMenuAsPrimaryAction` (existing, enhanced for pop-up button behavior)
- `UIButtonConfiguration` (`.filled()`, `.plain()`) — behavioral style interaction
- `UIButton.role` (`.primary`, `.destructive`)
- `UIButton.preferredBehavioralStyle` **[NEW]**
- `UIButton.behavioralStyle` **[NEW]** (read-only resolved)
- `UISlider.preferredBehavioralStyle` **[NEW]**
- `UISlider.behavioralStyle` **[NEW]** (read-only resolved)
- `UIBehavioralStyle` **[NEW]** (`.pad`, `.mac`)
- `UIToolTipInteraction` **[NEW]**
- `UIToolTipInteraction.init(defaultToolTip:)` **[NEW]**
- `UIToolTipInteractionDelegate` protocol
- `UIControl.toolTip` **[NEW]**
- `UILabel.showsExpansionTextWhenTruncated` **[NEW]**
- `UIScene.subtitle` **[NEW]**
- `UIResponder.printContent(_:)` **[NEW]**
- `UIResponder.canPerformAction(_:withSender:)`
- `UIResponder.target(forAction:withSender:)`
- `UIPrintInteractionController`
- `UIPointerLockState` **[NEW for Mac Catalyst]**
- `UIPointerShape.beam(preferredLength:axis:)` **[NEW for Mac Catalyst]**
- `UIPointerStyle.hidden` **[NEW for Mac Catalyst]**
- `UIApplicationSupportsPrintCommand` (Info.plist key) **[NEW]**
- `UIApplicationSupportsTabbedSceneCollection` (Info.plist key)

## Code Highlights

Pop-up button (both properties set to `true`):
```swift
let popup = UIButton(type: .system)
popup.changesSelectionAsPrimaryAction = true
popup.showsMenuAsPrimaryAction = true
popup.menu = UIMenu(children: [
    UIAction(title: "Redeem") { _ in },
    UIAction(title: "Cash Out") { _ in },
    UIAction(title: "Donate") { _ in }
])
```

Tooltip on an image view:
```swift
let interaction = UIToolTipInteraction(defaultToolTip: "A lush, deep green forest surrounds Iguaçu Falls.")
imageView.addInteraction(interaction)
```

Scene subtitle updated during navigation:
```swift
// At scene connection time or later via view.window?.windowScene
scene.subtitle = "Countries"
```

Responder chain print handling with conditional routing:
```swift
override func printContent(_ sender: Any?) {
    let printController = UIPrintInteractionController.shared
    // configure and present
}

override func canPerformAction(_ action: Selector, withSender sender: Any?) -> Bool {
    if action == #selector(printContent(_:)) {
        return shouldHandlePrinting  // business logic
    }
    return super.canPerformAction(action, withSender: sender)
}

override func target(forAction action: Selector, withSender sender: Any?) -> Any? {
    switch action {
    case #selector(UIResponder.printContent(_:)):
        return appropriatePrintHandler()
    default:
        return super.target(forAction: action, withSender: sender)
    }
}
```

iPad behavioral style on a Mac Catalyst button (fills frame like on iPad):
```swift
let button = UIButton(configuration: .filled(), primaryAction: nil)
button.configuration?.image = UIImage(systemName: "leaf")
button.preferredBehavioralStyle = .pad
button.configuration?.preferredSymbolConfigurationForImage =
    UIImage.SymbolConfiguration(pointSize: 60)
button.changesSelectionAsPrimaryAction = true
```

## Takeaways

- `UIScene.subtitle` and `UIToolTipInteraction` each require a single line of code and make a Mac Catalyst app feel immediately more native — prioritize these as low-effort, high-impact improvements.
- The four-mode button matrix (`changesSelectionAsPrimaryAction` × `showsMenuAsPrimaryAction`) maps directly to the four standard macOS button types; use `UIButtonConfiguration` to compose appearance and behavior together.
- `UIApplicationSupportsPrintCommand` in Info.plist is only half the story — still implement `printContent(_:)` in the responder chain; use `target(forAction:withSender:)` to route to the right handler without anti-pattern `becomeFirstResponder` calls.
- `UIBehavioralStyle = .pad` is a surgical override that preserves specific iPad layout behaviors (e.g., stretching button backgrounds) inside a Mac idiom app; apply it per-control rather than app-wide.

---
_Source: WWDC21 Session 10052 page (abstract, chapter summaries, code samples, and resource links)._
