# Use SwiftUI with AppKit and UIKit
**WWDC26 · Session 272** · [Watch](https://developer.apple.com/videos/play/wwdc2026/272/)

_Platforms:_ macOS (primary), iOS/iPadOS (UIKit side)

## Overview
This session demonstrates five concrete techniques for incrementally adopting SwiftUI inside existing AppKit (or UIKit) apps — using a lighting-control macOS app ("LightMix") as the running example. The central message is that there is no requirement to migrate an entire app; you can adopt SwiftUI one view, one gesture, one menu, or one scene at a time.

The five techniques are: (1) replacing manual `needsDisplay` invalidation with automatic Observation tracking, (2) embedding a SwiftUI `Canvas` view in an AppKit hierarchy via `NSHostingView`, (3) reusing an existing `NSGestureRecognizer` subclass in SwiftUI via `NSGestureRecognizerRepresentable`, (4) building an `NSMenu` item from a SwiftUI `View` using `NSHostingMenu`, and (5) adding complete SwiftUI scenes (a `MenuBarExtra` and `Settings` scene) to an `NSApplicationDelegate`-based app via `NSHostingSceneRepresentation`.

## Key Topics

### Observation in AppKit (2:33)
By marking a model class `@Observable @MainActor`, AppKit views (and UIKit views) that read its properties inside `draw(_:)`, `updateConstraints()`, `layout()`, or `updateLayer()` automatically re-execute those methods when the model changes — no `addObserver` / `removeObserver` plumbing required. This is back-deployable to macOS 15 / iOS 18 by opting in via `NSObservationTrackingEnabled` / `UIObservationTrackingEnabled`, and is on by default starting with the 2027 releases.

### Hosting SwiftUI in AppKit (5:41)
The color-picker control is rebuilt as a SwiftUI `Canvas` (analogous to `drawRect:`, but with `withCGContext` for reusing CoreGraphics code) and wrapped in `NSHostingView(rootView:)`. This exposes the full power of SwiftUI animations, gesture modifiers, and previews for a view that would otherwise require complex AppKit drawing code.

### AppKit Gestures in SwiftUI (7:48)
**[NEW]** `NSGestureRecognizerRepresentable` is a new protocol analogous to `UIGestureRecognizerRepresentable`. Implement `makeNSGestureRecognizer(context:)` to return any `NSGestureRecognizer` subclass, and `handleNSGestureRecognizerAction(_:context:)` to respond to state changes. Attach it with the standard `.gesture(_:)` modifier — demonstrated with a Force Click recognizer that resets color picker saturation and brightness.

### SwiftUI in the Main Menu (9:16)
**[NEW]** `NSHostingMenu` is an `NSMenu` subclass that takes a SwiftUI `View` as its root. Any SwiftUI view returning `Button`, `Divider`, `Picker`, or `Menu` items becomes a fully functional macOS menu, including support for `.keyboardShortcut` and `.pickerStyle(.palette)`. Wrap it in an `NSMenuItem` and add it to the `NSApplication.mainMenu` like any other menu.

### SwiftUI Scenes in AppKit (11:30)
**[NEW]** `NSHostingSceneRepresentation` accepts a `@SceneBuilder` closure of SwiftUI `Scene` values and registers them with the running `NSApplication`. This lets an `NSApplicationDelegate` app add a `MenuBarExtra`, `Settings`, or other SwiftUI scene without converting the entire app to the SwiftUI app lifecycle. The returned representation's `.environment` exposes `openSettings()` and similar environment actions so existing AppKit code can trigger them.

## APIs & Frameworks

**SwiftUI / AppKit interop**
- `NSHostingView<Content>` — embed a SwiftUI view in AppKit (existing)
- **[NEW]** `NSHostingMenu<Content: View>` — `NSMenu` subclass backed by a SwiftUI View
- **[NEW]** `NSHostingSceneRepresentation` — adds SwiftUI scenes to an NSApp-lifecycle app
- **[NEW]** `NSGestureRecognizerRepresentable` protocol
  - `makeNSGestureRecognizer(context:) -> GestureRecognizer`
  - `handleNSGestureRecognizerAction(_:context:)`
- `NSApplication.shared.addSceneRepresentation(_:)`
- `NSHostingSceneRepresentation.environment` — access SwiftUI environment actions (e.g., `openSettings()`)

**Observation framework**
- `@Observable` macro on `@MainActor` classes
- `NSObservationTrackingEnabled` (macOS 15 back-deploy opt-in)
- `UIObservationTrackingEnabled` (iOS 18 back-deploy opt-in)
- Automatic tracking in `draw(_:)`, `updateConstraints()`, `updateLayer()`, `layout()`, and UIViewController equivalents

**SwiftUI**
- `Canvas { context, size in }` — immediate-mode drawing (analogous to `drawRect`)
- `GraphicsContext.withCGContext { cgContext in }` — CoreGraphics escape hatch
- `@Animatable` / `@AnimatableIgnored` — custom animation conformance
- `DragGesture(minimumDistance:coordinateSpace:)` with `.onChanged` / `.onEnded`
- `.gesture(_:)` modifier — attaches NSGestureRecognizerRepresentable
- `MenuBarExtra` scene
- `Settings` scene
- `Button`, `Divider`, `Picker`, `Menu` (as SwiftUI menu items)
- `.keyboardShortcut(_:modifiers:)` modifier
- `.pickerStyle(.palette)`
- `Bindable` — wraps @Observable for two-way bindings
- `withAnimation { }` closure

**AppKit**
- `NSGestureRecognizer` subclassing
- `NSStatusItem.expandedInterfaceDelegate` (related session topic)
- `NSWindow.preventsApplicationTerminationWhenModal`

## Code Highlights

Automatic AppKit view invalidation via Observation:
```swift
@Observable @MainActor
final class ColorModel {
    var hue: Double = 0.6
    var saturation: Double = 1.0
    var brightness: Double = 1.0
}
// AppKit draw(_:) reads model properties → auto-redraws on change
```

Adding an NSGestureRecognizer to a SwiftUI view:
```swift
struct ForceClickReset: NSGestureRecognizerRepresentable {
    var model: ColorModel
    func makeNSGestureRecognizer(context: Context) -> ForceClickGestureRecognizer {
        ForceClickGestureRecognizer()
    }
    func handleNSGestureRecognizerAction(_ recognizer: ForceClickGestureRecognizer, context: Context) {
        withAnimation { model.saturation = 1; model.brightness = 1 }
    }
}
```

Adding SwiftUI scenes to an NSApplicationDelegate app:
```swift
let scenes = NSHostingSceneRepresentation {
    LightMenuBarExtra(appModel: model)
    LightSettings(appModel: model)
}
NSApplication.shared.addSceneRepresentation(scenes)
```

## Takeaways
- Adopt `@Observable` on existing model classes first — it immediately eliminates manual invalidation in AppKit/UIKit views and is back-deployable.
- When a feature needs SwiftUI-level interactivity or previews, wrap it in a SwiftUI `Canvas` / `View` and host it with `NSHostingView` rather than rewriting the entire screen.
- Use `NSGestureRecognizerRepresentable` to reuse existing platform-specific recognizers (Force Click, Magic Mouse gestures, Sidecar touch) inside SwiftUI views.
- `NSHostingSceneRepresentation` is the cleanest path to adding a `MenuBarExtra` or `Settings` window to an existing AppKit app without a full lifecycle migration.

---
_Source: WWDC26 Session 272 page (abstract, chapter summaries, code samples, and resource links)._
