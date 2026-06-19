# Update Your App for watchOS 10
**WWDC23 · Session 10031** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10031/)

_Platforms:_ watchOS 10

## Overview
This code-along session walks through updating an existing watchOS app (Backyard Birds) from the watchOS 9 SDK to take full advantage of the watchOS 10 redesign. The session focuses on four major structural and visual changes: adopting `NavigationSplitView` for a source list + detail pattern, replacing flat scrolling lists with a vertical-paging `TabView`, using new toolbar placement options for contextual controls, and applying per-container background gradients and materials for glanceability.

The watchOS 10 SDK brings automatic adoption of large scroll-transitioning titles, navigation bar blur materials, and updated toolbar button styling with zero code changes. From there, the code-along demonstrates targeted SwiftUI changes that restructure app navigation around the Digital Crown, enabling users to launch directly into the most important content and quickly switch contexts without multiple taps.

Materials, new in watchOS 10 for watch apps, and container background gradients are used to surface contextual information — such as low food/water levels — at a glance and to distinguish foreground from background elements with visual depth.

## Key Topics

### Building Against the watchOS 10 SDK
- Rebuilding against the watchOS 10 SDK automatically adopts large title scroll transitions, navigation bar blur materials, and updated toolbar button styling.
- Modal presentations gain a blurred background material for free.

### NavigationSplitView
- Replace `NavigationStack` + `navigationDestination` with `NavigationSplitView` to present the detail view (backyard detail) directly on launch.
- Use the new `List(data:selection:)` initializer (watchOS 10) with a `@State` selection binding to drive which detail is shown.
- The source list remains accessible via the source list button, with a full-screen transition animation to the detail.

### Vertical TabView
- Replace a long scrolling `List` with a `TabView` using `.tabViewStyle(.verticalPage)` to create distinct full-screen pages navigable by the Digital Crown.
- Each tab receives a `navigationTitle` displayed as the tab header.
- Place scrollable content (e.g., a `List`) in the last tab when possible to avoid conflicting scroll directions.
- Split related content (food gauge, water gauge) into separate tabs for clearer visual separation.

### Toolbar with New Placement Options
- `ToolbarItemGroup(placement: .bottomBar)` **[NEW]** places controls in a persistent bottom bar, consistent across all watch screen sizes.
- Use `Spacer()` inside `ToolbarItemGroup` to position buttons (e.g., trailing corner).

### Container Background Gradients
- `.containerBackground(_:for:)` **[NEW]** applies a per-container background gradient using a `ShapeStyle` and a `ContainerBackgroundPlacement` (e.g., `.tabView`).
- Use computed color properties to reactively change the background gradient based on data state (e.g., red when levels are low).
- Pass the same color to gauge tint and container background for a cohesive visual.

### Materials
- `Material` is new to watchOS in watchOS 10 (previously available on other platforms).
- `.background(Material.ultraThin, in: RoundedRectangle(cornerRadius:))` and `.background(.ultraThinMaterial, in: .circle)` add depth and visual distinction.
- Force light material variants with `.environment(\.colorScheme, .light)` when the underlying content is light/colorful.

## APIs & Frameworks

- `NavigationSplitView` **[NEW on watchOS]** — source list + detail navigation container
- `List(_:selection:)` **[NEW on watchOS 10]** — list initializer with selection binding driving `NavigationSplitView` detail
- `TabView` — existing container, now supports vertical paging on watchOS
- `.tabViewStyle(.verticalPage)` **[NEW]** — enables Digital Crown-driven vertical pagination in `TabView`
- `.containerBackground(_:for:)` **[NEW]** — applies a background gradient scoped to a container (e.g., tab view page)
- `ContainerBackgroundPlacement.tabView` **[NEW]** — placement value for tab view container backgrounds
- `ToolbarItemGroup(placement: .bottomBar)` **[NEW placement]** — places toolbar items in the watch bottom bar
- `ToolbarItemGroup` — groups toolbar items with a shared placement
- `Material` (watchOS) **[NEW on watchOS]** — blur materials available in watch apps for the first time
- `Material.ultraThin` — lightest material variant
- `.ultraThinMaterial` — material shorthand usable as `ShapeStyle`
- `.environment(\.colorScheme, .light)` — forces light color scheme for material rendering
- `Color.gradient` — generates a gradient from a color (existing, used with `containerBackground`)
- `.navigationTitle(_:)` — sets the tab/page title within a vertical `TabView`
- `Gauge` — SwiftUI gauge view used for food/water level display
- `Label(_:systemImage:)` — standard label for toolbar buttons
- `SwiftUI` — primary framework throughout

## Code Highlights

```swift
// NavigationSplitView with List selection
NavigationSplitView {
    List(backyardsData.backyards, selection: $selectedBackyard) { backyard in
        BackyardCell(backyard: backyard)
    }
    .listStyle(.carousel)
} detail: {
    if let selectedBackyard {
        BackyardView(backyard: selectedBackyard)
    } else {
        BackyardUnavailableView()
    }
}

// Vertical TabView with container backgrounds
TabView {
    TodayView()
        .navigationTitle("Today")
        .containerBackground(Color.accentColor.gradient, for: .tabView)
    HabitatGaugeView(level: $waterLevel, habitatType: .water, tintColor: waterColor)
        .navigationTitle("Water")
        .containerBackground(waterColor.gradient, for: .tabView)
    List { VisitorView().navigationTitle("Visitors") }
}
.tabViewStyle(.verticalPage)

// Bottom bar toolbar
.toolbar {
    ToolbarItemGroup(placement: .bottomBar) {
        Spacer()
        Button { level = min(100, level + 5) } label: { Label("Add", systemImage: "plus") }
    }
}
```

## Takeaways
- `NavigationSplitView` on watchOS 10 enables launching directly to the most important detail screen, reducing taps and focusing the experience.
- Vertical `TabView` with `.verticalPage` style maps perfectly to the Digital Crown, breaking up long lists into full-screen pages with distinct purposes.
- `containerBackground` and `Material` are the primary tools for adding glanceability and visual hierarchy in watchOS 10 apps.
- Building against the watchOS 10 SDK provides several free visual improvements (large title transitions, navigation bar blur, toolbar styling) with no code changes required.

---
_Source: WWDC23 Session 10031 page (abstract, chapter summaries, code samples, and resource links)._
