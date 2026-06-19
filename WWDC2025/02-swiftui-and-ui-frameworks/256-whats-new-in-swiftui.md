# What's new in SwiftUI
**WWDC25 · Session 256** · [Watch](https://developer.apple.com/videos/play/wwdc2025/256/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26, watchOS 26

## Overview
SwiftUI in 2025 spans five areas of improvement: visual redesign adoption (Liquid Glass), core framework foundations (performance, animations, layout), cross-platform expansion (new scene types, Controls everywhere, Widgets on visionOS and CarPlay), and two major new view capability areas — web content embedding via WebKit and rich text editing via `AttributedString`.

The session is presented through a hiking photo planning app built by two SwiftUI engineers. Every feature shown maps directly to real code changes — the session includes over a dozen compilable code snippets from the app.

## Key Topics

### Liquid Glass design adoption
Apps recompiled against the iOS 26 SDK automatically pick up the new Liquid Glass navigation, tab bar, and toolbar appearance. New additions:
- **`ToolbarSpacer(.fixed, placement:)`** — insert a fixed-width gap between toolbar item groups
- Toolbar items support `.borderedProminent` + `.tint(_:)` for colored glass
- Scroll edge effect: bar content blurs automatically as content scrolls under it
- Search is bottom-aligned on iPhone; tab-based search with `Tab(role: .search)` morphs into the search field
- **`.glassEffect()`** modifier applies the glass material to custom views
- `commands { TextEditingCommands() }` on a Scene produces the iOS 26 iPad menu bar

### Framework foundations
**Performance**: List performance on macOS improves 6x load / 16x update for large collections. Nested `ScrollView` with `LazyVStack` now correctly defers loading. New SwiftUI performance instrument in Xcode Instruments reveals per-view update causes and duration.

**Concurrency**: `@Observable`-backed views are safe with Swift structured concurrency. See "Explore concurrency in SwiftUI."

**Animations**: New **`@Animatable`** macro synthesizes `animatableData` automatically; use **`@AnimatableIgnored`** to exclude specific properties.

**3D layout (visionOS)**: `Alignment3D`, **`spatialOverlay(alignment:)`** modifier, **`.manipulable()`** modifier (picks up/moves 3D objects), **`.surfaceSnappingInfo`** environment value (table/floor/wall classification), **`VStackLayout().depthAlignment(.front/back/center)`**, **`rotation3DLayout(_:axis:)`**.

**macOS window**: **`.windowResizeAnchor(.top)`** modifier controls which edge stays fixed when window size changes.

### SwiftUI across the system
**Scene bridging**: UIKit and AppKit lifecycle apps can host SwiftUI scenes using `UIHostingSceneDelegate` / `NSHostingSceneDelegate`. Enables use of `MenuBarExtra`, `ImmersiveSpace`, `RemoteImmersiveSpace`, `AssistiveAccess` scene types from UIKit apps.

**AssistiveAccess**: New **`AssistiveAccess { ... }`** scene type for showing UI when the iPhone is in Assistive Access mode.

**RealityKit integration**: RealityKit `Entity` conforms to `Observable`. Direct gesture attachment with `GestureComponent`. SwiftUI popovers from RealityKit entities via `PresentationComponent`. New `realityViewSizingBehavior` modifier on `RealityView`.

**Controls**: Now available on macOS (menu bar + Control Center) and watchOS 26 (Control Center, Smart Stack, Action button).

**Widgets**: Level-of-detail via `@Environment(\.levelOfDetail)`. New visionOS widget families and mounting styles. Relevant widgets on watchOS (`RelevanceConfiguration`). Push-based widget updates.

### Expand SwiftUI views
**WebKit**: New **`WebView`** SwiftUI view for embedding web content. **`WebPage`** is a new `@Observable` model type providing programmatic navigation, page properties, JavaScript calls, custom URL schemes, and user agent control.

**3D charts**: **`Chart3D`** view with `SurfacePlot`, Z-axis modifiers (`chartZScale`, etc.).

**Drag and Drop (macOS)**: **`draggable(containerItemID:)`** modifier, **`dragContainer(for:selection:)`** modifier returns items lazily, **`DragConfiguration(allowMove:allowDelete:)`**, **`onDragSessionUpdated(_:)`** modifier, **`dragPreviewsFormation(.stack)`**.

**Rich text editing**: `TextEditor` now accepts a `Binding<AttributedString>` — change from `String` to `AttributedString` to get full styled text editing with system formatting controls.

## APIs & Frameworks

### SwiftUI
- **`ToolbarSpacer(.fixed, placement:)`** **[NEW]** — fixed gap between toolbar item groups
- **`.glassEffect()`** **[NEW]** — Liquid Glass material on custom views
- `Tab(role: .search)` **[NEW]** — dedicated search tab with morphing animation
- `.windowResizeAnchor(_:)` **[NEW]** — `.top`, `.leading`, etc.
- **`@Animatable`** macro **[NEW]** — auto-synthesize `animatableData`
- **`@AnimatableIgnored`** macro **[NEW]** — exclude properties from animation
- **`Alignment3D`** **[NEW]** — 3D alignment value
- **`.spatialOverlay(alignment:)`** **[NEW]** — overlay at 3D alignment
- **`.manipulable()`** **[NEW]** — pick up / move 3D objects
- **`@Environment(\.surfaceSnappingInfo)`** **[NEW]** — `SurfaceSnappingInfo` with `.classification`
- `VStackLayout().depthAlignment(_:)` **[NEW]**
- `.rotation3DLayout(_:axis:)` **[NEW]**
- **`AssistiveAccess { ... }`** **[NEW]** — scene type (iOS 26)
- `UIHostingSceneDelegate` / `NSHostingSceneDelegate` **[NEW]** — bridge UIKit/AppKit apps to SwiftUI scenes
- `realityViewSizingBehavior(_:)` **[NEW]** — on `RealityView`
- **`WebView(url:)`** **[NEW]** — WebKit-powered web view in SwiftUI
- **`WebView(_ page:)`** **[NEW]** — WebKit web view from `WebPage` model
- **`WebPage`** **[NEW]** — `@Observable` model: `load(_:)`, navigation APIs, page properties
- **`Chart3D`** **[NEW]** — 3D chart container
- `SurfacePlot(x:y:z:)` **[NEW]** — 3D surface plot mark
- `chartZScale(domain:)` **[NEW]**
- **`draggable(containerItemID:)`** **[NEW]**
- **`dragContainer(for:selection:)`** **[NEW]**
- **`DragConfiguration`** **[NEW]** — `allowMove:`, `allowDelete:`
- **`onDragSessionUpdated(_:)`** **[NEW]**
- `.dragPreviewsFormation(.stack)` **[NEW]**
- `TextEditor(text:)` accepting `Binding<AttributedString>` **[NEW]**
- `@Environment(\.levelOfDetail)` **[NEW]** — `.default` | `.simplified` for widget distance

### RealityKit (SwiftUI integration)
- `GestureComponent` **[NEW]** — attach SwiftUI gestures directly to entities
- `PresentationComponent` **[NEW]** — present SwiftUI sheets/popovers from entities
- `ViewAttachmentComponent` **[NEW]** — inline SwiftUI view in RealityKit scene

## Code Highlights

```swift
// Fixed toolbar spacer
.toolbar {
    ToolbarItemGroup(placement: .primaryAction) { UpButton(); DownButton() }
    ToolbarSpacer(.fixed, placement: .primaryAction)
    ToolbarItem(placement: .primaryAction) { SettingsButton() }
}

// Custom glass view
Button("To Top", systemImage: "chevron.up") { scrollToTop() }
    .padding().glassEffect()

// @Animatable macro
@Animatable
struct LoadingArc: Shape {
    var center: CGPoint; var radius: CGFloat
    var startAngle: Angle; var endAngle: Angle
    @AnimatableIgnored var drawPathClockwise: Bool
    func path(in rect: CGRect) -> Path { Path() }
}

// Spatial overlay
Model3D(named: "Map")
    .spatialOverlay(alignment: timeAlignment) { Sun() }

// WebView with observable model
@State private var page = WebPage()
WebView(page)
    .onAppear { page.load(URLRequest(url: sunshineMountainURL)) }

// 3D chart
Chart3D {
    SurfacePlot(x: "x", y: "y", z: "z") { x, y in sin(x) * cos(y) }
        .foregroundStyle(Gradient(colors: [.orange, .pink]))
}
.chartXScale(domain: -3...3).chartZScale(domain: -3...3)

// Rich text editing
@Binding var commentText: AttributedString
TextEditor(text: $commentText)
```

## Takeaways
- Recompile against the iOS 26 SDK to pick up Liquid Glass for free; audit toolbar, tab bar, and navigation stacks for layout issues, then use `ToolbarSpacer` and `.tint` for intentional placement.
- Enable Swift structured concurrency in SwiftUI code to get compile-time data race safety; use `@Animatable` + `@AnimatableIgnored` to simplify custom animated shapes.
- Adopt `WebView`/`WebPage` to embed web content without resorting to `WKWebView` wrappers; the observable `WebPage` model integrates cleanly with SwiftUI's update cycle.
- Switch `TextEditor`'s binding from `String` to `AttributedString` to unlock the system rich-text editing toolbar — one line of code change.

---
_Source: WWDC25 Session 256 page (abstract, chapter summaries, code samples, and resource links)._
