# Meet Desktop-Class iPad
**WWDC22 · Session 10069** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10069/)

_Platforms:_ iPadOS 16, iOS 16, macOS Ventura 13 (Mac Catalyst)

## Overview
iOS 16 introduces a comprehensive set of UIKit enhancements aimed at bringing desktop-class productivity to iPad. This session covers three major areas: new `UINavigationBar` styles and center item customization that surface more actions in the toolbar, document-centric features (title menu, drag & drop from the nav bar, inline rename) that elevate document-based apps, and updated Search with inline placement and search suggestions. All features are designed to translate automatically to Mac Catalyst, mapping navigation bar content to `NSToolbar` with minimal extra work.

## Key Topics
- **`UINavigationItem.ItemStyle`** — three styles: `.navigator` (default, centered title), `.browser` (leading title, suited for hierarchical browsing like Files/Safari), `.editor` (leading title with back button, suited for document editing destinations); browser and editor styles free the center of the bar for toolbar items
- **Center item groups** — `UINavigationItem.centerItemGroups` places `UIBarButtonItemGroup` collections in the liberated center region; three group types: fixed (always present, not movable), movable (can reorder but not remove), optional (user can add/remove/reorder); optional groups require a `customizationIdentifier`
- **UIBarButtonItemGroup customization** — `representativeItem` lets UIKit collapse a multi-item group when space is tight; if `representativeItem` has no action, UIKit synthesizes a submenu of the group's items; overflow menu automatically holds items that don't fit
- **Bar customization persistence** — set `UINavigationItem.customizationIdentifier` to enable user customization; UIKit automatically saves and restores the arrangement; omitting it means center items appear but cannot be customized
- **Mac Catalyst mapping** — in "Optimize for Mac" mode, UINavigationBar content maps to `NSToolbar`; center item groups, customization rules, and the window proxy/title all work via the standard NSToolbar customization mechanism
- **Title menu** — `UINavigationItem.titleMenuProvider` closure receives an array of system-suggested actions (Duplicate, Move, Rename, Export, Print — filtered by responder chain availability) and returns the final `UIMenu`; tapping the title presents the menu
- **UIDocumentProperties** — set on `UINavigationItem.documentProperties` to enable a document header in the title menu with drag & drop (`dragItemsProvider`) and sharing (`activityViewControllerProvider`); also drives the macOS proxy icon automatically
- **Inline rename** — set `UINavigationItem.renameDelegate` (conforming to `UINavigationItemRenameDelegate`) to get built-in rename UI inside the navigation bar title on iOS and in the window title on macOS; alternatively implement `UIResponder.rename(_:)` for custom rename UI
- **Inline search placement** — on iPadOS 16, the search bar is inline in the navigation bar by default (saving vertical space); `UINavigationItem.preferredSearchBarPlacement` restores the old behavior
- **Search suggestions** — `UISearchController.searchSuggestions` (array of `UISearchSuggestionItem`); updated in `updateSearchResults(for:)` as query changes; acted on in `updateSearchResults(for:selecting:)`

## APIs & Frameworks
**UIKit — UINavigationItem**
- `UINavigationItem.ItemStyle` **[NEW]** — `.navigator`, `.browser`, `.editor`
- `UINavigationItem.style: UINavigationItem.ItemStyle` **[NEW]**
- `UINavigationItem.centerItemGroups: [UIBarButtonItemGroup]` **[NEW]** — toolbar items in the center region
- `UINavigationItem.customizationIdentifier: String?` **[NEW]** — enables user customization and persistence
- `UINavigationItem.titleMenuProvider: (([UIMenuElement]) -> UIMenu?)?` **[NEW]** — closure to build the title menu
- `UINavigationItem.documentProperties: UIDocumentProperties?` **[NEW]** — document metadata for title menu header
- `UINavigationItem.renameDelegate: UINavigationItemRenameDelegate?` **[NEW]** — enables inline rename UI
- `UINavigationItem.preferredSearchBarPlacement: UINavigationItem.SearchBarPlacement` **[NEW]** — `.stacked` (old) or `.inline` (new default)

**UIKit — UIBarButtonItem / UIBarButtonItemGroup**
- `UIBarButtonItem.creatingFixedGroup() -> UIBarButtonItemGroup` **[NEW]** — convenience; creates a fixed (non-removable, non-movable) group from a single item
- `UIBarButtonItem.creatingMovableGroup(customizationIdentifier:) -> UIBarButtonItemGroup` **[NEW]** — convenience; creates a movable (non-removable) group
- `UIBarButtonItemGroup.fixedGroup(representativeItem:items:) -> UIBarButtonItemGroup` **[NEW]**
- `UIBarButtonItemGroup.movableGroup(customizationIdentifier:representativeItem:items:) -> UIBarButtonItemGroup` **[NEW]**
- `UIBarButtonItemGroup.optionalGroup(customizationIdentifier:isInDefaultCustomization:representativeItem:items:) -> UIBarButtonItemGroup` **[NEW]** — user can add/remove
- `UIBarButtonItemGroup.representativeItem: UIBarButtonItem?` — item shown when the group is collapsed; no action = UIKit synthesizes a submenu

