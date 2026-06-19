# Tailor macOS Windows with SwiftUI
**WWDC24 · Session 10148** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10148/)

_Platforms:_ macOS Sequoia 15

## Overview
This session walks through new SwiftUI APIs for customizing macOS windows across three dimensions: toolbar styling, window behavior refinement, and window placement control. Using the Destination Video sample app as context, the session demonstrates how a small number of modifiers can dramatically improve the fit-and-finish of focused-purpose windows—such as a media browser, an About window, and a video player—without sacrificing macOS idioms.

The primary goal throughout is to match window chrome to content: hiding titles and toolbar backgrounds where full-bleed imagery is the focus, applying materials for elegant translucency, disabling inappropriate controls like Minimize for static windows, suppressing state restoration for windows that should always start fresh, and computing content-aware initial and zoom placement for media playback.

## Key Topics

### Anatomy of a macOS Window
A typical macOS window consists of a toolbar region (window controls, title, toolbar items), the main content area, and the window background. Each element can be independently styled or suppressed using SwiftUI scene and view modifiers.

### Styling Window Toolbars
`.toolbar(removing: .title)` hides the window title from the toolbar while keeping it accessible via VoiceOver and the main menu. `.toolbarBackgroundVisibility(.hidden, for: .windowToolbar)` removes the toolbar background, allowing content to extend to the top edge of the window. These two modifiers together enable full-bleed content designs while preserving accessibility and system integration.

### Refining Window Behaviors
`.containerBackground(.thickMaterial, for: .window)` replaces the window's opaque background with a system material (frosted glass), tinting with desktop colors for a more elegant look. `.windowMinimizeBehavior(.disabled)` disables the Minimize (yellow) window control for windows where minimization makes no sense (e.g., an About window). `.restorationBehavior(.disabled)` prevents the system from saving and restoring the window's position/size across app launches—appropriate for utility windows like About.

### Adjusting Window Placement
`.defaultWindowPlacement { content, context in ... }` controls the initial size and position of a newly opened window. The closure receives a content proxy (`.sizeThatFits(_:)`) and a context with `defaultDisplay.visibleRect` (accounting for menu bar and Dock). Return a `WindowPlacement(size:)` (position defaults to centered) or a `WindowPlacement(size:anchor:)` for explicit positioning.

`.windowIdealPlacement { content, context in ... }` controls the target size when the user invokes Zoom from the Window menu. Use it to grow the window to fill the display while preserving an ideal aspect ratio.

### Additional APIs
`.windowStyle(.plain)` creates a borderless window for custom chrome experiences. `.defaultLaunchBehavior(.presented)` ensures a particular scene (e.g., a welcome window) is shown on first launch.

## APIs & Frameworks

- `.toolbar(removing:)` **[NEW]** — removes a specific toolbar element (e.g., `.title`) from the window toolbar
- `.toolbarBackgroundVisibility(_:for:)` — controls toolbar background visibility; pass `.hidden` and `.windowToolbar` to remove toolbar background
- `.containerBackground(_:for:)` **[NEW]** — sets window background to a material or color; pass `.thickMaterial` and `.window`
- `.windowMinimizeBehavior(_:)` **[NEW]** — controls minimize button behavior; `.disabled` grays out the yellow control
- `.restorationBehavior(_:)` **[NEW]** — controls state restoration; `.disabled` prevents size/position from being saved across launches (scene modifier)
- `.defaultWindowPlacement(_:)` **[NEW]** — closure-based initial window size/position; receives `WindowPlacementContext` and `WindowLayoutRoot`
- `.windowIdealPlacement(_:)` **[NEW]** — closure-based zoom target size/position; same parameters as `defaultWindowPlacement`
- `WindowPlacement` **[NEW]** — value type specifying a window's size and optional position
- `WindowPlacementContext` **[NEW]** — provides `defaultDisplay: DisplayProxy` in placement closures
- `DisplayProxy.visibleRect` **[NEW]** — the usable area of the display excluding menu bar and Dock
- `WindowLayoutRoot.sizeThatFits(_:)` **[NEW]** — queries the ideal size of the window content
- `.windowStyle(.plain)` **[NEW]** — creates a borderless window with no standard chrome
- `.defaultLaunchBehavior(.presented)` **[NEW]** — presents a scene by default on app launch (e.g., for welcome windows)
- `WindowGroup` — multi-instance window scene (unchanged)
- `Window` — single-instance window scene (unchanged)
- `Scene` protocol — parent protocol for all SwiftUI scenes
- `SwiftUI` framework

## Code Highlights

Hide toolbar title and background for full-bleed content:
```swift
WindowGroup {
    ContentView()
        .toolbar(removing: .title)
        .toolbarBackgroundVisibility(.hidden, for: .windowToolbar)
}
```

About window with material background, no minimize, no restoration:
```swift
Window("About", id: "about") {
    AboutView()
        .toolbar(removing: .title)
        .toolbarBackgroundVisibility(.hidden, for: .windowToolbar)
        .containerBackground(.thickMaterial, for: .window)
}
.windowMinimizeBehavior(.disabled)
.restorationBehavior(.disabled)
```

Content-aware initial placement for a video player window:
```swift
WindowGroup(for: Video.self) { $video in
    VideoPlayerView(video: video)
}
.defaultWindowPlacement { content, context in
    var size = content.sizeThatFits(.unspecified)
    let displayBounds = context.defaultDisplay.visibleRect
    // Scale down if video is larger than visible display area
    return WindowPlacement(size: size)
}
```

Zoom-aware placement preserving aspect ratio:
```swift
.windowIdealPlacement { content, context in
    var size = content.sizeThatFits(.unspecified)
    let displayBounds = context.defaultDisplay.visibleRect
    return WindowPlacement(size: zoomToFit(size, in: displayBounds))
}
```

Borderless window and default launch presentation:
```swift
.windowStyle(.plain)
.defaultLaunchBehavior(.presented)
```

## Takeaways

- `.toolbar(removing: .title)` and `.toolbarBackgroundVisibility(.hidden, for: .windowToolbar)` enable full-bleed content layouts while preserving accessibility and menu-bar window title display.
- `.containerBackground(.thickMaterial, for: .window)` applies a system material (frosted glass) to the window background, tinting with desktop color for an elegant translucent look.
- `.windowMinimizeBehavior(.disabled)` and `.restorationBehavior(.disabled)` are essential for windows with static content (like About windows) that should never be minimized or restored across launches.
- `.defaultWindowPlacement` and `.windowIdealPlacement` give precise, content-aware control over where and how large a window opens and zooms, critical for media playback across varied display configurations.

---
_Source: WWDC24 Session 10148 page (abstract, chapter summaries, code samples, and resource links)._
