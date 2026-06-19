# Meet SwiftUI spatial layout
**WWDC25 · Session 273** · [Watch](https://developer.apple.com/videos/play/wwdc2025/273/)

_Platforms:_ visionOS 26

## Overview
SwiftUI gains a native 3D layout system in visionOS 26, extending the familiar 2D layout model into depth. Previously, 3D positioning required dropping into RealityKit or using `offset(z:)` as a workaround. The new spatial layout APIs — `SpatialContainer`, `spatialOverlay`, `rotation3DLayout`, and depth alignment guides — let SwiftUI developers compose views along the Z axis with the same declarative ergonomics they use for X and Y.

The session walks through how depth alignment guides work (analogous to `HorizontalAlignment` and `VerticalAlignment`), how `DepthAlignment` and the `DepthAlignmentID` protocol enable custom alignment guides in the Z dimension, and how `SpatialContainer` acts as the 3D equivalent of a `ZStack` with explicit depth semantics.

`GeometryReader3D` and `scaledToFit3D()` round out the spatial sizing story, enabling views that respond to their 3D bounding volume rather than just their 2D frame.

## Key Topics

### Depth Alignment System
`DepthAlignment` is the new alignment type for the Z axis. It works like `HorizontalAlignment` — views in a layout can declare their depth alignment guide, and the layout container positions them accordingly. Custom alignment guides are defined by conforming to `DepthAlignmentID`.

### `depthAlignment()` Modifier
Applied to `Layout` types (e.g., `VStackLayout`, `HStackLayout`), this modifier specifies how child views are aligned along depth when they have different Z extents. Analogous to `alignment:` parameter on `VStack`.

### `rotation3DLayout()`
A layout modifier that rotates child views in 3D space as part of the layout pass — not as a post-layout visual transform. This means the layout system accounts for the rotated extents when placing adjacent views, enabling correct depth-aware compositions.

### `SpatialContainer`
The primary 3D layout primitive: a container that stacks child views along the Z axis with explicit depth spacing and alignment. It is analogous to `ZStack` but with depth semantics — each child occupies a distinct Z position determined by the container's layout rules.

### `spatialOverlay()`
Analogous to `.overlay()` in 2D — places a view in front of (or behind) another view in 3D space. The alignment is specified using `DepthAlignment` rather than the 2D `Alignment` type.

### Model3DAsset and `scaledToFit3D()`
`Model3DAsset` loads a USDZ/Reality file as a SwiftUI-compatible 3D asset. `scaledToFit3D()` scales the asset to fit its proposed 3D bounding volume, analogous to `scaledToFit()` for images. `GeometryReader3D` reads the proposed and actual 3D bounds.

## APIs & Frameworks

- **SpatialContainer** **[NEW]** — 3D Z-axis layout container
- **spatialOverlay()** **[NEW]** — 3D overlay placement modifier
- **rotation3DLayout()** **[NEW]** — layout-aware 3D rotation modifier
- **DepthAlignment** **[NEW]** — Z-axis alignment type (analogous to HorizontalAlignment)
- **DepthAlignmentID protocol** **[NEW]** — custom Z-axis alignment guide definition
- **depthAlignment()** modifier on Layout types **[NEW]** — specifies Z alignment for stacked views
- **alignmentGuide(.depthPodium)** — example built-in depth alignment guide (podium/stage metaphor)
- **GeometryReader3D** — reads proposed and actual 3D bounding volume
- **scaledToFit3D()** **[NEW]** — scales a 3D asset to fit its proposed bounding volume
- **Model3DAsset** — SwiftUI-compatible 3D asset loader (USDZ/Reality)
- **ZStack** — extended with depth semantics in spatial contexts

## Code Highlights

```swift
// SpatialContainer stacks views along the Z axis
SpatialContainer(depth: 60) {
    RoundedRectangle(cornerRadius: 12)
        .frame(width: 300, height: 200)
    Model3DAsset(named: "trophy.usdz")
        .scaledToFit3D()
        .frame(depth: 60)
        .depthAlignment(.front)
}
```

```swift
// Custom depth alignment guide
extension DepthAlignment {
    static let podium = DepthAlignment(DepthPodiumAlignment.self)
}

private struct DepthPodiumAlignment: DepthAlignmentID {
    static func defaultValue(in context: ViewDimensions3D) -> CGFloat {
        context.depth / 4  // 25% from front
    }
}
```

```swift
// spatialOverlay places a badge in front of a card
cardView
    .spatialOverlay(depthAlignment: .front) {
        BadgeView()
            .offset(z: 20)
    }
```

## Takeaways

- `SpatialContainer` is the correct replacement for `ZStack` when you need true depth composition — use it any time child views have Z extents that matter.
- The `DepthAlignment` system is intentionally parallel to the 2D alignment system; existing knowledge transfers directly.
- `rotation3DLayout()` is layout-participating, unlike `rotation3DEffect()` — use it when the rotated view must affect the layout of siblings.
- `scaledToFit3D()` and `GeometryReader3D` are necessary for correctly sizing USDZ model assets within SwiftUI layouts.

---
_Source: WWDC25 Session 273 page (abstract, chapter summaries, code samples, and resource links)._
