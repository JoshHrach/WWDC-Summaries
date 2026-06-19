# Accessibility Design for Mac Catalyst
**WWDC20 · Session 10117** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10117/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11 (Mac Catalyst)

## Overview
When you bring an iOS app to the Mac using Mac Catalyst, Apple's accessibility team automatically converts UIKit accessibility APIs to their macOS equivalents — meaning a well-accessible iOS app is automatically a well-accessible Mac app. This session focuses on three areas developers should address to go beyond the defaults: keyboard focus and shortcuts, accessible navigation using container groups, and testing with Accessibility Inspector improvements.

On macOS, the keyboard is the primary input method rather than a supplement, so keyboard focus and shortcuts must be a first-class concern. UIKit already makes standard controls keyboard-accessible using tab navigation and arrow keys, but custom elements may require additional work. The new `selectionFollowsFocus` property on `UITableView` and `UICollectionView` enables arrow-key selection in list views.

For assistive technologies like VoiceOver, Mac Catalyst apps benefit from a key behavior difference: `accessibilityContainerType` containers become independent accessibility elements (nodes) in the accessibility tree on macOS, not just hints to the screen reader. This dramatically reduces the number of elements a VoiceOver user must navigate, providing a more native macOS-style experience.

## Key Topics
- **Keyboard focus** — All interactable controls should be reachable by Tab key; use System Preferences > Keyboard > Shortcuts to enable keyboard navigation. Verify all controls are reachable without mouse interaction.
- **`selectionFollowsFocus`** **[NEW]** — Set to `true` on `UITableView` or `UICollectionView` so arrow keys change selection; UIKit sets this automatically for sidebar tables in `UISplitViewController`.
- **Keyboard shortcuts** — Override `buildMenu(with:)` in `AppDelegate` to add `UIKeyCommand` entries to the macOS menu bar. Shortcuts defined here also benefit Full Keyboard Access users on iPadOS.
- **Raw key codes** — `UIPress.key?.keyCode` (available since iOS 13.4) provides raw keyboard codes, useful for games or advanced keyboard handling.
- **Accessibility containers on macOS** — `accessibilityContainerType` containers become independent nodes in the Mac accessibility tree, grouping child elements into navigable subtrees (analogous to AppKit's model). Reduces visible accessibility elements from many to a handful.
- **Container types** — `.semanticGroup` is the general-purpose type; `.dataTable` for tabular data implementing `UIAccessibilityContainerDataTable`; `.list`, `.landmark` are for web/tvOS content. Standard UIKit views (`UITableView`, `UINavigationBar`, `UITabBar`, `UICollectionView`) are semantic groups by default.
- **Accessibility Inspector improvements** — Now shows iOS UIKit APIs when inspecting Mac Catalyst apps; new "Catalyst" section showing traits and container types; displays view controller for elements; reports automation type for XCUI testing (e.g., "table").
- **Accessibility labels on containers** — Since containers are independent elements on macOS, every container must have a meaningful `accessibilityLabel`; include relevant state (e.g., which item is selected).

## APIs & Frameworks

### UIKit / Accessibility
- **`UITableView.selectionFollowsFocus`** **[NEW]** — `Bool` property; when `true`, moving keyboard focus to a different cell automatically selects it
- **`UICollectionView.selectionFollowsFocus`** **[NEW]** — Same as above for collection views
- **`UIKeyCommand`** — `init(title:action:input:modifierFlags:)` — used to define menu bar keyboard shortcuts
- **`UIMenuBuilder`** — `buildMenu(with:)` override in `AppDelegate`; `insertChild(_:atEndOfMenu:)` for adding menus
- **`UIMenu`** — `init(title:identifier:options:children:)` with `.displayInline` option
- **`UIMenu.Identifier`** — Custom identifier for menu items
- **`UIPress.key?.keyCode`** — Raw key code from `UIPress`; available since iOS 13.4; use in `pressesBegan(_:with:)` override
- **`UIKeyboardHIDUsage`** — Enum values for key codes (e.g., `.keyboardLeftGUI`, `.keyboardB`)
- **`accessibilityContainerType`** — Property on `UIView`/accessibility elements; set to `.semanticGroup`, `.dataTable`, `.list`, or `.landmark`
- **`UIAccessibilityContainerType`** — Enum: `.semanticGroup`, `.dataTable`, `.list`, `.landmark`, `.none`
- **`accessibilityLabel`** — Standard accessibility label; must be set on containers since they become nodes on macOS
- **`isAccessibilityElement`** — Standard property determining whether a view appears in the accessibility tree
- **`UIAccessibilityContainerDataTable`** — Protocol for data table container type

### Tools
- **Accessibility Inspector** — Xcode tool; now shows UIKit iOS APIs for Mac Catalyst apps; new Catalyst section for traits/container types; XCUI automation type reporting

## Code Highlights

Enable arrow-key selection in a table view:
```swift
myTableView.selectionFollowsFocus = true
```

Add a keyboard shortcut to the macOS menu bar:
```swift
override func buildMenu(with builder: UIMenuBuilder) {
    super.buildMenu(with: builder)
    let shareCommand = UIKeyCommand(
        title: NSLocalizedString("Share", comment: ""),
        action: #selector(Self.handleShareMenuAction),
        input: "I",
        modifierFlags: [.command])
    let shareMenu = UIMenu(title: "",
        identifier: UIMenu.Identifier("com.example.RoastedBeans.share"),
        options: .displayInline,
        children: [shareCommand])
    builder.insertChild(shareMenu, atEndOfMenu: .edit)
}
```

Make a `UIStackView` an accessibility container:
```swift
stackView.accessibilityLabel = String(
    format: NSLocalizedString("Available at %@ locations", comment: ""),
    String(locationsAvailable.count))
stackView.accessibilityContainerType = .semanticGroup
```

## Takeaways
- A well-accessible iOS app becomes a well-accessible Mac Catalyst app automatically; the accessibility team converts UIKit APIs to macOS equivalents.
- On macOS, `accessibilityContainerType` containers become independent tree nodes — dramatically reducing navigation steps for VoiceOver users; all containers must have meaningful `accessibilityLabel` values.
- Keyboard shortcuts defined via `UIKeyCommand` and `buildMenu(with:)` serve both Mac Catalyst users and Full Keyboard Access users on iPadOS.
- Use Accessibility Inspector's updated Catalyst view (showing UIKit APIs, container types, and XCUI automation types) to audit your app during development.

---
_Source: WWDC20 Session 10117 page (abstract, chapter summaries, code samples, and resource links)._
