# What's new in SwiftUI
**WWDC22 · Session 10052** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10052/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
SwiftUI received its largest update since launch in 2022, spanning new data visualization, navigation architecture, window management, form controls, sharing, and layout capabilities. The session introduced Swift Charts — a brand-new declarative framework built on SwiftUI's design principles for creating rich, adaptive data visualizations across all platforms.

Navigation received a complete overhaul with `NavigationStack` and `NavigationSplitView`, replacing the older `NavigationView` with data-driven, type-safe APIs. Window management on macOS and iPadOS was expanded with the `Window` scene type, `MenuBarExtra`, `presentationDetents`, and environment-based `openWindow` actions.

Controls became deeper and more customizable: `TextField` gained a vertical `axis` and range-based `lineLimit`, `MultiDatePicker` landed on iOS, mixed-state `Toggle` and `Picker` arrived, tables came to iPadOS, toolbars gained user customization, and `ShareLink`/`PhotosPicker`/`Transferable` provided a unified sharing model. Layout gained the `Grid` container and the `Layout` protocol for fully custom layout algorithms.

## Key Topics

### Swift Charts
A new declarative framework using the same design patterns as SwiftUI (`Chart`, `BarMark`, `LineMark`, `RuleMark`, `PointMark`, etc.) for building charts that automatically handle Dark Mode, Dynamic Type, localization, and all Apple platforms.

### Navigation — NavigationStack and NavigationSplitView
`NavigationStack` replaces `NavigationView` for push-pop navigation and introduces value-based `NavigationLink` with `navigationDestination(for:)` for type-safe, state-driven navigation. `NavigationSplitView` provides two- and three-column layouts and automatically collapses to a stack on compact size classes.

### Scene and Window APIs
New `Window` scene type for single unique windows; `MenuBarExtra` for macOS menu bar apps; `presentationDetents` for resizable sheets on iOS; `openWindow` environment action for programmatic window presentation; and new `defaultPosition`, `defaultSize` window modifiers.

### Forms and Controls
New `formStyle(.grouped)` matches the macOS Ventura System Settings visual style. `LabeledContent` for aligning custom views in forms. Vertical `TextField` axis, range-based `lineLimit`, `MultiDatePicker`, mixed-state `Toggle` and `Picker`, `Stepper` with format parameter, `Stepper` on watchOS, and `accessibilityQuickAction`.

### Sharing — PhotosPicker, ShareLink, and Transferable
`PhotosPicker` is a new privacy-safe cross-platform photo picker. `ShareLink` presents the system share sheet from anywhere in SwiftUI (now including watchOS 9). Both build on the new `Transferable` protocol for Swift-first data transfer with `dropDestination`.

### Graphics and ShapeStyle
`Color.gradient` for subtle derived gradients, `ShapeStyle.shadow` modifier, and new SF Symbols integration. `withAnimation` now produces beautiful text weight/style interpolation.

### Layout — Grid and Layout Protocol
`Grid` + `GridRow` + `gridCellColumns` for two-dimensional layouts. New `Layout` protocol for custom first-class layout algorithms. `AnyLayout` for runtime switching between layouts.

## APIs & Frameworks

**Swift Charts (New Framework)** **[NEW]**
- `Chart` — top-level chart container view
- `BarMark` — bar chart mark **[NEW]**
- `LineMark` — line chart mark **[NEW]**
- `PointMark` — point/scatter mark **[NEW]**
- `RuleMark` — reference line mark **[NEW]**
- `.foregroundStyle(by:)` — grouping marks by data field
- `.symbol(by:)` — per-series symbol shapes
- `.annotation(position:alignment:)` — chart annotations

**SwiftUI — Navigation** **[NEW]**
- `NavigationStack` **[NEW]** — data-driven push-pop navigation
- `NavigationStack(path:)` — explicit path binding
- `NavigationLink(value:)` — value-based link **[NEW]**
- `.navigationDestination(for:destination:)` — destination registration **[NEW]**
- `NavigationSplitView` **[NEW]** — two- and three-column adaptive layout
- `NavigationPath` — type-erased navigation path **[NEW]**

**SwiftUI — Scenes and Windows**
- `Window` scene type **[NEW]**
- `MenuBarExtra` **[NEW]** — macOS menu bar app/window
- `.menuBarExtraStyle(.window)` **[NEW]**
- `\.openWindow` environment action **[NEW]**
- `.defaultPosition(_:)` window modifier **[NEW]**
- `.defaultSize(width:height:)` window modifier **[NEW]**
- `.windowResizability(_:)` modifier **[NEW]**
- `.keyboardShortcut(_:)` on `Window`
- `presentationDetents(_:)` **[NEW]** — resizable sheet stop points
- `.presentationDragIndicator(_:)` **[NEW]**

