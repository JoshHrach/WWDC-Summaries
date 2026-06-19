# Elevate Your Windowed App for Spatial Computing
**WWDC23 · Session 10110** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10110/)

_Platforms:_ visionOS 1

## Overview
This session walks through how to bring an existing multiplatform SwiftUI app—specifically the "Backyard Birds" sample app—into the visionOS Shared Space. It covers the steps to add the visionOS destination in Xcode, run the app in Simulator, and understand how SwiftUI's built-in controls automatically adapt to the new platform.

The session explores how glass backgrounds, vibrancy, and hover effects are central to the visionOS design language. Since the platform does not differentiate between light and dark appearances, semantic color styles and vibrant materials ensure legibility in all lighting conditions.

Finally, the session introduces platform-specific layout concepts like tab views, ornaments, and the `.glassBackgroundEffect()` modifier—tools for restructuring app UI to feel native and take full advantage of spatial computing.

## Key Topics

- **SwiftUI in the Shared Space** — Adding the visionOS run destination in Xcode; running in Simulator; how built-in controls adapt automatically (glass window backgrounds, sidebar darkening, hover highlights, button scale animations).
- **Polishing your app** — Using vector assets for dynamic content scaling; adopting vibrancy with semantic fill colors instead of solid colors; adding `hoverEffect()` to interactive controls; using `contentShape(_:_:)` to customize hover effect shape.
- **Brand new concepts** — Switching from `NavigationSplitView` to `TabView` for visionOS top-level navigation; using tab ornaments; adding bottom toolbar ornaments with `.toolbar` and `bottomOrnament` placement; building custom ornaments with the `ornament()` modifier; applying `.glassBackgroundEffect()` to custom ornament content.

## APIs & Frameworks

- **SwiftUI** (primary framework throughout)
- `NavigationSplitView` — existing multiplatform API; replaced by `TabView` for visionOS top-level nav
- `TabView` **[NEW on visionOS]** — renders as a floating tab bar ornament anchored to the left edge of the window
- `.tabItem(_:)` modifier — assigns label/icon to each tab
- `hoverEffect()` modifier **[NEW]** — adds system-managed hover highlight to interactive views
- `contentShape(_:_:)` modifier (with `hoverEffect` kind) **[NEW]** — customizes the shape used for the hover effect
- `.glassBackgroundEffect()` modifier **[NEW]** — applies the visionOS glass material to a view or ornament
- `ornament(attachmentAnchor:contentAlignment:content:)` modifier **[NEW]** — positions custom ornament views outside window boundaries
- `.toolbar` modifier with `bottomOrnament` placement **[NEW]** — adds a capsule-background toolbar ornament at the bottom of the window
- `Button` with `.plain` button style — renders without background, gets standard hover effect
- Semantic fill colors (`.primary`, `.secondary`, etc.) — automatically adopt vibrancy on visionOS
- Vector/PDF assets with **Preserve Vector Data** — required for dynamic content scaling on visionOS
- `NavigationStack` — still used within tab/detail views
- SF Symbols — vector by default, scale correctly on visionOS

## Code Highlights

Adding a hover effect with a custom rounded shape:
```swift
BirdView(bird: bird)
    .contentShape(.hoverEffect, RoundedRectangle(cornerRadius: 12))
    .hoverEffect()
```

Switching top-level navigation to a TabView:
```swift
TabView {
    BackyardsView()
        .tabItem { Label("Backyards", systemImage: "tree") }
    BirdsView()
        .tabItem { Label("Birds", systemImage: "bird") }
    PlantsView()
        .tabItem { Label("Plants", systemImage: "leaf") }
}
```

Adding a custom ornament:
```swift
.ornament(attachmentAnchor: .scene(.bottom), contentAlignment: .top) {
    FeederStatusView()
        .glassBackgroundEffect()
}
```

## Takeaways

- Add the visionOS destination in Xcode and run in Simulator as the first step; many SwiftUI controls adapt automatically.
- Replace solid colors with semantic fill colors to leverage vibrancy and ensure legibility against the glass background.
- Add `hoverEffect()` to any custom interactive view so users get visual confirmation before tapping.
- Use `ornament()` and `TabView` to restructure layouts for spatial computing rather than force-fitting iPad conventions.

---
_Source: WWDC23 Session 10110 page (abstract, chapter summaries, code samples, and resource links)._
