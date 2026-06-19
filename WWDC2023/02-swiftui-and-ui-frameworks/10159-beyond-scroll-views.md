# Beyond Scroll Views
**WWDC23 · Session 10159** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10159/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
iOS 17 significantly expands SwiftUI's `ScrollView` capabilities with new APIs for controlling margins, snap-scroll targets, programmatic scroll position, and visual effects tied to scroll position. The session walks through building a polished gallery feature using these new APIs, progressing from basic scroll view setup through content margins, view-aligned paging, programmatic navigation, and scroll-driven visual transitions.

The APIs introduced span four areas: safe area and content margin control, scroll target behaviors (paging and view-aligned snapping), the `scrollPosition` binding for bidirectional position control, and the new `scrollTransition` modifier for scroll-driven visual effects. A new `containerRelativeFrame` modifier also replaces the need for `GeometryReader` in many layout scenarios.

## Key Topics

### Margins and Safe Area
- `safeAreaPadding(_:)` — adds padding to the safe area (not the view), allowing the scroll view to extend edge-to-edge while content remains inset
- `contentMargins(_:_:)` — independently insets scroll content vs. scroll indicators; replaces safe-area tricks for this use case

### Scroll Targets and Paging
- `scrollTargetBehavior(.paging)` — whole-page snapping at the ScrollView's container size
- `scrollTargetBehavior(.viewAligned)` — snaps to individual child views; better for mixed screen sizes
- `scrollTargetLayout()` — applied to a `LazyHStack`/`LazyVStack` to mark all children as scroll targets; required for lazy layouts because views outside the visible region haven't been created yet
- `.scrollTarget()` — marks an individual view as a scroll target (for non-lazy layouts)
- `ScrollTargetBehavior` protocol — conform custom types and implement `updateTarget(_:context:)` for fully custom snap logic

### Container Relative Frame
- `containerRelativeFrame(_:count:spacing:)` — sizes a view relative to its container (ScrollView, NavigationSplitView column, window) with grid-like column counts; replaces `GeometryReader` for this pattern
- `\.horizontalSizeClass` environment value — now available on all platforms (previously iOS/iPadOS only)

### Scroll Position
- `scrollPosition(id:)` — binds a `Binding<ID?>` to the scroll view's current leading visible item identity; writing to the binding programmatically scrolls; reading it reflects current scroll position
- Replaces `ScrollViewReader` for many common use cases
- Works together with `scrollTargetLayout()` to determine which view's ID is "current"

### Scroll Indicators
- `scrollIndicators(.never)` — hides indicators even when a mouse is connected (use carefully; provide an alternative for mouse users)
- Default behavior: hides for trackpad/touch, shows for mouse input

### Scroll Transitions
- `scrollTransition(axis:)` — applies a `VisualEffect` closure to a view as it moves through the scroll view's visible region
- `ScrollTransitionPhase` — `.identity` (view is centered/fully visible), and non-identity phases (view is entering/leaving)
- `phase.isIdentity` — `Bool` distinguishing identity from transition phases
- `VisualEffect` protocol — safe subset of view modifiers for layout-dependent visual changes: `scaleEffect`, `rotationEffect`, `offset`, etc.
- Modifiers that change content size (e.g., `font`) are not allowed inside `scrollTransition`

## APIs & Frameworks

### SwiftUI — Scroll View APIs (all **[NEW]**)
- `ScrollView` — existing; content offset concept formalized
- `.safeAreaPadding(_:)` modifier **[NEW]**
- `.contentMargins(_:_:for:)` modifier — insets `.scrollContent` or `.scrollIndicators` independently **[NEW]**
- `ScrollTargetBehavior` protocol **[NEW]**
  - `PagingScrollTargetBehavior` (`.paging`) **[NEW]**
  - `ViewAlignedScrollTargetBehavior` (`.viewAligned`) **[NEW]**
  - `updateTarget(_:context:)` — required method for custom conformances
  - `ScrollTargetBehaviorContext` — provides container size and velocity info
- `.scrollTargetBehavior(_:)` modifier **[NEW]**
- `.scrollTargetLayout()` modifier — marks layout container's children as targets **[NEW]**
- `.scrollTarget(isEnabled:)` modifier — marks individual view as a scroll target **[NEW]**
- `.scrollPosition(id:)` modifier **[NEW]**
- `.scrollIndicators(_:axes:)` — existing; `.never` value **[clarified behavior]**
- `ScrollTransitionPhase` enum **[NEW]**
  - `.identity`
  - `.topLeading` / `.bottomTrailing`
  - `phase.isIdentity: Bool`
  - `phase.value: Double` — interpolated value for custom effects
- `.scrollTransition(_:axis:transition:)` modifier **[NEW]**
- `VisualEffect` protocol **[NEW]**
  - `.scaleEffect(x:y:)`
  - `.rotationEffect(_:)`
  - `.offset(x:y:)`
  - `.opacity(_:)`
  - `.blur(radius:)`

### SwiftUI — Layout APIs
- `.containerRelativeFrame(_:count:spacing:alignment:)` **[NEW]**
- `.containerRelativeFrame(_:)` — sizes to container width/height **[NEW]**
- `\.horizontalSizeClass` — now available on macOS, tvOS, watchOS (was iOS/iPadOS only) **[NEW]**

## Code Highlights

View-aligned scroll snap with content margins and scroll position binding:
```swift
ScrollView(.horizontal) {
    LazyHStack(spacing: hSpacing) {
        ForEach(palettes) { palette in
            GalleryHeroView(palette: palette)
        }
    }
    .scrollTargetLayout()
}
.contentMargins(.horizontal, hMargin)
.scrollTargetBehavior(.viewAligned)
.scrollPosition(id: $mainID)
.scrollIndicators(.never)
```

Scroll-driven scale transition:
```swift
.scrollTransition(axis: .horizontal) { content, phase in
    content.scaleEffect(
        x: phase.isIdentity ? 1.0 : 0.80,
        y: phase.isIdentity ? 1.0 : 0.80
    )
}
```

Container relative frame for adaptive column layout:
```swift
.containerRelativeFrame([.horizontal], count: columns, spacing: hSpacing)
```

## Takeaways
- `.contentMargins` replaces the safe-area workaround for separately insetting scroll content vs. indicators.
- `.scrollTargetBehavior(.viewAligned)` + `.scrollTargetLayout()` is the right pattern for horizontal carousels on any screen size.
- `.scrollPosition(id:)` provides bidirectional scroll position control with no `ScrollViewReader` boilerplate — write to scroll programmatically, read to know what's visible.
- `.scrollTransition` + `VisualEffect` enables scroll-driven visual effects safely, without the layout side-effects of arbitrary view modifiers inside scroll content.

---
_Source: WWDC23 Session 10159 page (abstract, chapter summaries, code samples, and resource links)._
