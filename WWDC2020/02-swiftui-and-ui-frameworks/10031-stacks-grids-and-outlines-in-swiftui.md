# Stacks, Grids, and Outlines in SwiftUI
**WWDC20 · Session 10031** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10031/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
SwiftUI 2.0 introduces a suite of new layout primitives for displaying collections of data: `LazyVStack` and `LazyHStack` for performant scrolling lists, `LazyVGrid` and `LazyHGrid` for grid layouts, and `OutlineGroup` and `DisclosureGroup` for hierarchical and progressive disclosure of content. Together with enhanced `List` support for hierarchical data (via the `children` key path), these primitives make it straightforward to build complex, adaptive layouts that work identically across iOS, iPadOS, and macOS.

The session uses a sandwich gallery app and the ShapeEdit document-based app as running examples. It explains when to prefer lazy vs. eager stacks, how to define adaptive grid columns, how `OutlineGroup` is implemented using recursive `DisclosureGroup` composition, and how to use `Form` with `DisclosureGroup` for inspector-style settings panels.

## Key Topics

### Lazy Stacks (NEW)
- `VStack` / `HStack` render all children eagerly — slow for large dynamic collections inside `ScrollView`
- `LazyVStack` and `LazyHStack` render content incrementally as it becomes visible during scrolling — preserves responsiveness and memory
- Rule: start with `VStack`/`HStack`; adopt `Lazy*Stack` only after profiling with Instruments reveals a bottleneck
- Content inside a `LazyVStack` that is already on-screen renders eagerly — nested non-scrolling stacks do not benefit from lazy rendering

### Lazy Grids (NEW)
- `LazyVGrid(columns:spacing:content:)` and `LazyHGrid(rows:spacing:content:)` — grid layouts that load content lazily
- Columns/rows defined by an array of `GridItem` values
- `GridItem(.flexible())` — default; fills available space equally among columns
- `GridItem(.fixed(size))` — fixed-size column
- `GridItem(.adaptive(minimum:))` — creates as many columns as fit while maintaining a minimum width; adapts to window/screen size automatically (great for resizable macOS windows and iPad landscape)
- Grid items accept `spacing` and `alignment` parameters

### List with Hierarchical Data (NEW)
- `List(data, children: \.keyPath) { item in ... }` — **new initializer** that traverses a tree using the `children` key path
- Items without children (nil value for the key path) are leaf nodes; items with children get a disclosure indicator
- Content is always loaded lazily in `List`
- `SidebarListStyle` (new in iOS 14) — bold section headers, sidebar appearance

### OutlineGroup (NEW)
- `OutlineGroup(data, children: \.keyPath) { item in ... }` — traverses hierarchical data and builds a recursive outline
- Can be placed inside a `List` or `ForEach` for custom hierarchical layouts (e.g., multiple canvases with separate outline sections per canvas)
- Implemented internally as: `ForEach` → `DisclosureGroup` (label from item, content from recursive `OutlineGroup` over children) — content of a `DisclosureGroup` is only evaluated when expanded

### DisclosureGroup (NEW)
- `DisclosureGroup("Label") { content }` — collapsible section with a disclosure indicator
- Optional `isExpanded: Binding<Bool>` parameter for programmatic control of expansion state
- Label can be any `View` (use `Label` with SF Symbol for icon + title)
- Ideal for inspector panels, settings sections, and any progressive disclosure UI

### Form
- `Form { ... }` — semantic container for settings and control-heavy views
- Works on all platforms; particularly well-suited for macOS Settings scenes (new in SwiftUI 2.0)
- Combine with `DisclosureGroup` to organize groups of related controls

### Composition Principle
- `OutlineGroup` is itself built from `ForEach` + `DisclosureGroup` recursion
- Start with simple primitives; combine them for complex behavior — this is SwiftUI's design philosophy

## APIs & Frameworks