**UIKit — UIDocumentProperties** **[NEW]**
- `UIDocumentProperties(url: URL)` — creates document metadata from a URL; UIKit fetches title/thumbnail
- `UIDocumentProperties.dragItemsProvider: ((UIDragSession) -> [UIDragItem])?` — enables drag from title menu
- `UIDocumentProperties.activityViewControllerProvider: (() -> UIActivityViewController)?` — enables sharing from title menu

**UIKit — UINavigationItemRenameDelegate** **[NEW]**
- `navigationItem(_:didEndRenamingWith:)` — required; called when user commits a rename

**UIKit — Search**
- `UISearchController.searchSuggestions: [UISearchSuggestion]?` **[NEW]** — array of suggestion items to display
- `UISearchSuggestionItem(localizedSuggestion:localizedDescription:iconImage:)` **[NEW]** — concrete suggestion item type
- `UISearchResultsUpdating.updateSearchResults(for:selecting:)` **[NEW]** — delegate method called when user picks a suggestion
- `UISearchTextField.searchSuggestions: [UISearchSuggestion]?` **[NEW]** — parallel property if using `UISearchTextField` directly

## Code Highlights
Configuring center item groups with all three group types:
```swift
navigationItem.customizationIdentifier = "com.jetpack.blueprints.maineditor"
navigationItem.centerItemGroups = [
    UIBarButtonItem(title: "Insert", image: UIImage(systemName: "photo"),
                    primaryAction: UIAction { _ in }).creatingFixedGroup(),
    UIBarButtonItem(title: "Draw", image: UIImage(systemName: "scribble"),
                    primaryAction: UIAction { _ in })
        .creatingMovableGroup(customizationIdentifier: "Draw"),
    .optionalGroup(
        customizationIdentifier: "Shapes",
        representativeItem: UIBarButtonItem(title: "Shapes",
                                            image: UIImage(systemName: "square.on.circle")),
        items: [
            UIBarButtonItem(title: "Square", image: UIImage(systemName: "square"),
                            primaryAction: UIAction { _ in }),
            UIBarButtonItem(title: "Circle", image: UIImage(systemName: "circle"),
                            primaryAction: UIAction { _ in }),
        ]),
    .optionalGroup(customizationIdentifier: "Format",
                   isInDefaultCustomization: false,   // not shown by default
                   representativeItem: UIBarButtonItem(title: "BIU",
                                                       image: UIImage(systemName: "bold.italic.underline")),
                   items: [ /* bold, italic, underline items */ ])
]
```

Title menu with custom action + document properties for sharing and drag & drop:
```swift
navigationItem.titleMenuProvider = { suggestedActions in
    UIMenu(children: suggestedActions + [
        UIAction(title: "Comments", image: UIImage(systemName: "text.bubble")) { _ in }
    ])
}

let documentProperties = UIDocumentProperties(url: documentURL)
if let itemProvider = NSItemProvider(contentsOf: documentURL) {
    documentProperties.dragItemsProvider = { _ in [UIDragItem(itemProvider: itemProvider)] }
    documentProperties.activityViewControllerProvider = {
        UIActivityViewController(activityItems: [itemProvider], applicationActivities: nil)
    }
}
navigationItem.documentProperties = documentProperties
```

Inline rename delegate:
```swift
navigationItem.renameDelegate = self

extension ViewController: UINavigationItemRenameDelegate {
    func navigationItem(_ navigationItem: UINavigationItem, didEndRenamingWith title: String) {
        documentBrowserViewController.renameDocument(at: documentURL, proposedName: title) { _, _ in }
    }
}
```

Search suggestions:
```swift
func updateSearchResults(for searchController: UISearchController) {
    searchController.searchSuggestions = [
        UISearchSuggestionItem(localizedSuggestion: "Sample Suggestion",
                               localizedDescription: nil,
                               iconImage: UIImage(systemName: "rectangle.and.text.magnifyingglass"))
    ]
}

func updateSearchResults(for searchController: UISearchController,
                         selecting searchSuggestion: UISearchSuggestion) {
    searchController.searchBar.text = searchSuggestion.localizedSuggestion
}
```

## Takeaways
- The new `.browser` and `.editor` nav bar styles free the center of the bar for toolbar items via `centerItemGroups`, enabling richer iPad toolbars that automatically map to `NSToolbar` on Mac Catalyst.
- Use `titleMenuProvider` + `UIDocumentProperties` to give document apps a title-tap menu with sharing, drag & drop, and rename — all from the navigation bar.
- Search suggestions via `UISearchController.searchSuggestions` help users discover queries faster; update them dynamically as the user types.
- Set `customizationIdentifier` on `UINavigationItem` to unlock user-customizable toolbars with automatic save/restore.

---
_Source: WWDC22 Session 10069 page (abstract, chapter summaries, code samples, and resource links)._
