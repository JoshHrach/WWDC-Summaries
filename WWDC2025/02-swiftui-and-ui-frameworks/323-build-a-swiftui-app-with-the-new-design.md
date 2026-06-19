# Build a SwiftUI App with the New Design
**WWDC25 · Session 323** · [Watch](https://developer.apple.com/videos/play/wwdc2025/323/)

_Platforms:_ iOS 26, macOS Tahoe 26, iPadOS 26

## Overview
This session is a practical walkthrough of adopting iOS 26's new design system in a SwiftUI app. The presenter updates a real app end-to-end, demonstrating how Liquid Glass material, new tab bar behaviors, new toolbar and navigation APIs, and updated control styles come together to create the fluid, glass-forward aesthetic of iOS 26 and macOS Tahoe.

The session shows that many updates are automatic on recompilation, but truly great results require deliberate use of new modifiers. It covers: Liquid Glass applied to tabs, toolbars, and custom surfaces; tab bar visibility and bottom accessory APIs; toolbar spacing and badge improvements; scroll edge effects; and the `glassEffect` API for custom components.

## Key Topics

### Automatic Design Adoption
Recompiling against the iOS 26 SDK automatically brings in the new glass-style tab bar, updated navigation bar, and new control renders. Developers get a baseline upgrade for free.

### Tab Bar and Navigation
The tab bar now floats with a Liquid Glass background. `.tabBarMinimizeBehavior(.onScrollDown)` hides the tab bar when the user scrolls down content, reclaiming screen space. `.tabViewBottomAccessory` attaches a persistent view below the tab bar — useful for Now Playing bars or persistent actions.

### Toolbar Updates
`ToolbarSpacer` provides fixed and flexible spacing between toolbar items, enabling precise layout control. `.sharedBackgroundVisibility` controls whether items share a unified glass background. `.badge` can be attached to any toolbar item. `.scrollEdgeEffectStyle` controls the fade behavior at scroll edges. `.searchToolbarBehavior` customizes search field visibility and placement. Sheet presentations from toolbar items morph seamlessly using navigation zoom transition.

### Background Extension
`.backgroundExtensionEffect` extends a view's background material to bleed under the navigation or tab bar, creating a seamless full-screen glass look — particularly effective with hero images and media content.

### Glass Effects API
`glassEffect(_:)` applies Liquid Glass material to any view. `GlassEffectContainer` groups multiple glass elements, enabling coordinated glass rendering. `.glassEffectID(_:in:)` assigns an identity within a container, enabling fluid morphing transitions between states (e.g., a search bar expanding from a button). `.interactive` makes the glass surface react to pointer hover. Button styles `.glass` and `.glass prominent` apply glass styling to buttons.

### Shapes and Controls
`.concentric` rectangle shape and `containerConcentric` corner configuration keep nested rounded-rectangle elements optically aligned. Sliders gain tick marks (automatic with `step:`, manual with the `ticks:` closure) and a `neutralValue` parameter. Extra-large button size is now supported via `.controlSize(.extraLarge)`.

## APIs & Frameworks

**SwiftUI (iOS 26, macOS Tahoe 26)**
- **[NEW]** `.tabBarMinimizeBehavior(_:)` modifier — `.onScrollDown` to auto-hide tab bar
- **[NEW]** `.tabViewBottomAccessory` modifier — persistent view below the tab bar
- **[NEW]** `ToolbarSpacer` — fixed and flexible spacing between toolbar items
- **[NEW]** `.sharedBackgroundVisibility` modifier — shared glass background for toolbar clusters
- **[NEW]** `.badge` modifier on toolbar items
- **[NEW]** `.scrollEdgeEffectStyle` modifier — scroll edge fade behavior
- **[NEW]** `.searchToolbarBehavior` modifier — customize search toolbar integration
- **[NEW]** `.backgroundExtensionEffect` modifier — extend background under system chrome
- **[NEW]** `glassEffect(_:)` modifier — apply Liquid Glass material to any view
- **[NEW]** `GlassEffectContainer` view — coordinate glass rendering for grouped elements
- **[NEW]** `.glassEffectID(_:in:)` modifier — identity for morphing glass transitions
- **[NEW]** `.interactive` modifier — enable hover interaction on glass surface
- **[NEW]** `.glass` button style
- **[NEW]** `.glass prominent` button style
- **[NEW]** `.concentric` rectangle shape
- **[NEW]** `containerConcentric` corner configuration
- Slider `ticks:` closure — manual tick mark specification
- Slider `neutralValue:` parameter — visual neutral/zero marker
- `.controlSize(.extraLarge)` — extra-large button size

## Code Highlights
Apply tab bar minimize behavior and bottom accessory:
```swift
TabView {
    // tabs...
}
.tabBarMinimizeBehavior(.onScrollDown)
.tabViewBottomAccessory {
    NowPlayingBar()
}
```

Apply Liquid Glass to a custom view and enable morphing transition:
```swift
GlassEffectContainer {
    Button("Search") { showSearch = true }
        .glassEffect()
        .glassEffectID("searchControl", in: namespace)
    
    if showSearch {
        SearchBar()
            .glassEffect()
            .glassEffectID("searchControl", in: namespace)
    }
}
```

Extend background under navigation bar:
```swift
HeroImageView()
    .backgroundExtensionEffect()
```

## Takeaways
- Recompile against iOS 26 SDK for automatic design system adoption; then apply specific new modifiers for a polished result.
- Use `GlassEffectContainer` + `.glassEffectID(_:in:)` for fluid morphing transitions between glass states (e.g., collapsed toolbar → expanded search).
- Apply `.tabBarMinimizeBehavior(.onScrollDown)` to maximize content space in scroll-heavy views; use `.tabViewBottomAccessory` for persistent mini-players or status bars.
- Use `.backgroundExtensionEffect` on hero images and media views to seamlessly extend content behind the glass navigation bar.

---
_Source: WWDC25 Session 323 page (abstract, chapter summaries, code samples, and resource links)._
