# Build Programmatic UI with Xcode Previews
**WWDC23 · Session 10252** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10252/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
This session is the comprehensive guide to the redesigned Xcode Previews system in Xcode 15, centered on the new `#Preview` macro. The macro replaces the previous `PreviewProvider` protocol with a simpler, file-scope syntax that works uniformly for SwiftUI views, UIKit view controllers and views, AppKit views, widgets with timeline providers, widgets with custom entry lists, and Live Activities.

The session covers three areas: the mechanics of how previews work (what gets compiled, how Xcode decides which executable to use), the writing patterns and canvas workflows for views and widgets (including timeline animation playback and pinning), and practical tips for previewing in library/framework targets, providing sample assets with Development Assets, and running previews on physical devices to access real data.

## Key Topics

### The `#Preview` Macro **[NEW]**
- `#Preview { ... }` — top-level macro that replaces `PreviewProvider`; returns one or more trailing closures of content.
- Compiled into the app target alongside all other code and resources — previews run real code, not an interpreter.
- When Swift code changes, Xcode recompiles the minimum necessary and re-runs the preview automatically.
- Optional first argument: a name string.
- Optional variadic traits after the name (for view previews): e.g., `.landscapeLeft`.

### View Previews
- **SwiftUI**: return any `View`; wrap in context views (e.g., `List`, `NavigationStack`); attach environment modifiers.
- **UIKit**: return a `UIViewController` or `UIView` — configure before returning.
- **AppKit**: return an `NSViewController` or `NSView`.
- Environment setup (data, stores, model containers) belongs in the preview, same as scene-level app setup.

### Canvas Modes
Three modes in the Xcode canvas bottom bar:
1. **Live / Interactive mode** (default): interact with the view — drag sliders, trigger animations, call async code.
2. **Selection / Static mode**: snapshot of the view; click an element to jump to its source; double-click text to edit inline.
3. **Variants mode**: shows all values for a device setting simultaneously (color scheme, dynamic type sizes, orientations); click a variant to inspect; page through with arrow keys.

Canvas also provides a **Device Settings popover** for quickly toggling dark mode, dynamic type size, etc., without editing code.

### Widget Previews **[NEW]**
Two patterns for widgets:

1. **With a timeline provider**: pass the widget, timeline provider, and widget family. Xcode snapshots every entry in the timeline and shows them in the canvas — no waiting for real time to elapse. Click through or use arrow keys; Xcode animates transitions between entries so you can spot animation issues.

2. **With a custom entry list** (`timeline:` trailing closure): hand-craft the exact set of entries to replicate a specific scenario (e.g., a bad animation between two particular states). Use the play/loop button in the canvas to repeatedly replay a transition while fixing it.

- **Pinning**: click the pin button (upper left of canvas) to keep a preview active while navigating to another file — essential when the code you need to fix is in a different source file than the preview.

### Live Activity Previews **[NEW]**
- Same API shape as widget previews but pass `ActivityAttributes` and an array of `ContentState` values to preview all states and transitions.

### Previews in Library Targets
- Previews need an executable (app or widget) to host them. Xcode traverses from the source file up through target dependencies and intersects with the active scheme to pick one.
- If no app exists, Xcode creates `XCPreviewAgent` automatically to host the library preview.
- Strategy: modularize into library targets + create small scheme-specific preview apps to get faster build times and correct capabilities (Info.plist keys, entitlements).
- Add required Info.plist keys to a minimal preview-only app target; embed the library via Target Dependencies and Copy Files build phase.

### Development Assets
- Development Assets: a build settings path (`DEVELOPMENT_ASSET_PATHS`) listing folders that are excluded from App Store submissions.
- Add asset catalogs, sample images, mock data, and other preview resources to a Development Assets folder — they're available in previews but stripped on distribution.
- New Xcode projects and app targets get a `Preview Content` folder configured automatically.

