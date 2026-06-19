# Meet SwiftUI for Spatial Computing
**WWDC23 · Session 10109** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10109/)

_Platforms:_ visionOS 1

## Overview
SwiftUI is the primary framework for building visionOS apps, and Apple built the visionOS system UI itself using SwiftUI. This session introduces the three fundamental scene types for spatial apps — windows, volumes, and Full Spaces — and demonstrates how existing SwiftUI knowledge transfers seamlessly to the new platform. New platform-specific behaviors include glass materials, hover effects, ornaments, 3D layout, the `Model3D` view, `RealityView` integration, `SpatialTapGesture`, `RealityView` attachments, `ImmersiveSpace`, and immersion style control.

All familiar SwiftUI controls (Button, Toggle, TabView, List, NavigationStack) adapt automatically to visionOS idioms: buttons gain vibrant material backgrounds and hover effects, TabView hangs off the leading edge of windows, and controls respond to eye-and-hand indirect gestures without any code changes.

## Key Topics

### Three Scene Types
- **Window** (`WindowGroup`) – familiar 2D windowed UI; supports multiple windows; glass background automatically applied; same navigation containers as iPadOS (TabView, NavigationStack, NavigationSplitView, List)
- **Volume** (`WindowGroup` + `.windowStyle(.volumetric)`) – 3D bounded window for displaying 3D content; can appear alongside other apps in Shared Space; `.defaultSize(width:height:depth:)` sets initial 3D dimensions
- **Full Space** (`ImmersiveSpace`) – hides other apps; lets the app place content anywhere; supports mixed, progressive, or full immersion styles

### Windows
- Glass background material applied automatically
- TabView appears as an ornament on the leading edge; expands labels on gaze
- Vibrant materials (`regularMaterial`, `thinMaterial`, etc.) for visual hierarchy in window content
- No light/dark appearance on visionOS; materials adapt automatically to the environment
- `.ornament(attachmentAnchor:contentAlignment:)` modifier for custom accessory views outside window bounds

### Materials and Visual Hierarchy
- Glass window background provides base transparency; vibrant materials compose on top
- Hierarchical shape styles: `.primary`, `.secondary`, `.tertiary` automatically adapt within their material context
- Use `.background(.regularMaterial, in: .rect(cornerRadius:))` for card-style sections

### Hover Effects (Interactive Regions)
- System applies hover feedback when user looks at interactive elements
- Standard controls (Button, Toggle, TextField, etc.) get hover effects automatically
- Custom button styles must add `.hoverEffect()` to receive the highlight
- Hover effects run outside the app process to protect user privacy (no gaze data exposed to app)
- `.hoverEffect()` selects an automatic appropriate style (highlight by default); shape matches the view's background shape

### Interaction Model
- Primary: look + indirect pinch (maps to TapGesture)
- Secondary: direct touch with hands
- Tertiary: connected trackpad or pointer
- Hardware keyboard: keyboard shortcuts, Focus, key modifiers
- Accessibility: VoiceOver, Switch Control, Dwell Control (eyes-only navigation), Dynamic Type, Invert Colors
- Existing `TapGesture` and `DragGesture` adapt to all input modes automatically

### Volumes and 3D Content
- `Model3D(named:)` – async-loading equivalent of `Image` for 3D assets; accepts placeholder closure
- ZStack is depth-aware: layout resolves in all 3 dimensions
- `.rotation3DEffect(_:axis:)` – 3D rotation; `.scaleEffect3D(_:)`, `.offset3D(_:)` — full 3D geometry effects
- `.padding3D(.front, _:)` – adds depth-axis spacing between views
- `.glassBackgroundEffect(in:)` – applies glass treatment to a shape (e.g., controls panel inside a volume)

### RealityView
- SwiftUI view that provides full access to RealityKit; renders RealityKit entities inline with SwiftUI layout
- `make` closure: async, loads `ModelEntity` or other RealityKit content and adds to `RealityViewContent`
- `update` closure: responds to SwiftUI state changes
- `attachments` closure + `attachments.entity(for:)`: embed arbitrary SwiftUI views as RealityKit entities (placed anywhere in the 3D scene)
- `SpatialTapGesture` – tap gesture with full 3D tap location
- `.targetedToAnyEntity()` gesture modifier – provides tapped entity and local coordinate context

### ImmersiveSpace and Immersion Styles
- `ImmersiveSpace(id:)` scene in the app body; contains a RealityKit/SwiftUI scene
- `openImmersiveSpace` environment action: `@Environment(\.openImmersiveSpace) var openImmersiveSpace`; called with an ID
- `.immersionStyle(selection:in:)` scene modifier – sets supported and current immersion style
- `ImmersionStyle` values: `.mixed` (content coexists with real world), `.progressive` (user controls depth with Digital Crown), `.full` (hides real world completely; must provide virtual environment)
- ARKit integration available in Full Spaces for world tracking, scene understanding, hand tracking

