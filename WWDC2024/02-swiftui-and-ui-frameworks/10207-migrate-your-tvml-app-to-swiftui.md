# Migrate Your TVML App to SwiftUI
**WWDC24 · Session 10207** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10207/)

_Platforms:_ tvOS 18

## Overview
TVMLKit is deprecated; SwiftUI is the preferred toolkit for tvOS 18 and beyond. This session shows developers how to recreate familiar TVMLKit/TVML UI patterns — lockups, shelves, catalogs, and search — using SwiftUI components. All four chapters follow a hands-on recipe format: here is the TVML pattern, here is how to build it in SwiftUI.

## Key Topics

**Lockups**
- A lockup is a focusable content card — image + title text stacked vertically
- In SwiftUI: a `Button` with a custom `label:` closure containing an `Image` (`.resizable().frame`) and `Text`
- Apply `.buttonStyle(.borderless)` for the standard lockup appearance on tvOS; focus effects and parallax are handled automatically by the system

**Shelves**
- A shelf is a horizontally scrolling row of lockups (the standard "related content" row)
- `ScrollView(.horizontal)` + `LazyHStack(spacing:)` + `ForEach` over your data model
- Wrap each item in a `Button` with a lockup label; the system handles focus ring and parallax on each card

**Catalogs (Landing Pages)**
- A catalog / landing page combines a hero image at top with multiple named sections of content below
- `ScrollView(.vertical)` + `LazyVStack(alignment: .leading, spacing:)` for the outer layout
- Hero: `Image` with a `LinearGradient` overlay and action buttons (`HStack` of `Button` views)
- Sections: SwiftUI `Section` with a title string and a child shelf view — sections self-organize with headers
- Use `LazyVStack` (not `VStack`) for long catalogs so off-screen sections are not rendered eagerly

**Card Buttons**
- For cards that show metadata (title, subtitle, action icons): build a custom `Button` label using `HStack` / `VStack` / `Spacer`
- `RoundedRectangle` as a background shape; metadata text and icons in a `VStack`
- Same focus engine handles highlighting, just like standard lockups

**Search**
- Tab-based search: add a `Tab("Search", systemImage: "magnifyingglass")` to a `TabView`
- Search page: `@State var searchTerm: String` + `ScrollView(.vertical)` + `LazyVGrid` with `GridItem(.flexible())`
- Attach `.searchable(text: $searchTerm)` modifier to filter the grid dynamically
- `LazyVGrid(columns:)` with `Array(repeating: GridItem(.flexible(), spacing:), count: N)` reproduces the TVML grid layout

**Navigation / Tab Bar**
- `TabView` with `Tab(title, systemImage:)` children is the direct replacement for the TVMLKit tab bar template
- New in tvOS 18: `.tabViewStyle(.sidebarAdaptable)` — the tab bar can optionally present as a sidebar on tvOS 18, matching the visionOS and iPadOS pattern

## APIs & Frameworks

**SwiftUI (tvOS)**
- `ScrollView(.horizontal)` — horizontally scrollable container for shelves
- `ScrollView(.vertical)` — vertically scrollable container for landing pages and grids
- `LazyHStack(spacing:)` — lazy horizontal stack for shelf content
- `LazyVStack(alignment:spacing:)` — lazy vertical stack for catalog sections
- `LazyVGrid(columns:)` — lazy grid for search results
- `GridItem(.flexible(), spacing:)` — flexible column definition for `LazyVGrid`
- `Button` with custom `label:` — creates a lockup; focus effects automatic on tvOS
- `.buttonStyle(.borderless)` — tvOS borderless lockup style
- `Section(title:content:)` — named content section with automatic header rendering
- `TabView` — top-level navigation container
- `Tab(_:systemImage:content:)` **[NEW]** — declarative tab definition (iOS 18 / tvOS 18 API)
- `.tabViewStyle(.sidebarAdaptable)` **[NEW]** — new tvOS 18 sidebar tab bar style
- `.searchable(text:)` — attach a search field to a view hierarchy
- `LinearGradient(stops:startPoint:endPoint:)` — gradient overlay for hero images
- `Image(_:).resizable().frame(width:height:)` — standard image sizing pattern
- `ForEach` — iterate over identifiable data to produce views
- `@State` — local state for search term and UI toggles

**TVMLKit**
- Deprecated in tvOS 18; migrate to SwiftUI

## Code Highlights

Borderless lockup button:
```swift
Button {} label: {
    Image("discovery_landscape")
        .resizable()
        .frame(width: 250, height: 375)
    Text("Borderless Portrait")
}
.buttonStyle(.borderless)
```

Standard content shelf:
```swift
ScrollView(.horizontal) {
    LazyHStack(spacing: 20) {
        ForEach(Asset.allCases) { asset in
            Button {} label: {
                // lockup image + text
            }
        }
    }
}
```

Landing page with hero and sections:
```swift
ScrollView(.vertical) {
    LazyVStack(alignment: .leading, spacing: 0) {
        VStack(alignment: .leading) {
            Text("tvOS with SwiftUI")
            Spacer().frame(height: 300)
            HStack {
                Button("Show") {}
                Button("More Info") {}
            }
        }
        Section("Movie Shelf") { MovieShelf() }
        Section("Content Cards") { CardShelf() }
    }
}
```

Search tab with grid:
```swift
@State var searchTerm: String = ""
let columns = Array(repeating: GridItem(.flexible(), spacing: 20), count: 4)

ScrollView(.vertical) {
    LazyVGrid(columns: columns) {
        ForEach(filteredResults) { item in
            Button { } label: { Text(item.title) }
        }
    }
}
.searchable(text: $searchTerm)
```

Sidebar tab view (tvOS 18):
```swift
TabView {
    Tab("Stack", systemImage: "line.3.horizontal") { StackView() }
    Tab("Search", systemImage: "magnifyingglass") { SearchView() }
}
.tabViewStyle(.sidebarAdaptable)
```

## Takeaways
- Replace every TVML lockup with a `Button` using a custom `label:` — SwiftUI handles focus highlighting, parallax, and selection sounds automatically on tvOS.
- Use `LazyHStack` inside a horizontal `ScrollView` for shelves, and `LazyVStack`/`Section` for catalogs — `Lazy` variants avoid rendering off-screen content.
- `Tab` + `TabView` is a direct replacement for the TVMLKit tab bar template; add `.tabViewStyle(.sidebarAdaptable)` for the new tvOS 18 sidebar navigation option.
- The companion sample app "Creating a tvOS media catalog app in SwiftUI" provides a full reference implementation of all the patterns covered in this session.

---
_Source: WWDC24 Session 10207 page (abstract, chapter list, code samples, and resource links)._
