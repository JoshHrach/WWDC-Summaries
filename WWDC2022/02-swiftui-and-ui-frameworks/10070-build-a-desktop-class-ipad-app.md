# Build a Desktop-Class iPad App
**WWDC22 · Session 10070** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10070/)

_Platforms:_ iPadOS 16, macOS Ventura 13 (via Mac Catalyst)

## Overview
This code-along session demonstrates how to modernize an existing iPad Markdown editor using all of iPadOS 16's desktop-class UIKit APIs. The session is organized around four upgrades: navigation bar organization (navigation style, document properties, title menu, rename UI, and center bar button items), collection view multi-item selection and menus, Find and Replace, and the new edit menu. Each feature is applied incrementally to a real app so developers can see the before/after and apply similar patterns to their own code.

The session emphasizes that desktop-class iPad apps should minimize reliance on distinct "edit modes" and instead expose actions in context menus, overflow menus, and the navigation bar. The APIs introduced automatically translate to native macOS behavior when the app is built with Mac Catalyst — navigation bar editor style becomes a leading-title toolbar, center items become macOS toolbar items with customization, and title menu system actions surface in the File menu.

## Key Topics

### Navigation Bar Styles & Document Properties
- `UINavigationItem.style` — three new values: `.navigator` (push/pop), `.browser` (back/forward), `.editor` (leading title + center area) **[NEW]**
- `.editor` style aligns title to the leading edge, opening the center for additional controls
- `UINavigationItem.backAction: UIAction?` — replaces the auto-generated back button with a custom action **[NEW]**
- `UIDocumentProperties(url:)` — generates a document header card (name, size, icon) for display in the title menu **[NEW]**
  - `dragItemsProvider: (([UIDragSession]) -> [UIDragItem])?` — returns drag items for the document icon
  - `activityViewControllerProvider: (() -> UIActivityViewController)?` — returns share sheet
- `UINavigationItem.documentProperties: UIDocumentProperties?` — attaches document header to navigation item **[NEW]**
- `UINavigationItem.titleMenuProvider: (([UIMenuElement]) -> UIMenu?)?` — closure receives system-suggested actions; return a `UIMenu` to show in title menu **[NEW]**

### Rename UI & System Title Menu Actions
- `UINavigationItemRenameDelegate` protocol **[NEW]**
  - `navigationItem(_:didEndRenamingWith title:)` — called when user commits rename
- `UINavigationItem.renameDelegate` — assign to activate built-in inline rename UI in the title bar **[NEW]**
- System title menu actions automatically suggested when these `UIResponder` methods are overridden:
  - `duplicate(_:)`, `move(_:)`, `rename(_:)`, `exportToService(_:)` — override to opt into system actions

### Center Bar Button Item Groups
- `UINavigationItem.customizationIdentifier: String?` — opt into toolbar customization; persists user layout across launches **[NEW]**
- `UINavigationItem.centerItemGroups: [UIBarButtonItemGroup]` — array of groups shown in the center area **[NEW]**
- `UIBarButtonItem.creatingFixedGroup() -> UIBarButtonItemGroup` — non-customizable group **[NEW]**
- `UIBarButtonItem.creatingOptionalGroup(customizationIdentifier:) -> UIBarButtonItemGroup` — user-movable group **[NEW]**
- `UIBarButtonItemGroup.optionalGroup(customizationIdentifier:isInDefaultCustomization:representativeItem:items:)` — create group with representative item for overflow/customizer popover **[NEW]**
- `UIBarButtonItemGroup.menuRepresentation: UIMenuElement?` — custom menu for overflow representation **[NEW]**
- Items not fitting in the bar appear in an auto-generated overflow menu; custom view items need `menuRepresentation` set

### Multi-Item Collection View Menus
- `UICollectionView.allowsMultipleSelection = true` — lightweight multiple selection (no edit mode) **[NEW usage]**
- `UICollectionView.allowsFocus = true` + `selectionFollowsFocus = true` — keyboard-driven selection **[NEW]**
- `collectionView(_:performPrimaryActionForItemAt:)` — new delegate method; called only on single tap without multi-selection; replaces selection-based scroll logic **[NEW]**
- `collectionView(_:contextMenuConfigurationForItemsAt:point:)` — new multi-item variant of context menu delegate; `indexPaths` can be empty (blank space), single, or multiple **[NEW]**
- Single-item variant deprecated in iPadOS 16

### Find and Replace & Edit Menu
- `UITextView.isFindInteractionEnabled = true` — enables system Find and Replace (Cmd+F) **[NEW]**
- `textView(_:editMenuForTextIn:suggestedActions:) -> UIMenu?` — new `UITextViewDelegate` method; combine custom and suggested actions **[NEW]**
- `UIMenu.preferredElementSize: UIMenu.ElementSize` — `.small` for compact side-by-side layout **[NEW]**
- `UIMenuElement.Attributes.keepsMenuPresented` — action keeps menu open after invocation **[NEW]**

## APIs & Frameworks

**UIKit — Navigation** **[NEW]**
- `UINavigationItem.ItemStyle` — `.automatic`, `.navigator`, `.browser`, `.editor` **[NEW]**
- `UINavigationItem.style: UINavigationItem.ItemStyle` **[NEW]**
- `UINavigationItem.backAction: UIAction?` **[NEW]**
- `UIDocumentProperties` **[NEW]**
  - `init(url: URL)`
  - `dragItemsProvider: (([UIDragSession]) -> [UIDragItem])?`
  - `activityViewControllerProvider: (() -> UIActivityViewController)?`