### Previewing on Physical Devices
- The **Preview Device Picker** (canvas bottom bar) lets you choose:
  - **Automatic** (tracks the active run destination's device family).
  - Specific simulator models (from the Devices window).
  - Devices by feature (Touch ID, Face ID, etc.).
  - Any connected physical device.
- When a physical device is selected, Xcode builds for that device only, bypassing the simulator. All canvas modes and device settings still work.
- Use physical devices to access real data (Photos library, files, sensors, camera) without needing to set up mock data.

## APIs & Frameworks

### Xcode 15 Previews **[NEW]**
- `#Preview` macro — unified preview definition for SwiftUI/UIKit/AppKit/Widgets/LiveActivity **[NEW]**
- `#Preview("Name") { view }` — named preview **[NEW]**
- `#Preview("Name", traits: .landscapeLeft) { view }` — preview with orientation trait **[NEW]**
- `#Preview(as: .systemSmall) { Widget() } timelineProvider: { Provider() }` — widget preview with timeline provider **[NEW]**
- `#Preview(as: .systemSmall) { Widget() } timeline: { entry1; entry2 }` — widget preview with custom entries **[NEW]**
- Live Activity preview: `#Preview(as: .dynamicIsland(.expanded), using: MyAttributes()) { Widget() } contentStates: { state1; state2 }` **[NEW]**
- Canvas modes: Live, Selection (Static), Variants **[enhanced]**
- Canvas Device Settings popover — color scheme, dynamic type, orientation **[enhanced]**
- Pin button — keep preview active while navigating to another file **[NEW]**
- Preview Device Picker — select simulator, feature-based device, or connected physical device **[enhanced]**
- `DEVELOPMENT_ASSET_PATHS` build setting — exclude development resources from App Store submissions (existing; auto-configured for new projects)
- `XCPreviewAgent` — auto-generated executable for library-only previews

### SwiftUI
- `.environment(_:)` — inject observable objects / environment values into preview
- `.modelContainer(_:)` — inject SwiftData container into preview (existing)
- `PreviewProvider` — replaced by `#Preview` macro (deprecated path)

### UIKit / AppKit
- `UIViewController` / `UIView` — return directly from `#Preview` closure; configure before returning
- `NSViewController` / `NSView` — same pattern for AppKit

## Code Highlights

Basic SwiftUI preview:
```swift
#Preview {
    MyView()
}
```

SwiftUI view in context with environment data:
```swift
#Preview {
    List {
        CollageView(layout: .twoByTwoGrid)
    }
    .environment(CollageLayoutStore.sample)
}
```

Named preview with landscape trait:
```swift
#Preview("2x2 Grid", traits: .landscapeLeft) {
    List { CollageView(layout: .twoByTwoGrid) }
        .environment(CollageLayoutStore.sample)
}
```

UIKit view controller preview:
```swift
#Preview("All Filters", traits: .landscapeLeft) {
    let vc = FilterRenderingViewController()
    vc.imageData = UIImage(named: "sample-001")?.cgImage
    vc.filter = Filter(bloomAmount: 1.0, vignetteAmount: 1.0, saturationAmount: 0.5)
    return vc
}
```

Widget preview with timeline provider:
```swift
#Preview(as: .systemSmall) {
    FrameWidget()
} timelineProvider: {
    RandomCollageProvider()
}
```

Widget preview with specific entries for debugging a transition:
```swift
#Preview(as: .systemSmall) {
    FrameWidget()
} timeline: {
    CollageEntry(date: .now, layout: .twoByTwoGrid)
    CollageEntry(date: .now + 3600, layout: .threeColumns)
}
```

## Takeaways
- Replace all `PreviewProvider` conformances with `#Preview` — it's shorter, works for SwiftUI/UIKit/AppKit/widgets uniformly, and supports traits directly in the macro call.
- Use **Variants mode** to see all color scheme or dynamic type variations simultaneously; this catches layout issues at multiple type sizes without writing multiple previews.
- For widget timeline debugging, switch from `timelineProvider:` to `timeline:` to craft the exact two entries that reproduce an animation issue, then use pin + loop to fix it without losing context.
- Connect a physical device in the Preview Device Picker to access real Photos/files/sensor data without needing to set up mock data in your project.

---
_Source: WWDC23 Session 10252 page (abstract, chapter summaries, code samples, and resource links)._
