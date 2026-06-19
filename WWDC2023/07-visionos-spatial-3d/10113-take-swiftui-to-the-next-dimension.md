# Take SwiftUI to the next dimension
**WWDC23 · Session 10113** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10113/)

_Platforms:_ visionOS 1

## Overview
This session extends SwiftUI into three dimensions for visionOS, covering the APIs that bridge familiar 2D SwiftUI development into spatial computing. Using the "World" sample app as a guide, it introduces four major topic areas: Volumes (a new bounded 3D scene type), 3D views and layout (including `Model3D`, 3D frame modifiers, and 3D geometry effects), RealityView attachments (embedding live SwiftUI views alongside RealityKit entities), and 3D gestures (`SpatialTapGesture`, `DragGesture.translation3D`, `RotateGesture3D`, and `targetedToEntity`).

The session demonstrates that SwiftUI's existing composition model, state management, and gesture system extend naturally into 3D — developers already familiar with SwiftUI can build volumetric and immersive experiences without abandoning the patterns they know.

## Key Topics

### Volumes
A `Volume` is a fixed-scale, horizontally-aligned 3D scene container that maintains the same physical size regardless of its distance from the user. Unlike windows (which scale dynamically with distance), volumes are ideal for displaying bounded 3D content without taking over the full space. Create one by applying `.windowStyle(.volumetric)` to a `WindowGroup`.

### Model3D
`Model3D` is a SwiftUI view that asynchronously loads USDZ or Reality assets, analogous to `AsyncImage` for images. It uses a phase-based API (`empty`, `failure`, `success`) to handle the loading lifecycle. Use `.resizable()` and `.scaledToFit()` on the loaded model to control sizing.

### 3D Layout
SwiftUI's layout system extends to 3D:
- **`frame(depth:alignment:)`** — adds a depth dimension to the existing `frame` modifier; `DepthAlignment` values include `.front`, `.back`, `.center`
- **`Rotation3DEffect`** — new geometry effect for 3D rotation; requires an axis (e.g., `.y`) and an `Angle`
- **`offset(z:)`** — Z-axis offset; complements existing `offset(x:y:)` for 3D positioning
- **`scaleEffect(_:anchor:anchorZ:)`** — 3D scale

### RealityView Attachments
The `attachments` closure in `RealityView` allows pairing tagged SwiftUI views with RealityKit entities. Attachments are live views (not snapshots) — they respond to state, run animations, and handle gestures. Look up an attachment entity by its tag using `attachments.entity(for:)`, then position it using RealityKit's `Entity` transform APIs (e.g., `lookAt`).

### 3D Gestures and Input
- **`SpatialTapGesture`** — tap gesture with `location3D: Point3D` property giving the 3D tap location in SwiftUI local coordinate space
- **`DragGesture`** — extended with `translation3D: Vector3D` for 3D drag translation
- **`RotateGesture3D`** — new gesture measuring unconstrained 3D rotation (`rotation: Rotation3D`) via hand tracking
- **`MagnifyGesture`** — existing gesture; `magnification: CGFloat` for scale
- **`targetedToEntity(_:)`** — new gesture modifier; filters gesture to fire only when targeting a specific entity or its descendants; adds coordinate space conversion helpers to the gesture value
- For an entity to receive input, it needs `InputTargetComponent` and a `CollisionComponent` (defines interactive region shape)
- **`AffineTransform3D`** — represents scale, rotation, and translation in 3D; used with combined gestures
- Use `.updating(_:body:)` to track transient gesture state; state auto-resets on gesture failure

### Coordinate Space Conversion
`targetedToEntity` adds `convert(_:from:to:)` helpers to the gesture value for converting between SwiftUI local coordinate space (points) and the RealityView scene coordinate space (meters).

## APIs & Frameworks

