# Demystify SwiftUI Containers
**WWDC24 · Session 10146** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10146/)

_Platforms:_ iOS 18, iPadOS 18, macOS 15, tvOS 18, watchOS 11, visionOS 2

## Overview
iOS 18 unlocks the container composition model that powers SwiftUI's own `List`, `TabView`, `Group`, and `ForEach` for third-party developers. A new family of APIs — `ForEach(subviews:)`, `Group(subviewsOf:)`, and `containerValue` — lets custom container views iterate over their content children as discrete subviews, read per-subview values, and compose them into any layout. This makes it possible to build custom `List`-style or `TabView`-style components that are opaque from the outside but fully introspectable from the inside.

The session walks through building a card stack container from scratch, progressively adding support for custom section headers, per-card badge values, and dynamic pinning behavior, demonstrating each new API in isolation before composing them.

## Key Topics
- **`ForEach(subviews:content:)`** — iterate over the resolved subview trees of a `content` view, yielding a typed `Subview` proxy for each; works with `Group`, `if/else`, and any SwiftUI view builder content.
- **`Group(subviewsOf:transform:)`** — alternative to `ForEach` for transforming all subviews as a collection; useful when the whole set must be inspected before rendering.
- **`containerValue(_:_:)`** — a new modifier that attaches a strongly-typed value (keyed by a `ContainerValueKey`) to a subview; readable from the parent container via `Subview.containerValues`.
- **`ContainerValueKey` protocol** — define custom keys with a `defaultValue`; analogous to `PreferenceKey` but designed for container-to-child communication in both directions.
- **Section support** — `ForEach(subviews:)` and `Group(subviewsOf:)` transparently handle `Section` children, exposing header/footer content as part of the subview tree.

## APIs & Frameworks

**SwiftUI**
- **[NEW]** `ForEach(subviews: content, content: (Subview) -> some View)` — iterate over subviews of a `@ViewBuilder` content block; `Subview` is the proxy type
- **[NEW]** `Group(subviewsOf: content, transform: ([Subview]) -> some View)` — access all subviews as a `[Subview]` array for collection-level operations
- **[NEW]** `Subview` — strongly-typed view proxy representing a resolved child view
  - `Subview.id` — stable identity across redraws
  - `Subview.containerValues` — `ContainerValues` struct holding all `containerValue` data attached to this subview
  - `Subview` conforms to `View` — pass it directly to layout containers
- **[NEW]** `ContainerValues` — keyed collection of per-subview values; access via `containerValues[MyKey.self]`
- **[NEW]** `ContainerValueKey` protocol — define a key: `struct BadgeKey: ContainerValueKey { static let defaultValue = 0 }`
- **[NEW]** `.containerValue(_:_:)` modifier — attach a `ContainerValueKey`-keyed value to a view; readable by the enclosing custom container
- **[NEW]** `ContainerValues` extension pattern — extend `ContainerValues` with a computed property using a custom key for ergonomic access: `var badge: Int { get { self[BadgeKey.self] } set { self[BadgeKey.self] = newValue } }`
- `Section` — unchanged; now transparently introspectable by `ForEach(subviews:)` and `Group(subviewsOf:)`
- `Layout` protocol — unchanged; custom layouts continue to work; `ForEach(subviews:)` feeds subviews into layout containers

## Code Highlights
Build a custom card stack that reads per-card badge values:

```swift
struct CardStack<Content: View>: View {
    @ViewBuilder var content: Content

    var body: some View {
        ForEach(subviews: content) { subview in
            let badge = subview.containerValues.badge
            ZStack(alignment: .topTrailing) {
                subview
                if badge > 0 {
                    Text("\(badge)").badge()
                }
            }
        }
    }
}

// Usage:
CardStack {
    CardView("Alpha")
        .containerValue(\.badge, 3)
    CardView("Beta")
}
```

Define and extend a custom container value key:

```swift
struct BadgeKey: ContainerValueKey {
    static let defaultValue: Int = 0
}
extension ContainerValues {
    var badge: Int {
        get { self[BadgeKey.self] }
        set { self[BadgeKey.self] = newValue }
    }
}
```

## Takeaways
- `ForEach(subviews:)` is the idiomatic way to build custom containers in iOS 18 — use it whenever you need to transform or lay out a variable number of child views passed by the caller.
- `containerValue` / `ContainerValueKey` replaces the `PreferenceKey` workaround developers previously used to attach metadata to child views; it is simpler and does not require a preference reduction.
- `Section` children are exposed as flat subviews with `header`/`footer` content available on the `Subview`; you do not need to special-case sections in your container.
- Combine `Group(subviewsOf:)` with standard Swift collection APIs (`first`, `last`, `enumerated`, `prefix`) for index-aware rendering like "pin the first card to the top."

---
_Source: WWDC24 Session 10146 page (abstract, chapter summaries, code samples, and resource links)._
