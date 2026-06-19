# SwiftUI on iPad: Organize Your Interface
**WWDC22 · Session 10058** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10058/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This is the first session in the two-part "SwiftUI on iPad" series, focused on building desktop-class iPad apps with SwiftUI. It covers three major areas: multi-column tables, a robust selection model with context menus, and the new `NavigationSplitView` type for sidebar-based layouts.

Multi-column `Table` views (introduced on macOS Monterey) arrive on iPadOS 16 with the same API, supporting multiple columns, sortable headers, and sections. The session pairs tables with SwiftUI's selection model — explaining tags, selection state types, and the new lightweight multiple selection with keyboards — and adds multi-select context menus that operate on a set of selected identifiers. The session concludes by demonstrating `NavigationSplitView`, which replaces the older `NavigationView` with explicit two- and three-column layouts that automatically collapse in compact size classes.

## Key Topics

### Multi-Column Tables on iPadOS
`Table` with a column builder creates a sortable, information-dense grid on iPad (same API as macOS). Columns specify a `value` key path for comparator-based sorting. In compact size classes, only the first column is shown, so the first column should be designed for compact layout. Tables do not scroll horizontally on iPad.

### Sections in Tables (New)
SwiftUI adds support for `Section` within `Table` on iPadOS 16 and macOS Ventura.

### Selection Model
Selection in lists and tables is driven by a `Binding` to a tag-matching state type. `Table` automatically uses the row value's `Identifiable.ID` as the selection tag. Supported selection types: `Optional<ID>` (single), `Set<ID>` (multiple). New in iOS 16: single-row selection no longer requires edit mode.

### Lightweight Multiple Selection (New on iPad)
With a hardware keyboard attached, users can select multiple rows without entering edit mode using Shift/Command shortcuts. Touch users still use a two-finger pan gesture (handled automatically by SwiftUI) or the `EditButton`.

### Multi-Select Context Menus (New)
`.contextMenu(forSelectionType:perform:)` presents context menus that receive the current selection as a `Set`. The closure handles three cases: empty set (empty area), single item, and multiple items.

### NavigationSplitView (New)
`NavigationSplitView` replaces `NavigationView` for sidebar-based layouts. Supports two-column (sidebar + detail) and three-column (sidebar + content + detail) layouts with automatic collapse to a navigation stack in compact size classes.

### NavigationSplitViewStyle
- `.automatic` — recommended default; adapts to context
- `.balanced` — equal column weighting
- `.prominentDetail` — always prefers showing the detail column

## APIs & Frameworks

**SwiftUI**

_Tables_
- `Table(_:columns:)` **[NEW on iPad]** — multi-column table view; previously macOS-only
- `Table(_:selection:sortOrder:columns:)` **[NEW on iPad]** — with selection and sort bindings
- `TableColumn(_:value:content:)` **[NEW on iPad]** — column builder with key path comparator
- `TableColumn(_:value:)` **[NEW on iPad]** — convenience overload for string content (no view builder)
- `TableColumn.width(_:)` / `.width(min:ideal:max:)` — column width constraints
- `KeyPathComparator(_:order:)` — comparator for sortable table columns (existing, used with tables)
- `Section` inside `Table` **[NEW]** — section support in tables

_Selection_
- `List(_:selection:rowContent:)` — existing; single-row selection no longer requires edit mode **[changed]**
- `Table(_:selection:sortOrder:columns:)` — table with selection binding
- `.tag(_:)` — manual tag for selectable views

_Context Menus_
- `.contextMenu(forSelectionType:perform:)` **[NEW]** — multi-select context menu; closure receives `Set<SelectionType>`
- `.contextMenu(menuItems:preview:)` **[NEW]** — existing context menu with custom preview image

_Navigation_
- `NavigationSplitView(sidebar:detail:)` **[NEW]** — two-column split view
- `NavigationSplitView(sidebar:content:detail:)` **[NEW]** — three-column split view
- `NavigationSplitViewStyle` **[NEW]** — style protocol
- `.navigationSplitViewStyle(_:)` **[NEW]** — modifier to set split view style
- `NavigationSplitViewVisibility` **[NEW]** — controls column visibility programmatically
- `EditButton()` — existing; recommended for edit mode toggle on iPad

## Code Highlights

Multi-column sortable table with selection:
```swift
struct PlacesTable: View {
    @EnvironmentObject var modelData: ModelData
    @State private var sortOrder = [KeyPathComparator(\Place.name)]
    @State private var selection: Set<Place.ID> = []

    var body: some View {
        Table(modelData.places, selection: $selection, sortOrder: $sortOrder) {
            TableColumn("Name", value: \.name) { place in PlaceCell(place) }
            TableColumn("Comfort Level", value: \.comfortDescription).width(200)
            TableColumn("Noise", value: \.noiseLevel) { NoiseLevelView(level: $0.noiseLevel) }
        }
        .onChange(of: sortOrder) { modelData.sort(using: $0) }
    }
}
```

Multi-select context menu:
```swift
Table(...)
.contextMenu(forSelectionType: Place.ID.self) { items in
    if items.isEmpty {
        AddPlaceButton()
    } else {
        if items.count == 1 {
            FavoriteButton(isSet: $modelData.places[items.first!].isFavorite)
        }
        AddToGuideButton(items)
    }
}
```

Two-column `NavigationSplitView`:
```swift
NavigationSplitView {
    SidebarView()
} detail: {
    Text("Select a place")
}
```

## Takeaways
- `Table` brings macOS-style multi-column sortable tables to iPadOS 16 with the same API; design the first column for compact size classes.
- SwiftUI's selection model is tag-based; `Table` auto-tags rows by `Identifiable.ID`. Single-row selection no longer requires edit mode in iOS 16.
- `.contextMenu(forSelectionType:)` enables rich multi-select context menus with a single modifier; handle empty, single, and multiple item sets in one closure.
- `NavigationSplitView` replaces `NavigationView` for sidebar layouts and collapses automatically to a stack in compact size classes.

---
_Source: WWDC22 Session 10058 page (abstract, chapter summaries, code samples, and resource links)._