**SwiftUI — Forms and Controls**
- `formStyle(.grouped)` **[NEW]** — macOS Ventura System Settings style
- `LabeledContent` **[NEW]** — aligned label/value or label/custom-view pairs
- `TextField(_:text:axis:)` — vertical axis support **[NEW]**
- `.lineLimit(_:reservesSpace:)` — space-reserving line limit **[NEW]**
- `.lineLimit(_ range: ClosedRange<Int>)` — range-based line limit **[NEW]**
- `MultiDatePicker` **[NEW]** — non-contiguous multi-date selection
- `Toggle(isOn: [Binding<Bool>])` — mixed-state aggregate toggle **[NEW]**
- `Picker(selection: [Binding<T>])` — mixed-state picker **[NEW]**
- `Stepper(_:value:format:)` — formatted stepper **[NEW]**
- `Stepper` on watchOS **[NEW]**
- `.accessibilityQuickAction(style:content:)` **[NEW]**
- `Table` on iPadOS **[NEW]**
- `TableColumn`
- `.contextMenu(forSelectionType:menu:)` — selection-based context menus **[NEW]**
- `.toolbar(id:)` — customizable toolbars with identifiers **[NEW]**
- `ToolbarItem(id:placement:showsByDefault:)` — identifiable toolbar items **[NEW]**
- `.toolbarRole(.editor)` **[NEW]**
- `ToolbarItemPlacement.secondaryAction` **[NEW]**
- `.searchable(text:tokens:suggestions:)` — search with tokens **[NEW]**
- `.searchScopes(_:)` **[NEW]**
- `.searchCompletion(_:)` **[NEW]**

**SwiftUI — Sharing**
- `PhotosPicker` **[NEW]** — cross-platform privacy-safe photo/video picker
- `PhotosPickerItem` **[NEW]**
- `ShareLink` **[NEW]** — system share sheet view (including watchOS 9)
- `Transferable` protocol **[NEW]**
- `.dropDestination(for:action:)` **[NEW]**

**SwiftUI — Graphics and ShapeStyle**
- `Color.gradient` property **[NEW]** — derived subtle gradient
- `ShapeStyle.shadow(_:)` modifier **[NEW]**
- Text weight/style animation (automatic via `withAnimation`) **[NEW]**
- Xcode 14 Preview variants (appearance, type size, orientation) **[NEW]**

**SwiftUI — Layout**
- `Grid` container **[NEW]**
- `GridRow` **[NEW]**
- `.gridCellColumns(_:)` modifier **[NEW]**
- `Layout` protocol **[NEW]** — custom first-class layout type
- `AnyLayout` **[NEW]** — type-erased layout for runtime switching

## Code Highlights

Value-based NavigationStack with programmatic path control:
```swift
NavigationStack(path: $selectedFoodItems) {
    List(foodItems) { item in
        NavigationLink(value: item) { FoodRow(food: item) }
    }
    .navigationDestination(for: FoodItem.self) { item in
        FoodDetailView(item: item, path: $selectedFoodItems)
    }
}
```

New Window scene with openWindow environment action:
```swift
Window("Party Budget", id: "budget") { Text("Budget View") }
    .keyboardShortcut("0")
    .defaultPosition(.topLeading)
    .defaultSize(width: 220, height: 250)
// Usage:
@Environment(\.openWindow) var openWindow
openWindow(id: "budget")
```

Resizable sheets with detents:
```swift
.sheet(isPresented: $presented) {
    Text("Budget View")
        .presentationDetents([.height(250), .medium])
        .presentationDragIndicator(.visible)
}
```

Mixed-state aggregate Toggle:
```swift
Toggle("All Decorations", isOn: [$includeBalloons, $includeConfetti,
                                  $includeInflatables, $includeBlowers])
```

## Takeaways
- `NavigationStack` and `NavigationSplitView` replace `NavigationView` with fully data-driven, type-safe, programmatically controllable navigation.
- Swift Charts brings declarative, SwiftUI-style data visualization to all Apple platforms out of the box.
- `ShareLink`, `PhotosPicker`, and the `Transferable` protocol unify sharing and data transfer in a Swift-native API.
- The new `Layout` protocol gives developers the same layout power Apple uses for stacks and grids, enabling fully custom layout algorithms as first-class SwiftUI types.

---
_Source: WWDC22 Session 10052 page (abstract, chapter summaries, code samples, and resource links)._
