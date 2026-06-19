# Work with Windows in SwiftUI
**WWDC24 · Session 10149** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10149/)

_Platforms:_ visionOS 2, macOS Sequoia 15, iPadOS 18

## Overview
This session focuses on creating great single and multi-window apps using SwiftUI's window management APIs. Using the BOT-anist sample app as its running example across visionOS and macOS, the session covers three major areas: fundamentals of defining and opening windows (including the new `pushWindow` action), controlling initial window placement with `defaultWindowPlacement`, and sizing windows using `defaultSize`, `windowResizability`, and content-driven frame constraints. The concepts apply to all multi-window platforms but the demos center on visionOS.

## Key Topics

### Fundamentals — Defining and Opening Windows
Each window in a SwiftUI app is backed by a `WindowGroup` (or `Window` for singletons) declared in the app body and identified by a string ID. Three environment actions are available anywhere in the SwiftUI hierarchy:
- `openWindow(id:)` — opens a new instance of the named `WindowGroup`
- `dismissWindow(id:)` — closes a window by ID
- `pushWindow(id:)` **[NEW]** — opens the new window and hides the originating window; closing the new window automatically restores the originating one

`pushWindow` is ideal for flows where the new content replaces the current view (e.g., opening a movie portal from an editor window — the editor disappears, and closing the movie returns to the editor). No additional restoration logic is required.

### Enhancing Windows
In visionOS, windows can be enhanced with platform-specific features:
- `ToolbarItem` and `ToolbarTitleMenu` add controls and document-related actions in the window bar
- `.persistentSystemOverlays(.hidden)` hides the window bar and close button — useful for immersive content like video playback
- `.windowStyle(.volumetric)` makes a `WindowGroup` a 3D volume in visionOS

### Placement — `defaultWindowPlacement`
The `defaultWindowPlacement(_:)` modifier on a `WindowGroup` accepts a closure receiving `content` and `context` and returning a `WindowPlacement`. This sets the initial position and optionally size of the window on first open. Platform-specific positions include:
- `.utilityPanel` (visionOS) — places the window close to the user, within direct touch range
- Calculated `CGPoint` from `context.defaultDisplay.visibleRect` and `content.sizeThatFits(.unspecified)` (macOS) — manual positioning relative to display bounds
- Relative positions like `.leading`, `.trailing` (relative to other windows)
- The user can freely reposition the window after the initial placement

### Sizing
Multiple mechanisms control window size:
- `defaultSize(width:height:)` — sets initial size; ignored if a window placement provides a size, or when scenes are restored
- For pushed windows, `defaultSize` matches the originating window's size
- `.windowResizability(.contentSize)` — constrains the window's resizable range to the min/max of its content view; combine with `.frame(minWidth:maxWidth:minHeight:maxHeight:)` on the content to set exact limits
- When content views have fixed (not range) sizes, the window becomes non-resizable by the user
- `.windowResizability(.automatic)` is the default (system-determined resizability)

## APIs & Frameworks

**SwiftUI — Window Environment Actions**
- `@Environment(\.openWindow) var openWindow` — open a named window **[existing]**
- `@Environment(\.dismissWindow) var dismissWindow` — close a window by ID **[existing]**
- `@Environment(\.pushWindow) var pushWindow` **[NEW]** — open window, hide originating; restore on dismiss

**SwiftUI — Scene Modifiers**
- `defaultWindowPlacement(_:)` **[NEW]** — closure returning `WindowPlacement` for initial position/size
- `WindowPlacement(_ position: WindowPlacement.Position)` **[NEW]** — position-only placement
- `WindowPlacement(_ position: CGPoint, size: CGSize)` **[NEW]** — explicit position and size
- `WindowPlacement.Position.utilityPanel` **[NEW]** — visionOS near-user panel placement
- `context.defaultDisplay.visibleRect` **[NEW]** — safe display area for macOS placement calculation
- `content.sizeThatFits(_:)` **[NEW]** — ask window content for its preferred size
- `.windowResizability(.contentSize)` **[NEW]** — limit resize to content's min/max frame
- `.windowResizability(.automatic)` — existing default behavior
- `defaultSize(width:height:)` — existing, sets initial size
- `.windowStyle(.volumetric)` — existing visionOS volume style
- `.windowStyle(.plain)` — borderless window (mentioned in related session 10148)
- `.persistentSystemOverlays(.hidden)` **[NEW]** — hide window bar and close button
- `ToolbarTitleMenu` — existing; useful for document-related window actions

**SwiftUI — App Structure**
- `WindowGroup(id:)` — multi-instance window scene identified by string ID
- `Window` — singleton window scene

## Code Highlights

App structure with multiple `WindowGroup` scenes:
```swift
@main
struct BOTanistApp: App {
    var body: some Scene {
        WindowGroup(id: "editor") {
            EditorContentView()
        }
        WindowGroup(id: "game") {
            GameContentView()
        }
        .windowStyle(.volumetric)
        WindowGroup(id: "movie") {
            MovieContentView()
        }
        WindowGroup(id: "controller") {
            ControllerContentView()
        }
    }
}
```

Opening a window with `openWindow`:
```swift
struct EditorContentView: View {
    @Environment(\.openWindow) private var openWindow

    var body: some View {
        Button("Open Movie", systemImage: "tv") {
            openWindow(id: "movie")
        }
    }
}
```

Pushing a window (hides originating window):
```swift
struct EditorContentView: View {
    @Environment(\.pushWindow) private var pushWindow

    var body: some View {
        Button("Open Movie", systemImage: "tv") {
            pushWindow(id: "movie")
        }
    }
}
```

Hiding system window controls for immersive content:
```swift
WindowGroup(id: "movie") {
    MovieContentView()
}
.persistentSystemOverlays(.hidden)
```

`defaultWindowPlacement` — visionOS utility panel + macOS calculated position:
```swift
WindowGroup(id: "controller") {
    ControllerContentView()
}
.defaultWindowPlacement { content, context in
    #if os(visionOS)
    return WindowPlacement(.utilityPanel)
    #elseif os(macOS)
    let displayBounds = context.defaultDisplay.visibleRect
    let size = content.sizeThatFits(.unspecified)
    let position = CGPoint(
        x: displayBounds.midX - (size.width / 2),
        y: displayBounds.maxY - size.height - 20
    )
    return WindowPlacement(position, size: size)
    #endif
}
```

`windowResizability` with content frame limits:
```swift
WindowGroup(id: "movie") {
    MovieContentView()
        .frame(
            minWidth: 680, maxWidth: 2720,
            minHeight: 680, maxHeight: 1020
        )
}
.windowResizability(.contentSize)
```

Non-resizable controller window (fixed content size):
```swift
WindowGroup(id: "controller") {
    ControllerContentView()
}
.windowResizability(.contentSize)
```

## Takeaways
- Use `pushWindow` for forward-navigation flows where the user dives into new content and the originating window is no longer needed — the system automatically restores it when the pushed window closes.
- Use `defaultWindowPlacement` to give windows a sensible initial position: `.utilityPanel` on visionOS places the window within touch reach; on macOS, calculate from `context.defaultDisplay.visibleRect` and `content.sizeThatFits` for precise centering.
- Combine `.windowResizability(.contentSize)` with `.frame(minWidth:maxWidth:minHeight:maxHeight:)` on the content view to give users a well-bounded resize experience — omit min/max to make the window non-resizable.
- Use `.persistentSystemOverlays(.hidden)` on immersive content windows (video, portals) to keep the focus on content rather than window chrome.

---
_Source: WWDC24 Session 10149 page (abstract, chapter summaries, code samples, and resource links)._
