# SwiftUI on the Mac: Build the Fundamentals
**WWDC21 · Session 10062** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10062/)

_Platforms:_ macOS Monterey 12

## Overview
This is the first part of a two-session code-along series focused on building a full-featured Mac app using SwiftUI. The session establishes four principles that distinguish great Mac apps — flexible, familiar, expansive, and precise — then demonstrates each principle by building a garden-tracking app from scratch.

The code-along covers constructing a two-column `NavigationView` with a sidebar featuring `DisclosureGroup` outline navigation, converting a list to a multi-column sortable `Table`, adding toolbar actions and the `.searchable` modifier, and wiring up commands in the macOS main menu. State is persisted across window restores using `@SceneStorage`, and multi-window support emerges naturally from `WindowGroup`.

Part two of the series (Session 10039) continues with accent color customization, drag and drop, and camera integration with an iOS device.

## Key Topics

### Four Principles of Great Mac Apps
Flexible (adapts to workflows, keyboard, mouse, customization), Familiar (standard controls, consistent placement), Expansive (uses screen space with outlines, sidebars, popovers, tabs), and Precise (tight spacing, mouse-targeted controls).

### Sidebar with DisclosureGroup and SceneStorage
A `List` containing a `DisclosureGroup` creates a collapsible outline sidebar. `@SceneStorage` persists the expansion state and selection across app launches and window restores without extra boilerplate.

### Table with Sortable Columns
`Table` replaces `List` when multiple columns of textual data and sorting are needed. `TableColumn` accepts either a key path for simple text or a `ViewBuilder` closure for custom cell content. A `sortOrder` binding enables column-header sorting.

### Toolbar and Searchable
The `.toolbar` modifier adds buttons for common actions. Adding `.searchable(text:)` to the `Table` is all that is needed to surface a macOS search field; the developer filters the data source by the bound search text.

### Multi-Window Support
`WindowGroup` automatically enables multiple independent windows via File > New Window. Each window maintains its own `@SceneStorage` selection and expansion state, so users can view different gardens simultaneously.

### Main Menu Commands with FocusedBinding
`CommandGroup` and `CommandMenu` declare app-specific menu items. `@FocusedBinding` and `.focusedSceneValue(_:_:)` route menu actions to the frontmost window's data, enabling per-window operations like adding plants or marking selections as watered.

## APIs & Frameworks

- `NavigationView` — two-column layout for Mac sidebar + detail
- `List` — sidebar list with selection binding
- `DisclosureGroup` — collapsible outline groups in a sidebar
- `@SceneStorage` — persists lightweight state per window scene
- `Table` **[NEW]** — multi-column sortable data table for macOS
- `TableColumn` **[NEW]** — defines a column in a `Table` with key path or `ViewBuilder`
- `.searchable(text:)` **[NEW]** — adds a search field to a view; binds to a `String`
- `.toolbar { }` — adds toolbar items to a macOS window
- `ToolbarItem` — individual toolbar item within a `.toolbar` block
- `SidebarCommands()` **[NEW]** — adds system-provided sidebar toggle menu items
- `CommandGroup` — inserts custom menu items relative to system positions
- `CommandMenu` — creates a new top-level menu in the macOS menu bar
- `@FocusedBinding` **[NEW]** — reads a value from the focused scene's environment
- `.focusedSceneValue(_:_:)` **[NEW]** — exposes a binding from a scene for `@FocusedBinding`
- `FocusedValues` — extension point for declaring custom focused value keys
- `WindowGroup` — entry point for multi-window SwiftUI apps on macOS
- `.badge(_:)` — adds a numeric badge to a `List` row or tab
- `Label` — icon + text label using SF Symbols

## Code Highlights

Two-column NavigationView with sidebar selection persisted via SceneStorage:
```swift
@SceneStorage("expansionState") var expansionState = false
@SceneStorage("selectedGardenID") var selectedGardenID: Garden.ID?

NavigationView {
    Sidebar(selection: $selectedGardenID, expansionState: $expansionState)
    GardenDetail(selection: $selectedGardenID)
}
```

Multi-column sortable table:
```swift
@State private var sortOrder: [KeyPathComparator<Plant>] = []

Table(garden.plants.sorted(using: sortOrder), sortOrder: $sortOrder) {
    TableColumn("Name", value: \.name)
    TableColumn("Days to Maturity") { plant in
        Text("\(plant.daysToMaturity) days")
    }
    TableColumn("Favorite", value: \.isFavorite) { plant in
        Toggle("", isOn: binding(for: plant))
    }
}
.searchable(text: $searchText)
```

Routing a menu action to the frontmost window:
```swift
@FocusedBinding(\.garden) private var garden

// In GardenDetail:
.focusedSceneValue(\.garden, $garden)
```

## Takeaways

- `Table` and `TableColumn` bring native Mac-style multi-column sorted lists to SwiftUI with minimal code.
- `@SceneStorage` is the right tool for persisting UI state (selection, expansion) per window with zero persistence boilerplate.
- `.searchable` adds a full macOS search field with a single modifier; the developer only needs to filter the data source.
- `@FocusedBinding` and `.focusedSceneValue` solve the multi-window menu routing problem — actions always target the frontmost scene.

---
_Source: WWDC21 Session 10062 page (abstract, chapter summaries, code samples, and resource links)._