**SwiftUI layout primitives:**
- `VStack` / `HStack` / `ZStack` — eager layout containers
- `LazyVStack(alignment:spacing:pinnedViews:content:)` **[NEW]** — vertically lazy stack
- `LazyHStack(alignment:spacing:pinnedViews:content:)` **[NEW]** — horizontally lazy stack
- `LazyVGrid(columns:alignment:spacing:pinnedViews:content:)` **[NEW]** — vertical lazy grid
- `LazyHGrid(rows:alignment:spacing:content:)` **[NEW]** — horizontal lazy grid
- `GridItem` **[NEW]** — column/row description for grids
  - `GridItem(.flexible(minimum:maximum:))` — flexible width
  - `GridItem(.fixed(_:))` — fixed width
  - `GridItem(.adaptive(minimum:maximum:))` — adaptive multi-column
- `List(data:children:rowContent:)` **[NEW initializer]** — hierarchical list with children key path
- `OutlineGroup(_:children:content:)` **[NEW]** — hierarchical outline view
- `DisclosureGroup(_:isExpanded:content:)` **[NEW]** — collapsible content section with optional binding
- `DisclosureGroup(isExpanded:content:label:)` **[NEW]** — custom label variant
- `Form { ... }` — settings/control container
- `Section(header:content:)` — list/form section with header
- `ForEach` — iterates flat or hierarchical collections
- `ScrollView` — scrollable container (used with Lazy stacks/grids)
- `SidebarListStyle` **[NEW / iOS 14]** — sidebar appearance with bold headers
- `Label(_:systemImage:)` **[NEW]** — icon + title combination view

## Code Highlights

Lazy stack for large scrollable collections:
```swift
ScrollView {
    LazyVStack(spacing: 0) {
        ForEach(sandwiches) { sandwich in
            HeroView(sandwich: sandwich)
        }
    }
}
```

Fixed 3-column grid:
```swift
let columns = [GridItem(spacing: 0), GridItem(spacing: 0), GridItem(spacing: 0)]
ScrollView {
    LazyVGrid(columns: columns, spacing: 0) {
        ForEach(sandwiches) { sandwich in HeroView(sandwich: sandwich) }
    }
}
```

Adaptive grid (fills available width automatically):
```swift
let columns = [GridItem(.adaptive(minimum: 300), spacing: 0)]
ScrollView { LazyVGrid(columns: columns, spacing: 0) { ... } }
```

Hierarchical list (built-in outline):
```swift
List(graphics, children: \.children) { graphic in
    GraphicRow(graphic)
}.listStyle(SidebarListStyle())
```

Custom multi-section outline with `OutlineGroup`:
```swift
List {
    ForEach(canvases) { canvas in
        Section(header: Text(canvas.name)) {
            OutlineGroup(canvas.graphics, children: \.children) { graphic in
                GraphicRow(graphic)
            }
        }
    }
}
```

`DisclosureGroup` with binding and custom label:
```swift
Form {
    DisclosureGroup(isExpanded: $areFillControlsShowing) {
        Toggle("Fill shape?", isOn: $isFilled)
        ColorRow("Fill color", color: $fillColor)
    } label: {
        Label("Fill", systemImage: "rectangle.3.offgrid.fill")
    }
}
```

## Takeaways

- Use `LazyVStack`/`LazyHStack` inside `ScrollView` for large dynamic collections — they eliminate the upfront rendering cost that causes `VStack` to block the main thread; profile first before adopting.
- `LazyVGrid` with `GridItem(.adaptive(minimum:))` creates automatically responsive multi-column grids that adapt to any screen size or window width with a single column declaration.
- Pass a `children` key path to `List` to display a full outline of hierarchical data with a single line of code; use `OutlineGroup` when you need to compose it into a custom layout.
- `DisclosureGroup` and `Form` together are the idiomatic SwiftUI pattern for inspector-style settings panels — bind `isExpanded` to `@State` to control which sections open by default.

---
_Source: WWDC20 Session 10031 page (abstract, chapter summaries, code samples, and resource links)._
