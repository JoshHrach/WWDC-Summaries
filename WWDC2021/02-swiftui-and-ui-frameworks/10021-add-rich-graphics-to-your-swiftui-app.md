# Add Rich Graphics to Your SwiftUI App
**WWDC21 · Session 10021** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10021/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session walks through multiple SwiftUI graphics APIs added or enhanced in 2021, using a gradient-building app as a running example. The topics progress from layout fundamentals (safe areas and keyboard avoidance) through visual styling (materials and vibrancy), to high-performance drawing with the all-new `Canvas` view and frame-rate-driven `TimelineView`.

The session demonstrates that SwiftUI's composable architecture means features like gestures, animations, and accessibility modifiers work consistently whether you are placing standard views or writing imperative drawing code in a `Canvas`. The result is a unified model for both declarative and imperative graphics that runs on all Apple platforms.

## Key Topics

### Safe Area and Keyboard Avoidance
- Safe areas: container safe area (device chrome, bars) and keyboard safe area (new explicit region)
- `ignoresSafeArea()` / `ignoresSafeArea(.keyboard)` — opt specific views out of safe areas
- New `background(_:ignoresSafeAreaEdges:)` modifier automatically extends backgrounds beyond safe area
- New `safeAreaInset(edge:content:)` **[NEW]** — adds auxiliary content (e.g., floating toolbars) while properly insetting the content view's safe area so scroll content isn't obscured

### Materials and Vibrancy
- New `Material` type: `.ultraThinMaterial`, `.thinMaterial`, `.regularMaterial`, `.thickMaterial`, `.ultraThickMaterial` **[NEW]**
- Apply via `.background(.thinMaterial)` — works across all platforms automatically
- Foreground styles `.secondary`, `.tertiary`, `.quaternary` automatically use Vibrancy in material contexts — no extra API needed
- `.foregroundStyle(_:)` **[NEW]** — accepts any `ShapeStyle` including colors, gradients, hierarchical styles
- Setting a color on a parent cascades tinted versions to each semantic level; gradients work too

### Canvas View
- `Canvas` **[NEW]** — high-performance imperative drawing view; similar to `drawRect` but SwiftUI-native
- Available on iOS, iPadOS, macOS, watchOS, tvOS
- Drawing closure receives `GraphicsContext` and `CGSize`
- `GraphicsContext.resolveSymbol(id:)` / `resolveText` / `resolve(_:)` — resolve SwiftUI views/images once for reuse
- Drawing operations: `context.draw(_:at:)`, `context.fill(_:with:)`, `context.stroke(...)`, `context.drawLayer(...)`
- `GraphicsContext.Shading` — wraps any SwiftUI `ShapeStyle` (color, gradient, etc.)
- Modifying context via copy: changes to a copied context don't affect the original (replaces save/restore pattern)
- `blendMode`, `opacity`, `transform` — standard drawing state properties on `GraphicsContext`
- Accessibility: `Canvas` is a single graphic; use `.accessibilityLabel` / new `.accessibilityChildren` modifier

### TimelineView
- `TimelineView(_:content:)` **[NEW]** — updates its content on a schedule; analogous to `CADisplayLink`
- `.animation` schedule — updates as fast as display refresh rate
- `.periodic(from:by:)` schedule — fixed timer interval
- Context provides current `Date` for driving time-based animations
- Combine with `Canvas` for high-performance particle systems, simulations

### drawingGroup
- `drawingGroup()` — existing modifier; flattens a view tree into a single Metal-backed layer
- Best for large numbers of graphical views; not suitable for interactive UI controls

## APIs & Frameworks

### SwiftUI
- `Canvas` **[NEW]** — imperative drawing view
- `GraphicsContext` **[NEW]** — drawing context passed to `Canvas` closure
  - `draw(_:at:anchor:)`, `draw(_:in:)` — draw resolved images/symbols
  - `fill(_:with:style:)` — fill a `Path` with a `Shading`
  - `stroke(_:with:lineWidth:)` — stroke a `Path`
  - `resolve(_:)` — resolve a SwiftUI `Image` to `GraphicsContext.ResolvedImage` **[NEW]**
  - `resolveSymbol(id:)` — resolve a tagged SwiftUI view **[NEW]**
  - `opacity`, `blendMode`, `transform`, `environment` — drawing state
  - Copy-on-modify semantics for isolated state changes
- `GraphicsContext.Shading` **[NEW]** — `color(_:)`, `linearGradient(...)`, `style(_:)`, etc.
- `TimelineView(_:content:)` **[NEW]** — schedule-driven updating view
  - `TimelineView.Context` — provides `date`, `cadence`
  - `.animation` schedule **[NEW]**
  - `.periodic(from:by:)` schedule **[NEW]**
- `safeAreaInset(edge:alignment:spacing:content:)` **[NEW]** — inset safe area for auxiliary overlay content
- `.background(_:ignoresSafeAreaEdges:)` **[NEW]** — extended background modifier
- `.ignoresSafeArea(_:edges:)` — opt out of specific safe areas
- `Material` **[NEW]** — `.ultraThinMaterial`, `.thinMaterial`, `.regularMaterial`, `.thickMaterial`, `.ultraThickMaterial`
- `.foregroundStyle(_:)` **[NEW]** — hierarchical/semantic foreground styling
- `.foregroundStyle(.primary/.secondary/.tertiary/.quaternary)` **[NEW]** — hierarchical levels with automatic vibrancy in material context
- `drawingGroup()` — flatten views to single Metal layer
- `.accessibilityChildren(children:)` **[NEW]** — specify a view hierarchy for accessibility representation of a `Canvas`

## Code Highlights

Ignore safe area for keyboard only:
```swift
ContentView()
    .ignoresSafeArea(.keyboard)
```

Apply material background with automatic edge extension:
```swift
.background(.thinMaterial)
```

Add auxiliary bottom bar without obscuring scroll content:
```swift
.safeAreaInset(edge: .bottom) {
    ControlsView()
}
```

Canvas with TimelineView for animation:
```swift
TimelineView(.animation) { timeline in
    Canvas { context, size in
        let time = timeline.date.timeIntervalSinceReferenceDate
        var resolved = context.resolve(Image(systemName: "sparkle"))
        resolved.shading = .color(.blue)
        // draw with imperative commands
        context.draw(resolved, at: CGPoint(x: size.width/2, y: size.height/2))
    }
}
```

Hierarchical foreground styles with gradient tinting:
```swift
let blueGradient = LinearGradient(colors: [.blue, .teal], startPoint: .leading, endPoint: .trailing)
VStack {
    Text("Primary").foregroundStyle(.primary)
    Text("Secondary").foregroundStyle(.secondary)
}
.foregroundStyle(blueGradient)
```

## Takeaways
- `Canvas` and `TimelineView` together enable high-performance, frame-rate-driven rendering of complex graphics and simulations entirely in SwiftUI, available on all platforms.
- `safeAreaInset` solves the common "floating bar obscures scrollable content" problem cleanly without manual geometry math.
- Materials and the updated `foregroundStyle` API bring automatic vibrancy, Dark Mode adaptation, and multi-level semantic styling without per-case code.
- `GraphicsContext`'s copy-modify pattern replaces the old save/restore idiom with a safe, value-type approach.

---
_Source: WWDC21 Session 10021 page (abstract, chapter summaries, code samples, and resource links)._
