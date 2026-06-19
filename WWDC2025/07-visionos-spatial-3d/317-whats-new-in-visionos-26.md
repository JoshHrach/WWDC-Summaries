# What's new in visionOS 26
**WWDC25 · Session 317** · [Watch](https://developer.apple.com/videos/play/wwdc2025/317/)

_Platforms:_ visionOS 26, macOS Tahoe 26

## Overview
This broad survey session covers eight areas of visionOS 26 improvement: volumetric APIs (3D layout, presentations in volumes, dynamic bounds, object manipulation), system features (Foundation Models, SpeechAnalyzer, scene/widget persistence), games and spatial accessories (3x faster hand tracking, PS VR2 Sense controller, Logitech Muse, TabletopKit updates), shared experiences (Nearby Window Sharing, shared world anchors, Spatial Persona improvements), immersive media (Apple Projected Media Profile for 180°/360°/wide-FOV), spatial web (new HTML `<model>` element, video in Safari, Web Backdrop preview), and enterprise APIs (main camera in shared space, stereo camera, CameraRegionProvider, Protected Content, Window Follow Mode).

## Key Topics

### Volumetric features
**3D layout**: `VStackLayout().depthAlignment(.front/back/center)`, `rotation3DLayout(angle, axis:)` for layout-aware rotation. Many SwiftUI 2D modifiers now have 3D analogs.

**Presentations in volumes**: Alerts, sheets, menus, and popovers now work inside and on top of volumes. Break-through visual treatments ensure key UI is visible over 3D content.

**Dynamic bounds restrictions**: `.preferredWindowClippingMargins(.all, 400)` allows content to render outside app bounds for a more immersive feel without changing window size.

**Object manipulation**: `.manipulable()` modifier on `Model3D` for natural hand manipulation. RealityKit: `ManipulationComponent.configureEntity(_:)`. Quick Look 3D models get manipulation for free.

**Unified Coordinate Conversion API**: Move views and entities between SwiftUI, RealityKit, and ARKit coordinate spaces.

**RealityKit Observable**: Entities and animations conform to `Observable` — observe changes directly in SwiftUI views. SwiftUI gestures can be attached to entities via `GestureComponent`. `ViewAttachmentComponent` for inline UI in RealityKit scenes.

**Model3D enhancements**: Play/pause/resume animations, control animation time, load USD variants, load `.reality` file configurations.

**RealityView sizing**: `realityViewSizingBehavior` modifier.

**Spatial audio**: Spatial Audio Experience API spatializes audio per window/volume; move sounds between scenes.

### System features
**Foundation Models**: On-device large language model access, guided generation, tool calling.

**SpeechAnalyzer** **[NEW]**: New API for speech-to-text on iOS, macOS, visionOS using Swift-based speech recognition. SpeechTranscriber model for challenging tasks. Runs on-device.

**Scene/widget persistence**: Windows, volumes, and 2D/3D Quick Look content persist in the same physical location across restarts. SwiftUI restoration APIs: `.defaultLaunchBehavior(.suppressed)` and `.restorationBehavior(.disabled)`. Surface snapping: `@Environment(\.surfaceSnappingInfo)`.

**Widgets on visionOS**: Snap to walls/tables, automatic depth/dimension visual treatments, `levelOfDetail` API, `widgetTexture` API.

### Games and spatial accessories
**3x faster hand tracking** for immersive apps — no code changes required.

**Sony PlayStation VR2 Sense controller**: 6DoF wireless tracking, hand breakthrough, system navigation, haptics. Use `GameController` framework for Bluetooth discovery; `RealityKit` or `ARKit` for tracking.

**Logitech Muse**: Precision stylus input, 4 sensors (tip + side button, variable input), haptics.

**Increased memory limits**: High-end iPad games can now run on Vision Pro via App Store Connect.

**Progressive immersion**: Now supports landscape and portrait aspect ratios. Extended to Compositor Services.

**Compositor Services**: Hover effects (tracking areas texture), dynamic render quality control.

**macOS spatial rendering**: `RemoteImmersiveSpace` scene type, Compositor Services and ARKit now on macOS.

**TabletopKit**: **`CustomEquipmentState`** — add custom data fields to Equipment, auto-networked. **`CustomActions`** — define custom networked game actions (ownership changes, color updates).

### Shared experiences
**Nearby Window Sharing**: SharePlay-based shared experiences for co-located users. Works with existing SharePlay apps — no additional code. ARKit shared world anchors for precise shared content placement. Quick Look shared object manipulation.

**Spatial Personas**: Out of beta; improved hair, complexion, expressions.

### Immersive media
**Apple Projected Media Profile (APMP)**: Metadata-based approach for QuickTime/MPEG-4 files to signal 180°, 360°, wide-FOV content. Supports high-motion detection for viewer comfort. Automatically generates APMP metadata for select third-party cameras (Canon, GoPro, Insta360). Integration via AVKit, RealityKit, Quick Look, and WebKit.

**Immersive Media Support framework** **[NEW]**: Create, process, share Apple Immersive Video from production pipelines.

### Spatial web
**HTML `<model>` element**: Declarative 3D model embedding in web pages via USDZ; CSS-stylable, JS-configurable, drag to real world via Quick Look.

**Spatial video in Safari**: Add spatial videos (all APMP formats + Apple Immersive Video) via existing HTML `<video>` element for immersive full-screen playback.

**Web Backdrop** (developer preview): Custom immersive environments through HTML markup.

**Look to Scroll**: Eye-based hands-free scrolling in Safari, TV, Notes, Mail, and custom apps.

### Enterprise APIs
- Shared space main camera access (approved entitlement) — concurrent with other spatial apps
- Stereo camera access — both left and right main camera feeds simultaneously
- **`ARKit.CameraRegionProvider`** **[NEW]** — stabilized video feed of a selected region of interest, with contrast/vibrancy control
- **`.contentCaptureProtected()`** **[NEW]** — prevent screenshots, recordings, AirPlay, SharePlay of a view
- Window Follow Mode — app windows follow user position (licensed entitlement)
- Return to Service — enterprise device sharing with data erasure between sessions
- QuickStart enhancements — import saved Vision Pro setup from iCloud
- **`SharedCoordinateSpaceProvider`** in ARKit (managed entitlement) — co-locate users for enterprise collaboration

## APIs & Frameworks

### SwiftUI
- `VStackLayout().depthAlignment(.front/.back/.center)` **[NEW]**
- `.rotation3DLayout(_:axis:)` **[NEW]**
- `.preferredWindowClippingMargins(_:_:)` **[NEW]**
- `.manipulable()` **[NEW]** — `Model3D` view modifier
- `@Environment(\.surfaceSnappingInfo)` **[NEW]**
- `.defaultLaunchBehavior(.suppressed)` **[NEW]** — on Window scene
- `.restorationBehavior(.disabled)` **[NEW]** — on Window scene
- `.scrollInputBehavior(.enabled, for: .look)` **[NEW]** — Look to Scroll
- `Tab(role: .search)` morphing search (see SwiftUI session)

### RealityKit
- `ManipulationComponent.configureEntity(_:)` **[NEW]**
- `GestureComponent` **[NEW]**
- `ViewAttachmentComponent` **[NEW]**
- `EnvironmentBlendingComponent` **[NEW]**
- `MeshInstancesComponent` **[NEW]**
- `ImagePresentationComponent` **[NEW]**
- `AnchorStateEvents.DidAnchor` / `WillUnanchor` **[NEW]**
- `ARKitAnchorComponent` **[NEW]**
- AVIF texture support **[NEW]**
- `realityViewSizingBehavior(_:)` **[NEW]** — on `RealityView`
- `HoverEffectComponent` groupID **[NEW]**
- `RealityView.customPostProcessing(_:)` **[NEW]**

### ARKit
- Shared world anchors **[NEW]** — for Nearby Window Sharing
- **`CameraRegionProvider`** **[NEW]** — enterprise stabilized region feed
- **`SharedCoordinateSpaceProvider`** **[NEW]** — enterprise co-location (managed entitlement)
- `WorldTrackingProvider` — now available on macOS for `RemoteImmersiveSpace`

### Compositor Services
- Hover effects / tracking areas texture **[NEW]**
- Dynamic render quality control **[NEW]**

### Speech (new framework/API)
- **`SpeechAnalyzer`** **[NEW]** — on-device speech-to-text API
- **`SpeechTranscriber`** model **[NEW]**

### Immersive Media Support
- **`Immersive Media Support`** framework **[NEW]** — Apple Immersive Video production pipeline

### TabletopKit
- **`CustomEquipmentState`** **[NEW]** — custom networked data on Equipment
- **`CustomActions`** **[NEW]** — custom networked game actions

### UIKit
- `UIScrollView.lookToScrollAxes` **[NEW]** — Look to Scroll

### GameController
- PlayStation VR2 Sense controller support **[NEW]**
- Logitech Muse support **[NEW]**

### Enterprise
- `.contentCaptureProtected()` **[NEW]** — view modifier

## Code Highlights

```swift
// Depth alignment
VStackLayout().depthAlignment(.front) {
    ResizableLandmarkModel()
    LandmarkNameCard()
}

// Dynamic bounds
.preferredWindowClippingMargins(.all, 400)

// Object manipulation (SwiftUI)
ForEach(rocks) { rock in
    Model3D(named: rock.name) { model in
        model.model?.resizable().scaledToFit3D()
    }
    .manipulable()
}

// Object manipulation (RealityKit)
for rock in rocks {
    ManipulationComponent.configureEntity(rock)
    content.add(rock)
}

// Scene restoration
Window("Inspector", id: "Inspector") { InspectorView() }
    .defaultLaunchBehavior(.suppressed)
    .restorationBehavior(.disabled)

// Look to Scroll (SwiftUI)
ScrollView { HikeDetails() }
    .scrollInputBehavior(.enabled, for: .look)
```

## Takeaways
- Use `.manipulable()` on `Model3D` and `ManipulationComponent` in RealityKit for natural object manipulation — both get the same intuitive hand-rotation behavior with minimal code.
- Adopt `SpeechAnalyzer` for on-device speech recognition — it's already powering Notes, FaceTime, and Call Transcription, runs entirely on-device, and is faster than the previous generation model.
- Add APMP support for immersive media by integrating with AVKit/RealityKit/Quick Look — no new session types needed, and the system automatically handles third-party camera metadata for Canon, GoPro, and Insta360.
- For enterprise apps, implement `CameraRegionProvider` for stabilized region-of-interest feeds and `.contentCaptureProtected()` to protect sensitive views from screenshots and AirPlay sharing.

---
_Source: WWDC25 Session 317 page (abstract, chapter summaries, code samples, and resource links)._
