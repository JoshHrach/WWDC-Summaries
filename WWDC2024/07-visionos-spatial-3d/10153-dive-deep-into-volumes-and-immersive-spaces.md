# Dive Deep into Volumes and Immersive Spaces
**WWDC24 · Session 10153** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10153/)

_Platforms:_ visionOS 2

## Overview
visionOS 2 delivers a substantial set of new SwiftUI and RealityKit APIs for customizing how volumetric apps present and behave. The session builds the BOT-anist sample app to demonstrate each feature: baseplates, resizable volumes, toolbars, ornaments, viewpoints, world alignment, dynamic scale, and full immersive spaces with coordinate conversions.

The second half covers immersive space enhancements: a new `ImmersiveSpace` coordinate space for conversions between RealityKit and SwiftUI worlds, custom immersion amounts with the Digital Crown, anchored UI placement using `SpatialTrackingSession`, and a `preferredSurroundingsEffect` API to tint passthrough to match environment lighting.

## Key Topics

### Volumes
The **baseplate** (automatic in visionOS 2) fades in at the volume's bottom edge when looked at, guiding users to resize handles. `volumeBaseplateVisibility(.visible/.hidden)` controls it explicitly. Volumes now support resize handles: by specifying `minWidth/maxWidth/minDepth/maxDepth` on child views the volume becomes resizable. Programmatic resize is driven by updating frame values in state. **Toolbars** float below the volume as bottom ornaments and auto-move to the nearest viewpoint. Custom **ornaments** can be placed at any `UnitPoint3D` scene anchor (e.g., `.topBack`). The `onVolumeViewpointChange` modifier fires whenever the user moves to a new viewpoint; `supportedVolumeViewpoints` restricts which sides show controls. Two new volume-level modifiers: `volumeWorldAlignment(.gravityAligned)` keeps the volume floor-parallel regardless of tilt, and `defaultWorldScalingBehavior(.dynamic)` makes the volume scale like windows as distance increases.

### Immersive Spaces
`RealityViewContent.transform(from:to:)` converts entity transforms between RealityKit scene spaces and the SwiftUI `.immersiveSpace` coordinate space, enabling a seamless visual handoff when moving content from a volume to an immersive space. `ImmersionStyle.progressive(_:initialAmount:)` accepts a custom range and initial value, letting apps set precise starting immersion. `onImmersionChange` modifier provides a context with the current `amount` (0–1). `SpatialTrackingSession` combined with `AnchorEntity(.plane(.horizontal, classification: .floor, ...))` enables placing content on real-world floors. `preferredSurroundingsEffect(.colorMultiply(color))` tints passthrough to match virtual environment colors.

## APIs & Frameworks

**SwiftUI (visionOS 2 additions)**
- `volumeBaseplateVisibility(_:)` modifier **[NEW]**
- `windowResizability(.contentSize)` — default for volumes (documented)
- `volumeWorldAlignment(_:)` modifier **[NEW]**
- `defaultWorldScalingBehavior(_:)` scene modifier **[NEW]**
- `onVolumeViewpointChange(updateStrategy:_:)` modifier **[NEW]**
- `supportedVolumeViewpoints(_:)` modifier **[NEW]**
- `Viewpoint3D.SquareAzimuth` (`.front`, `.left`, `.right`, `.back`) **[NEW]**
- `Viewpoint3D.SquareAzimuth.Set` **[NEW]**
- `.ornament(attachmentAnchor: .scene(UnitPoint3D))` for volumes **[NEW/enhanced]**
- `ImmersionStyle.progressive(_:initialAmount:)` **[NEW]**
- `onImmersionChange { context in }` modifier **[NEW]**
- `preferredSurroundingsEffect(_:)` modifier **[NEW]**
- `SurroundingsEffect.colorMultiply(_:)` **[NEW]**
- `.toolbar { ToolbarItem { ... } }` with `.bottomOrnament` placement (enhanced auto-placement)
- `TabSection`, `TabSectionGroup`

**RealityKit / SwiftUI interop**
- `RealityViewContent.transform(from:to:)` — coordinate space conversion **[NEW]**
- Named coordinate space `.immersiveSpace` **[introduced visionOS 1.1, highlighted]**
- `SpatialTrackingSession` **[NEW]**
  - `SpatialTrackingSession.Configuration(tracking: [.plane])`
  - `session.run(_:)` async
- `AnchorEntity(.plane(.horizontal, classification: .floor, minimumBounds:))` **[NEW classification]**
- `SpatialTapGesture(coordinateSpace: .immersiveSpace)` **[NEW param]**
- `EntityTargetValue.convert(_:from:to:)` **[NEW]**
- `PlantComponent: Component` (custom)
- `CharacterControllerComponent.Collision` (existing)

## Code Highlights

```swift
// Resizable volume
WindowGroup(id: "RobotExploration") {
    ExplorationView()
        .frame(minWidth: 900, maxWidth: 1800, minHeight: 500, maxHeight: 1000)
        .frame(minDepth: 900, maxDepth: 1800)
}
.windowStyle(.volumetric)

// Custom immersion range
@State private var immersionStyle: ImmersionStyle = .progressive(0.2...1.0, initialAmount: 0.8)

// Tint passthrough
.preferredSurroundingsEffect(SurroundingsEffect.colorMultiply(appModel.tintColor ?? .clear))
```

## Takeaways
- Enable volume resizability by specifying `min/max` frame values so users can scale content using the new corner handles.
- Use `onVolumeViewpointChange` to animate content (e.g., a character waving) in response to the user moving around the volume.
- Use `RealityViewContent.transform(from:to:)` for seamless visual handoffs between a volume and an immersive space.
- Apply `preferredSurroundingsEffect` to tint passthrough when a plant grows — a subtle but powerful immersion enhancer.

---
_Source: WWDC24 Session 10153 page (abstract, chapter summaries, code samples, and resource links)._
