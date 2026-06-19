# SwiftUI on iPad: Add Toolbars, Titles, and More
**WWDC22 · Session 110343** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110343/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This is the second session in the two-part "SwiftUI on iPad" series. Where the first part (10058) covered lists, tables, selection, and split views, this session focuses on toolbars, navigation titles, and document-oriented UI — all significantly upgraded in iOS 16 to take advantage of the iPad's extra screen real estate.

The session walks through refactoring a sample "Places" app toolbar: moving from a compact iPhone-style "More" menu to a fully expanded, user-customizable iPad toolbar with secondary actions, overflow menus, grouped controls, and a `ShareLink` item. It then covers new navigation title APIs — editable titles, title menus, `RenameButton`, and `navigationDocument` — that bring Mac-like document-aware chrome to iPad.

Key new concepts are the `.secondaryAction` toolbar placement, `.toolbarRole(.editor)`, and toolbar customization via stable string IDs. These features are built on the same customization API already available on macOS, so a single codebase now supports both platforms.

## Key Topics

### Overflow Menus and ToolbarItemGroup
Replacing a manual `Menu { }` inside a `ToolbarItem` with a `ToolbarItemGroup` lets the system automatically manage an overflow menu on iPad and Mac. Items that don't fit move into the overflow menu without any extra code.

### Secondary Action Placement (New)
`.secondaryAction` is a new `ToolbarItemPlacement` for controls that are useful but not the most common action. By default these live in the overflow menu; adopting `.toolbarRole(.editor)` promotes them into the center of the navigation bar.

### Toolbar Role (New)
`.toolbarRole(_:)` assigns a semantic role to a toolbar. The `.editor` role shifts the navigation title to the leading position, freeing the center for secondary action items — mirroring the Mac `NSToolbar` layout.

### User-Customizable Toolbars (New on iPad)
Adding a string `id` to the `.toolbar(id:)` modifier and to each `ToolbarItem(id:)` opts the toolbar into user customization (already available on macOS, now on iPadOS). Items with `showsByDefault: false` start life in the customization popover.

### ControlGroup in Toolbar
Wrapping related buttons in `ControlGroup { } label: { }` inside a `ToolbarItem` lets users add/remove them as a unit and collapses them into a smaller labeled menu before overflow.

### ShareLink (New)
`ShareLink(item:)` is a new SwiftUI view that presents a system share sheet for any `Transferable` value, suitable for placement directly in a toolbar.

### Navigation Title Menus and Editable Titles (New)
Passing a trailing closure to `.navigationTitle(_:) { }` creates a title menu (like the macOS File menu). Passing a `Binding` to `.navigationTitle($binding)` enables inline title editing.

### RenameButton (New)
`RenameButton()` placed inside a navigation title's menu actions activates the inline rename field when tapped.

### navigationDocument (New)
`.navigationDocument(_:)` associates a `Transferable` value or `URL` with the view, rendering a specialized title menu header with a draggable document preview and a quick share action. On macOS it also sets the window proxy icon.

## APIs & Frameworks

**SwiftUI**

_Toolbar_
- `ToolbarItem(id:placement:showsByDefault:content:)` **[NEW]** — `showsByDefault` parameter is new; allows items to be hidden initially (customization only)
- `ToolbarItemGroup(placement:content:)` — existing; gains overflow menu behavior on iPad in iOS 16
- `ToolbarItemPlacement.secondaryAction` **[NEW]** — new placement for non-primary toolbar actions
- `.toolbar(id:content:)` **[NEW]** — toolbar modifier overload with string ID enabling user customization on iPad
- `.toolbarRole(_:)` **[NEW]** — assigns semantic role to a toolbar
- `ToolbarRole.editor` **[NEW]** — editor role; moves title leading, expands center for secondary actions
- `ToolbarRole.automatic` / `.navigationStack` / `.browser` **[NEW]** — other roles

_Controls_
- `ControlGroup(content:label:)` **[NEW]** — wraps related controls; collapses to a labeled menu in toolbars before overflow
- `ShareLink(item:)` **[NEW]** — share sheet trigger for any `Transferable` value
- `ShareLink(item:subject:message:)` **[NEW]** — with optional subject and message
- `RenameButton()` **[NEW]** — activates navigation title rename field when inside a title menu

_Navigation Titles_
- `.navigationTitle(_:content:)` **[NEW]** — overload with trailing closure creates a title menu
- `.navigationTitle(_:content:)` with `Binding` **[NEW]** — enables editable navigation title
- `.navigationDocument(_:)` **[NEW]** — associates a `Transferable` value or `URL` with the view for document header in title menu and macOS proxy icon

## Code Highlights

Full customizable toolbar with secondary actions, `ControlGroup`, and `ShareLink`:
```swift
PlaceDetailContent(place: $place)
    .toolbar(id: "place") {
        ToolbarItem(id: "new", placement: .primaryAction) {
            NewButton()
        }
        ToolbarItem(id: "favorite", placement: .secondaryAction) {
            FavoriteToggle(place: $place)
        }
        ToolbarItem(id: "image", placement: .secondaryAction) {
            ControlGroup {
                AdjustImageButton(place: $place)
                AdjustMapButton(place: $place)
            } label: {
                Label("Edits", systemImage: "wand.and.stars")
            }
        }
        ToolbarItem(id: "share", placement: .secondaryAction, showsByDefault: false) {
            ShareLink(item: place)
        }
    }
    .toolbarRole(.editor)
```

Editable navigation title with title menu and document association:
```swift
PlaceDetailContent(place: $place)
    .navigationTitle($place.name) {
        MyPrintButton()
        RenameButton()
    }
    .navigationDocument(place.url)
```

## Takeaways
- Replace manual "More" menus with `ToolbarItemGroup` and `.secondaryAction` placement; iPad handles overflow automatically.
- `.toolbarRole(.editor)` + `.toolbar(id:)` + per-item `id`s enable user-customizable toolbars on iPad (same API as macOS).
- `ControlGroup(content:label:)` groups related toolbar buttons so users customize them as a unit.
- `ShareLink`, `RenameButton`, and `.navigationDocument(_:)` bring document-centric Mac patterns to iPadOS with minimal code.

---
_Source: WWDC22 Session 110343 page (abstract, chapter summaries, code samples, and resource links)._