- **SwiftUI** (visionOS)
  - `WindowGroup` with `.windowStyle(.volumetric)` **[NEW]** — creates a Volume scene
  - `Model3D` **[NEW]** — async USDZ/Reality asset loader
    - `init(named:bundle:content:)` — named asset initializer
    - `init(url:content:)` — URL-based initializer
    - `ResolvedModel3D` — success phase; supports `.resizable()`, `.scaledToFit()`, `.scaledToFill()`
    - `Model3DPhase` — `.empty`, `.failure(Error)`, `.success(ResolvedModel3D)`
  - `frame(depth:alignment:)` **[NEW]** — depth dimension for layout
    - `DepthAlignment` — `.front`, `.back`, `.center`
  - `Rotation3DEffect` **[NEW]** — `rotation3DEffect(_:axis:anchor:anchorZ:perspective:)`
  - `offset(z:)` **[NEW]** — Z-axis positioning modifier
  - `scaleEffect(_:anchor:anchorZ:)` **[NEW]** — 3D scale modifier
  - `RealityView` **[NEW]** (from RealityKit integration)
    - `init(make:update:attachments:)` — closure receives `RealityViewContent` and `RealityViewAttachments`
    - `RealityViewAttachments.entity(for:)` — look up attachment entity by tag
  - `.tag(_:)` modifier on SwiftUI views — labels views for use as RealityView attachments
  - `SpatialTapGesture` **[NEW]**
    - `.location3D: Point3D` — 3D tap location in SwiftUI coordinate space
  - `DragGesture` — extended with:
    - `.translation3D: Vector3D` **[NEW]** — 3D drag translation
  - `RotateGesture3D` **[NEW]** — unconstrained 3D rotation gesture
    - `.rotation: Rotation3D` — measured rotation
  - `MagnifyGesture` — existing; `.magnification: CGFloat`
  - `.targetedToEntity(_:)` gesture modifier **[NEW]** — restricts gesture to a specific RealityKit entity; adds `convert(_:from:to:)` on gesture value
  - `.simultaneously(with:)` — existing combinator; used to compose drag + magnify + rotate
  - `AffineTransform3D` **[NEW]** — 3D affine transform combining scale, rotation, translation
  - `TimelineView` — existing; used to drive time-based 3D animations
- **RealityKit**
  - `InputTargetComponent` — required on an entity to receive SwiftUI gestures
  - `CollisionComponent` — defines entity's interactive region shape (e.g., `.sphere(radius:)`)
  - `Entity` — base RealityKit entity type
    - `lookAt(_:from:relativeTo:)` — orient label entity toward a point on a surface
  - `RealityViewContent` — provides `add(_:)` to insert entities into the scene
  - `Vector3D`, `Point3D`, `Size3D`, `Rotation3D` — 3D geometric types (from Spatial framework)

## Code Highlights

Volumetric window and Model3D:
```swift
@main struct WorldApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .windowStyle(.volumetric)
    }
}

struct MoonView: View {
    var body: some View {
        Model3D(named: "Moon") { phase in
            switch phase {
            case .empty: ProgressView()
            case let .failure(error): Text(error.localizedDescription)
            case let .success(model):
                model.resizable().scaledToFit()
            }
        }
    }
}
```

3D frame and rotation:
```swift
ForEach(objects) { object in
    CelestialObjectView(named: object.name)
        .frame(width: object.size, height: object.size, depth: object.size,
               alignment: .init(horizontal: .center, vertical: .center, depth: .front))
        .rotation3DEffect(.degrees(angle), axis: .y)
}
```

RealityView with attachments:
```swift
RealityView { content, attachments in
    content.add(earthEntity)
    for place in favoritePlaces {
        if let label = attachments.entity(for: place.id) {
            label.position = place.surfacePosition
            content.add(label)
        }
    }
} attachments: {
    ForEach(favoritePlaces) { place in
        Text(place.name)
            .padding()
            .glassBackgroundEffect()
            .tag(place.id)
    }
}
```

Combined 3D manipulation gesture:
```swift
var manipulationGesture: some Gesture<AffineTransform3D> {
    DragGesture()
        .simultaneously(with: MagnifyGesture())
        .simultaneously(with: RotateGesture3D())
        .map { gesture in
            let translation = gesture.first?.first?.translation3D ?? .zero
            let magnification = gesture.first?.second?.magnification ?? 1
            let size = Size3D(width: magnification, height: magnification, depth: magnification)
            let rotation = gesture.second?.rotation ?? .identity
            return AffineTransform3D(scale: size, rotation: rotation, translation: translation)
        }
}
```

SpatialTapGesture with targetedToEntity:
```swift
.gesture(
    SpatialTapGesture()
        .targetedToEntity(earthEntity)
        .onEnded { value in
            let sceneLocation = value.convert(value.location3D, from: .local, to: .scene)
            addFavoritePlace(at: sceneLocation)
        }
)
```

## Takeaways

- Volumes (`.windowStyle(.volumetric)`) are the right scene type for bounded 3D experiences — they maintain fixed physical size at any distance and support viewing from any angle without requiring a full immersive space.
- `Model3D` loads USDZ assets asynchronously with a phase-based API that mirrors `AsyncImage`; combine with `frame(depth:alignment:)` and `Rotation3DEffect` to compose full 3D layouts using familiar SwiftUI modifiers.
- RealityView `attachments` enable live SwiftUI views (with state, animations, gestures) to be anchored to specific RealityKit entities — use `.tag(_:)` on SwiftUI views and `attachments.entity(for:)` in the `make` closure.
- Combine `SpatialTapGesture`, `DragGesture` (with `translation3D`), `MagnifyGesture`, and `RotateGesture3D` via `.simultaneously(with:)` for full 6DOF manipulation; use `.targetedToEntity(_:)` to scope gestures to specific entities and get automatic coordinate space conversion.

---
_Source: WWDC23 Session 10113 page (abstract, chapters, transcript, and code samples)._