- `UINavigationItem.documentProperties: UIDocumentProperties?` **[NEW]**
- `UINavigationItem.titleMenuProvider: (([UIMenuElement]) -> UIMenu?)?` **[NEW]**
- `UINavigationItem.renameDelegate: UINavigationItemRenameDelegate?` **[NEW]**
- `UINavigationItemRenameDelegate` — `navigationItem(_:didEndRenamingWith:)` **[NEW]**

**UIKit — Bar Button Groups** **[NEW]**
- `UINavigationItem.customizationIdentifier: String?` **[NEW]**
- `UINavigationItem.centerItemGroups: [UIBarButtonItemGroup]` **[NEW]**
- `UIBarButtonItem.creatingFixedGroup() -> UIBarButtonItemGroup` **[NEW]**
- `UIBarButtonItem.creatingOptionalGroup(customizationIdentifier:) -> UIBarButtonItemGroup` **[NEW]**
- `UIBarButtonItemGroup.optionalGroup(customizationIdentifier:isInDefaultCustomization:representativeItem:items:)` **[NEW]**
- `UIBarButtonItemGroup.menuRepresentation: UIMenuElement?` **[NEW]**

**UIKit — Collection View** **[NEW]**
- `UICollectionViewDelegate.collectionView(_:performPrimaryActionForItemAt:)` **[NEW]**
- `UICollectionViewDelegate.collectionView(_:contextMenuConfigurationForItemsAt:point:) -> UIContextMenuConfiguration?` **[NEW multi-item variant]**

**UIKit — Text / Menus** **[NEW]**
- `UITextView.isFindInteractionEnabled: Bool` **[NEW]**
- `UITextViewDelegate.textView(_:editMenuForTextIn:suggestedActions:) -> UIMenu?` **[NEW]**
- `UIMenu.preferredElementSize: UIMenu.ElementSize` — `.small`, `.medium`, `.large` **[NEW]**
- `UIMenuElement.Attributes.keepsMenuPresented` **[NEW]**

## Code Highlights

Navigation bar editor style and document properties:
```swift
// Enable editor style
navigationItem.style = .editor

// Replace done button with a back action
navigationItem.backAction = UIAction(title: "Documents") { [weak self] _ in
    self?.dismiss(animated: true)
}

// Attach document header
let properties = UIDocumentProperties(url: document.fileURL)
if let itemProvider = NSItemProvider(contentsOf: document.fileURL) {
    properties.dragItemsProvider = { _ in [UIDragItem(itemProvider: itemProvider)] }
    properties.activityViewControllerProvider = {
        UIActivityViewController(activityItems: [itemProvider], applicationActivities: nil)
    }
}
navigationItem.documentProperties = properties
```

Title menu with system and custom actions:
```swift
navigationItem.titleMenuProvider = { [unowned self] suggested in
    var children = suggested // includes Rename, Duplicate, Move from UIResponder overrides
    children += [
        UIMenu(title: "Export…", image: UIImage(systemName: "arrow.up.forward.square"),
               children: [
                   UIAction(title: "HTML") { _ in /* ... */ },
                   UIAction(title: "PDF")  { _ in /* ... */ }
               ])
    ]
    return UIMenu(children: children)
}
navigationItem.renameDelegate = self
```

Center items with customization:
```swift
navigationItem.customizationIdentifier = "editorView"
navigationItem.centerItemGroups = [
    UIBarButtonItem(title: "Sync Scrolling", ...).creatingFixedGroup(),
    UIBarButtonItem(title: "Add Link", ...).creatingOptionalGroup(customizationIdentifier: "addLink"),
    UIBarButtonItemGroup.optionalGroup(customizationIdentifier: "textFormat",
                                        isInDefaultCustomization: false,
                                        representativeItem: UIBarButtonItem(title: "Format"),
                                        items: [bold, italic, underline])
]
```

Multi-item context menu:
```swift
func collectionView(_ collectionView: UICollectionView,
                    contextMenuConfigurationForItemsAt indexPaths: [IndexPath],
                    point: CGPoint) -> UIContextMenuConfiguration? {
    if indexPaths.isEmpty { /* blank space menu */ }
    else if indexPaths.count == 1 { /* single item menu */ }
    else { /* multi-item menu */ }
    return UIContextMenuConfiguration(actionProvider: { _ in UIMenu(children: actions) })
}
```

## Takeaways
- Set `UINavigationItem.style = .editor` to align the title leading and unlock the center bar area for contextual tools; this automatically becomes a proper macOS toolbar on Catalyst.
- `UIDocumentProperties` adds a rich document header to the title menu — including drag-out and sharing support — with just a few lines of code backed by `NSItemProvider`.
- Use `creatingFixedGroup()` for critical toolbar items and `creatingOptionalGroup(customizationIdentifier:)` for everything else, so users can tailor the bar to their workflow without losing essential controls.
- Replace `didSelectItemAt` scroll logic with `performPrimaryActionForItemAt` so that multi-item selection does not trigger the single-item action.

---
_Source: WWDC22 Session 10070 page (abstract, transcript, and code samples)._
