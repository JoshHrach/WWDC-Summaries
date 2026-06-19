# Compose Custom Layouts with SwiftUI
**WWDC22 · Session 10056** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10056/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
This session introduces several new SwiftUI layout tools that together cover a wide spectrum from simple two-dimensional grids to fully custom layout containers. The presenter walks through building a polling app with a leaderboard, equal-width voting buttons, and an animated radial ranking display — using each new API in context.

The centerpiece is the new `Layout` protocol, which allows developers to define their own general-purpose or narrowly targeted layout containers that participate directly in SwiftUI's layout engine. This eliminates the need to misuse `GeometryReader` for measurement tasks and provides a clean, loop-safe alternative for size-dependent arrangement.

The session also introduces `ViewThatFits` for automatic view selection based on available space, and `AnyLayout` for seamlessly transitioning between different layout types with animations.

## Key Topics

### Grid — Two-Dimensional Static Layouts
The new `Grid` container (distinct from `LazyHGrid`/`LazyVGrid`) loads all views at once and automatically sizes rows and columns to accommodate their content in both dimensions. `GridRow` groups views into rows; views outside a `GridRow` span the entire grid width. Column alignment can be set globally on the `Grid` or per-column via `gridColumnAlignment(_:)`, and a single view can span multiple columns with `gridCellColumns(_:)`.

### Layout Protocol — Custom Containers
Conforming a type to `Layout` requires implementing two methods: `sizeThatFits(proposal:subviews:cache:)` and `placeSubviews(in:proposal:subviews:cache:)`. Subviews are accessed only through `LayoutSubview` proxies, which expose `sizeThatFits(_:)`, `spacing`, and `place(at:anchor:proposal:)`. A bidirectional `cache` parameter enables sharing intermediate calculations across method calls for performance. Custom layout values can be attached to subviews via `LayoutValueKey` and `layoutValue(key:value:)`, and read inside layout methods via subscript on each proxy.

### ViewThatFits
`ViewThatFits` tries each provided view in order and selects the first one that fits the available space, making it ideal for adaptive layouts that need to fall back from horizontal to vertical arrangements under constrained widths.

### AnyLayout — Animated Layout Transitions
`AnyLayout` wraps any `Layout`-conforming type and allows the active layout to be switched at runtime. Because the view hierarchy's structural identity is preserved, SwiftUI animates the transition smoothly when combined with `.animation(_:value:)`.

## APIs & Frameworks

### SwiftUI — Grid
- `Grid(alignment:horizontalSpacing:verticalSpacing:content:)` **[NEW]** — two-dimensional non-lazy grid container
- `GridRow(alignment:content:)` **[NEW]** — defines a row within a `Grid`
- `.gridColumnAlignment(_:)` **[NEW]** — sets alignment for all cells in a column
- `.gridCellColumns(_:)` **[NEW]** — makes a cell span multiple columns
- `.gridCellAnchor(_:)` **[NEW]** — positions a cell within its allocated space

### SwiftUI — Layout Protocol
- `Layout` protocol **[NEW]** — defines a custom layout container
- `Layout.sizeThatFits(proposal:subviews:cache:)` **[NEW]** — returns the container's size given a size proposal
- `Layout.placeSubviews(in:proposal:subviews:cache:)` **[NEW]** — places subviews within the container bounds
- `Layout.Subviews` / `LayoutSubview` **[NEW]** — proxy types for accessing subview measurements and placement
- `LayoutSubview.sizeThatFits(_:)` **[NEW]** — asks a subview for its size given a `ProposedViewSize`
- `LayoutSubview.spacing` — `ViewSpacing` instance with directional spacing preferences
- `ViewSpacing.distance(to:along:)` **[NEW]** — resolves spacing between two adjacent views
- `LayoutSubview.place(at:anchor:proposal:)` **[NEW]** — places a subview at a point with an anchor and size proposal
- `ProposedViewSize` **[NEW]** — a potentially-nil size proposal passed during layout
- `ProposedViewSize.unspecified` **[NEW]** — asks for a view's ideal size
- `ProposedViewSize.replacingUnspecifiedDimensions()` **[NEW]** — replaces nil dimensions with default values
- `LayoutValueKey` protocol **[NEW]** — declares a per-subview custom value type
- `.layoutValue(key:value:)` **[NEW]** — attaches a custom layout value to a view

### SwiftUI — ViewThatFits
- `ViewThatFits(in:content:)` **[NEW]** — selects the first child view that fits the available space

### SwiftUI — AnyLayout
- `AnyLayout` **[NEW]** — type-erased `Layout` wrapper enabling runtime layout switching
- `HStackLayout` **[NEW]** — `Layout`-conforming version of `HStack`
- `VStackLayout` **[NEW]** — `Layout`-conforming version of `VStack`
- `ZStackLayout` **[NEW]** — `Layout`-conforming version of `ZStack`

## Code Highlights

Defining a custom equal-width horizontal layout:
```swift
struct MyEqualWidthHStack: Layout {
    func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout Void) -> CGSize {
        let maxSize = maxSize(subviews: subviews)
        let spacing = spacing(subviews: subviews)
        let totalSpacing = spacing.reduce(0) { $0 + $1 }
        return CGSize(width: maxSize.width * CGFloat(subviews.count) + totalSpacing, height: maxSize.height)
    }
    func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout Void) {
        let maxSize = maxSize(subviews: subviews)
        let placementProposal = ProposedViewSize(width: maxSize.width, height: maxSize.height)
        var x = bounds.minX + maxSize.width / 2
        for index in subviews.indices {
            subviews[index].place(at: CGPoint(x: x, y: bounds.midY), anchor: .center, proposal: placementProposal)
            x += maxSize.width + spacing(subviews: subviews)[index]
        }
    }
}
```

Switching layouts with animation using `AnyLayout`:
```swift
let layout = isThreeWayTie ? AnyLayout(HStackLayout()) : AnyLayout(MyRadialLayout())
layout {
    ForEach(pets) { pet in
        Avatar(pet: pet).rank(rank(pet))
    }
}
.animation(.default, value: pets)
```

## Takeaways
- `Grid` is ideal for static two-dimensional layouts where both row height and column width should be determined automatically — use it instead of `LazyVGrid`/`LazyHGrid` when content is not scrollable.
- The `Layout` protocol is the correct tool for measurement-dependent arrangements, replacing `GeometryReader` hacks that can cause layout loops.
- `ViewThatFits` makes adaptive layouts concise: simply list fallback layouts in order and SwiftUI picks the first one that fits.
- `AnyLayout` enables smooth animated transitions between entirely different layout strategies without replacing view identity.

---
_Source: WWDC22 Session 10056 page (abstract, chapter summaries, code samples, and resource links)._
