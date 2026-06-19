# Go Beyond the Window with SwiftUI
**WWDC23 · Session 10111** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10111/)

_Platforms:_ visionOS 1

## Overview
This session is a deep dive into `ImmersiveSpace`, a new SwiftUI scene type for visionOS that enables apps to place content anywhere in the user's surroundings without clipping bounds. Unlike windows and volumes — which constrain content within defined boundaries — an `ImmersiveSpace` gives apps the entire physical space around the user, supporting mixed, progressive, and fully immersive experiences within a single scene.

The session covers the complete lifecycle of an immersive space: defining the scene, opening and dismissing it via environment actions, handling scene phase transitions, converting coordinates between the space and other windows, switching immersion styles dynamically based on gestures, and adding advanced customizations such as surrounding effects dimming, upper limb visibility control, and ARKit-powered virtual hand rendering using `HandTrackingProvider`.

Code is demonstrated using the "World" sample app, progressively building from a basic `ImmersiveSpace` declaration to a fully customized space featuring virtual space gloves.

## Key Topics

### ImmersiveSpace Scene Type
- Defined like other SwiftUI scenes; accepts any SwiftUI view in its body.
- Opening a space puts the app into a Full Space — all other apps are hidden; only the active app is visible.
- Only one space can be open at a time; dismiss the current space before opening another.
- Space origin is located near the user's feet when the space is first opened.
- Space coordinate system uses the same orientation as SwiftUI (y-axis down, z-axis toward viewer); RealityKit within a space uses inverted y-axis (y-axis up).

### Opening and Dismissing Spaces
- `openImmersiveSpace` environment action — async, takes an identifier string; returns a result indicating success or failure.
- `dismissImmersiveSpace` environment action — async, no argument (only one space can be open).
- Async calls allow reacting to animation completion.

### Scene Phase Management
- `ImmersiveSpace` supports `.active`, `.inactive`, and `.background` scene phases.
- Inactive phase triggered by leaving system-defined boundaries or system alerts.
- Use `.onChange(of: scenePhase)` to respond — e.g., scale down content when inactive, restore when active.

### Coordinate Conversions
- `GeometryReader3D` provides a 3D geometry proxy.
- `proxy.transform(in: .immersiveSpace)` converts a view's position into the immersive space coordinate system.
- Enables placing RealityKit entities relative to SwiftUI window positions.

### Immersion Styles
- Three styles: `.mixed` (default, passthrough overlay), `.progressive` (portal view with Digital Crown control), `.full` (no passthrough).
- `immersionStyle(selection:in:)` scene modifier declares the supported styles and tracks the current selection via a binding.
- Styles can be changed dynamically at runtime (e.g., based on `MagnifyGesture` magnitude).

### Launching Directly into a Space
- Configure the scene manifest with `ImmersiveSpace` application role and desired immersion style.
- App launches directly into the space without showing a window first.

### Advanced Customizations
- `preferredSurroundingsEffect(.systemDark)` modifier — dims passthrough to focus on content (e.g., when switching to progressive style).
- `upperLimbVisibility(.hidden)` modifier — hides the user's real hands in full immersion so virtual hands can be rendered instead.
- Virtual hands implemented via `HandTrackingProvider` (ARKit), anchoring loaded USDZ glove entities to joint transforms from `AnchorUpdate`.

## APIs & Frameworks
- `ImmersiveSpace` **[NEW]** — SwiftUI scene type for boundless immersive experiences on visionOS
- `openImmersiveSpace` environment action **[NEW]** — async action to open a space by identifier
- `dismissImmersiveSpace` environment action **[NEW]** — async action to dismiss the open space
- `ImmersionStyle` **[NEW]** — enum with cases `.mixed`, `.progressive`, `.full`
- `immersionStyle(selection:in:)` scene modifier **[NEW]** — declares and binds supported immersion styles
- `preferredSurroundingsEffect(_:)` modifier **[NEW]** — configures passthrough dimming effect (`.systemDark`)
- `upperLimbVisibility(_:)` modifier **[NEW]** — shows or hides real hands in immersive spaces
- `GeometryReader3D` **[NEW]** — 3D geometry proxy container for coordinate conversions
- `GeometryProxy3D.transform(in:)` **[NEW]** — converts position to a named coordinate space
- `.immersiveSpace` coordinate space **[NEW]** — named coordinate space representing the open immersive space
- `RealityView` **[NEW]** — SwiftUI view for hosting RealityKit entities; ideal for use inside `ImmersiveSpace`
- `Model3D` **[NEW]** — SwiftUI view for async 3D model loading with phase handling
- `MagnifyGesture` — SwiftUI gesture for pinch-to-zoom; used here to trigger style transitions
- `scenePhase` environment value — tracks `.active`, `.inactive`, `.background` states
- `ARKitSession` **[NEW]** — manages ARKit provider sessions on visionOS
- `HandTrackingProvider` **[NEW]** — ARKit provider delivering hand anchor updates
- `HandAnchor` **[NEW]** — anchor describing a tracked hand's position and skeleton
- `HandAnchor.chirality` **[NEW]** — `.left` or `.right` handedness
- `HandAnchor.skeleton` **[NEW]** — joint hierarchy with `definition.jointNames` and per-joint local transforms
- `AnchorUpdate` **[NEW]** — async stream update from an ARKit provider
- `Entity.loadModel(named:)` — RealityKit API for loading a USDZ asset
- `Entity.addChild(_:)` — adds a child entity to the scene hierarchy
- `WindowGroup` — standard SwiftUI window scene (used alongside `ImmersiveSpace`)

## Code Highlights

Minimal ImmersiveSpace definition:
```swift
@main
struct WorldApp: App {
    var body: some Scene {
        ImmersiveSpace(id: "solar") {
            SolarSystem()
        }
    }
}
```

Opening and dismissing the space:
```swift
struct SpaceControl: View {
    @Environment(\.openImmersiveSpace) private var openImmersiveSpace
    @Environment(\.dismissImmersiveSpace) private var dismissImmersiveSpace
    @State private var isSpaceHidden = true

    var body: some View {
        Button(isSpaceHidden ? "View Outer Space" : "Exit") {
            Task {
                if isSpaceHidden {
                    let result = await openImmersiveSpace(id: "solar")
                    // handle result
                } else {
                    await dismissImmersiveSpace()
                    isSpaceHidden = true
                }
            }
        }
    }
}
```

Dynamic immersion style switching on gesture:
```swift
.immersionStyle(selection: $currentStyle, in: .mixed, .progressive, .full)
```

Surrounding effects and hidden limbs:
```swift
ImmersiveSpace(id: "solar") {
    SolarSystem().preferredSurroundingsEffect(.systemDark)
}
.immersionStyle(selection: $currentStyle, in: .full)
.upperLimbVisibility(.hidden)
```

## Takeaways
- `ImmersiveSpace` is the third SwiftUI scene type (alongside `WindowGroup` and the volumetric window style) and is the entry point for fully immersive visionOS experiences.
- Use `openImmersiveSpace` and `dismissImmersiveSpace` environment actions (both async) to control space visibility; always handle the returned result.
- Immersion style can be switched dynamically at runtime — `.mixed` → `.progressive` → `.full` — using the `immersionStyle` modifier with a bound state variable.
- ARKit `HandTrackingProvider` combined with `upperLimbVisibility(.hidden)` enables seamless virtual hand replacement in full immersion experiences.

---
_Source: WWDC23 Session 10111 page (abstract, chapter summaries, code samples, and resource links)._