## APIs & Frameworks

- **SwiftUI** – primary framework for visionOS app development **[NEW visionOS support]**
- `WindowGroup` – windowed and volumetric scenes
- `.windowStyle(.volumetric)` **[NEW]** – declares a volumetric window
- `.defaultSize(width:height:depth:)` **[NEW]** – 3D default window size
- `ImmersiveSpace` **[NEW]** – Full Space scene type
- `.immersionStyle(selection:in:)` **[NEW]** – immersion style modifier
- `ImmersionStyle` **[NEW]** – `.mixed`, `.progressive`, `.full`
- `openImmersiveSpace` environment action **[NEW]** – programmatically open an ImmersiveSpace
- `dismissImmersiveSpace` environment action **[NEW]** – dismiss the open ImmersiveSpace
- `.ornament(attachmentAnchor:contentAlignment:)` modifier **[NEW]** – places accessory views relative to a window
- `.hoverEffect()` modifier **[NEW on visionOS]** – adds highlight on gaze
- `.glassBackgroundEffect(in:)` modifier **[NEW]** – glass material treatment
- `Model3D(named:)` **[NEW]** – async-loading SwiftUI view for 3D assets
- `RealityView` **[NEW]** – SwiftUI view embedding RealityKit content; `make`, `update`, `attachments` closures
- `RealityViewContent` – parameter in `RealityView` closures; `.add(_:)` for entities
- `RealityViewAttachments` – provides `.entity(for:)` to look up SwiftUI attachment entities
- `.tag(_:)` on attachment views – identifies attachments in `RealityViewAttachments`
- `SpatialTapGesture` **[NEW]** – tap gesture with 3D location (`value.location3D`)
- `.targetedToAnyEntity()` modifier **[NEW]** – constrains gesture to RealityKit entities; provides entity and local coordinate context
- `.rotation3DEffect(_:axis:)` – updated for visionOS 3D rendering
- `.padding3D(_:_:)` **[NEW]** – depth-axis padding
- `ModelEntity` – RealityKit entity type for 3D models; `ModelEntity(named:)` async initializer
- **ARKit** – world tracking, scene understanding, hand tracking in Full Spaces
- **RealityKit** – 3D rendering, physics, materials, custom shaders
- Hierarchical shape styles: `.secondary`, `.tertiary` with vibrant material adaptation
- `.background(_:in:)` modifier with `ShapeStyle` and `Shape` parameter

## Code Highlights

Volumetric window with a 3D globe:
```swift
@main
struct WorldApp: App {
    var body: some Scene {
        WindowGroup { ContentView() }
        WindowGroup { Globe() }
            .windowStyle(.volumetric)
            .defaultSize(width: 600, height: 600, depth: 600)
        ImmersiveSpace(id: "solar-system") { SolarSystem() }
            .immersionStyle(selection: $selectedStyle, in: .full)
    }
}
```

Model3D with tap gesture and 3D rotation:
```swift
Model3D(named: "Earth")
    .rotation3DEffect(rotation, axis: .y)
    .onTapGesture {
        withAnimation(.bouncy) { rotation.degrees += randomRotation() }
    }
    .padding3D(.front, 200)
```

RealityView with attachments and SpatialTapGesture:
```swift
RealityView { content in
    if let earth = try? await ModelEntity(named: "Earth") {
        content.add(earth)
    }
} update: { content, attachments in
    if let pin = attachments.entity(for: "pin") {
        content.add(pin)
    }
} attachments: {
    if let pinLocation { GlobePin(pinLocation: pinLocation).tag("pin") }
}
.gesture(
    SpatialTapGesture().targetedToAnyEntity().onEnded { value in
        pinLocation = lookUpLocation(at: value)
    }
)
```

Opening a Full Space:
```swift
@Environment(\.openImmersiveSpace) var openImmersiveSpace

Button("View Outer Space") {
    Task { await openImmersiveSpace(id: "solar-system") }
}
```

Custom button style with hover effect:
```swift
struct FunFactButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding()
            .background(.regularMaterial, in: .rect(cornerRadius: 12))
            .hoverEffect()
            .scaleEffect(configuration.isPressed ? 0.95 : 1)
    }
}
```

## Takeaways
- The three visionOS scene types — windows, volumes, and Full Spaces — each serve a distinct purpose and can be mixed freely within the same app.
- Existing SwiftUI views and controls require no changes for the shared-space experience; they receive appropriate platform defaults (glass backgrounds, hover effects, TabView ornament style) automatically.
- Custom button styles and interactive views must add `.hoverEffect()` manually — without it there is no gaze feedback, making custom interactive content feel unresponsive.
- `RealityView` is the bridge between SwiftUI layout and full RealityKit 3D capabilities; the `attachments` API makes it easy to embed SwiftUI views as positioned 3D entities.

---
_Source: WWDC23 Session 10109 page (abstract, chapter summaries, code samples, and resource links)._
