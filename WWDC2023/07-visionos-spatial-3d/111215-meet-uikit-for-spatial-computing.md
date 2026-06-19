# Meet UIKit for Spatial Computing
**WWDC23 · Session 111215** · [Watch](https://developer.apple.com/videos/play/wwdc2023/111215/)

_Platforms:_ visionOS 1

## Overview
This session walks through the complete process of bringing an existing UIKit iPad app to visionOS. Starting from a UIKit pixel-art animation app, the presenters demonstrate how to add the visionOS destination in Xcode, resolve unavailable API compile errors, polish the visual appearance using semantic colors and materials, wire up hover effects using new `UIView.hoverStyle` API, adapt gesture recognizers for the eye-and-hand input model, and then extend the app beyond its 2D window bounds using ornaments and RealityKit content via `UIHostingController`.

UIKit apps run in visionOS with minimal changes because UIKit's standard components — controls, lists, navigation containers, sheets, alerts, popovers — already adapt to the platform's glass material, hover effects, and spatial presentation styles automatically. The required effort is removing deprecated or unavailable APIs and applying semantic styling.

## Key Topics

### Getting Started
- Add visionOS as a run destination in the Xcode project's General tab
- App icons on visionOS are three-layer images (foreground, middle, background) that respond dynamically when viewed
- Compile errors typically fall into two categories: APIs deprecated before iOS 14 (unavailable on visionOS) and APIs that do not conceptually translate (e.g., `UIDeviceOrientation`, `UIScreen`, `UITabBar.leadingAccessoryView`)
- `UIPencilInteraction` is unavailable on visionOS (no Apple Pencil support); conditionalize with `#if` or runtime checks

### Platform Differences — Unavailable APIs
- `UIDeviceOrientation` – visionOS has no device orientation concept
- `UIScreen` – single hardware screen representation does not apply
- `UITabBar.leadingAccessoryView` / `trailingAccessoryView` – tab bar has a different spatial design (placed as an ornament on the leading edge of the window)
- APIs deprecated before iOS 14 – all unavailable; use this as an opportunity to modernize shared code
- Check documentation for the full list of unavailable API

### Polishing: Semantic Colors and Materials
- Use semantic colors (`UIColor.label`, `UIColor.secondaryLabel`, `UIColor.systemBackground`, etc.) instead of RGB-defined colors; they automatically adapt to visionOS's glass background and vibrant rendering
- Semantic font styles (Headline, Body, etc.) scale with Dynamic Type and look correct on visionOS
- `UILabel` instances using semantic colors receive vibrancy automatically
- `UITextField.borderStyle = .roundedRect` produces the recessed appearance characteristic of visionOS text fields
- Glass background is automatically applied to every `UINavigationController` and `UISplitViewController`
- Override `preferredContainerBackgroundStyle` on `UIViewController` to return `.automatic`, `.glass`, or `.hidden`
- Materials adapt contrast and color balance to lighting conditions; there is no light/dark appearance distinction on visionOS

### Polishing: Hover Effects
- System provides hover highlight when the user looks at interactive elements; exact gaze location is never delivered to the app process
- Standard controls get hover effects automatically; custom views do not
- New `UIView.hoverStyle` property (visionOS): set to a `UIHoverStyle` with `.effect` (`.highlight` or `.lift`) and an optional `UIShape`
- `UIShape` options: `.rect`, `.roundedRect(cornerRadius:)`, `.circle`, `.capsule`, and more — shape defines the hover highlight boundary
- Set `hoverStyle = nil` to disable hover effects on a view

### Polishing: Input
- Look + indirect pinch = `TapGesture`; pinch + move + release = `PanGesture`; direct touch also supported
- Trackpad and accessibility technologies (VoiceOver, Switch Control) work automatically with standard gesture recognizers
- Maximum two simultaneous inputs (one per hand); replace four-finger gestures with two-finger equivalents when `traitCollection.userInterfaceIdiom == .reality`

### Spatial Presentations
- Sheets: push the presenting view controller back and dim it; do not dismiss on outside taps regardless of `isModalInPresentation`; always present from the appropriate view controller
- Alerts: display a 2D app icon at the top; follow same presentation rules as sheets
- Popovers: not constrained to scene bounds on visionOS (similar to macOS); avoid hard-coding `permittedArrowDirections` — use `.any` on visionOS, which allows the system to choose the best placement; check `traitCollection.userInterfaceIdiom == .reality`

### Ornaments
- Ornaments place content outside the scene window, around the scene within reasonable limits
- Created with `UIHostingOrnament(sceneAlignment:contentAlignment:)` — host any SwiftUI content
- `sceneAlignment`: where on the scene boundary the ornament attaches (`.bottom`, `.leading`, `.top`, etc.)
- `contentAlignment`: how the ornament content aligns relative to the attachment point
- Set `UIViewController.ornaments` to an array of `UIHostingOrnament` instances
- Ornaments share their view controller's lifecycle: hidden/removed when the view controller leaves the hierarchy
- Ornaments do not automatically add a background; apply `.glassBackgroundEffect()` SwiftUI modifier for glass treatment
- System components already use ornaments: `UITabBar` (leading ornament), Safari toolbar (above content), Freeform bottom toolbar

### Adding RealityKit via UIHostingController
- `RealityView` (SwiftUI) hosts RealityKit entities; use `UIHostingController` to embed it in a UIKit hierarchy
- Steps: create SwiftUI view containing `RealityView` → `UIHostingController(rootView:)` → `addChild(_:)` → `addSubview(_:)` → `didMove(toParent:)`
- No need to rewrite UIKit app; UIHostingController bridges SwiftUI/RealityKit capabilities into existing UIKit architecture

## APIs & Frameworks

- **UIKit** – primary framework for the session; visionOS adaptation
- `UIView.hoverStyle` **[NEW on visionOS]** – `UIHoverStyle?`; sets hover effect and shape on any view
- `UIHoverStyle(effect:shape:)` **[NEW]** – `.highlight`, `.lift` effects; custom `UIShape`
- `UIShape` **[NEW]** – `.rect`, `.roundedRect(cornerRadius:)`, `.circle`, `.capsule`, `.beam(preferredLength:axis:)`
- `UIViewController.preferredContainerBackgroundStyle` **[NEW on visionOS]** – `.automatic`, `.glass`, `.hidden`
- `UIContainerBackgroundStyle` **[NEW]** – enum for container glass/hidden control
- `UIHostingOrnament(sceneAlignment:contentAlignment:content:)` **[NEW]** – creates a visionOS ornament hosting SwiftUI content
- `UIViewController.ornaments` **[NEW on visionOS]** – `[UIHostingOrnament]`; assigns ornaments to a view controller
- `UIPopoverPresentationController.permittedArrowDirections` – use `.any` on visionOS to allow system-preferred placement
- `traitCollection.userInterfaceIdiom == .reality` **[NEW]** – detects visionOS idiom at runtime
- `UIColor.label`, `UIColor.secondaryLabel`, `UIColor.systemBackground` – semantic colors; adapt automatically to visionOS vibrancy
- `UITextField.borderStyle = .roundedRect` – produces recessed appearance on visionOS
- `UIHostingController` – embeds SwiftUI views (including `RealityView`) in UIKit hierarchy
- **RealityView** (SwiftUI) **[NEW]** – hosts RealityKit entities; accessed from UIKit via `UIHostingController`
- **RealityKit** – 3D rendering; entities parented in SwiftUI hierarchy via `RealityView`
- `UISwipeGestureRecognizer.numberOfTouchesRequired` – set to 2 on visionOS (max simultaneous inputs = 2)
- `.glassBackgroundEffect()` (SwiftUI modifier) **[NEW]** – glass material for ornament content

## Code Highlights

Platform-conditional popover arrow direction:
```swift
if traitCollection.userInterfaceIdiom == .reality {
    presentationController.permittedArrowDirections = .any
} else {
    presentationController.permittedArrowDirections = .right
}
```

Adding an ornament with glass background:
```swift
func showEditingControlsOrnament() {
    let ornament = UIHostingOrnament(sceneAlignment: .bottom, contentAlignment: .center) {
        EditingControlsView(model: controlsViewModel)
            .glassBackgroundEffect()
    }
    self.ornaments = [ornament]
}
```

Embedding RealityKit content via UIHostingController:
```swift
func showEntityPreview() {
    let entityView = PixelArtEntityView(model: entityViewModel)
    let controller = UIHostingController(rootView: entityView)
    addChild(controller)
    view.addSubview(controller.view)
    controller.didMove(toParent: self)
}
```

Semantic colors and rounded text field:
```swift
titleTextField.textColor = UIColor.label
authorLabel.textColor = UIColor.secondaryLabel
titleTextField.borderStyle = .roundedRect
```

Custom hover style with rounded rect shape:
```swift
self.hoverStyle = UIHoverStyle(
    effect: .highlight,
    shape: .roundedRect(cornerRadius: 8.0)
)
```

Adapting multi-touch gesture for visionOS (max 2 touches):
```swift
if traitCollection.userInterfaceIdiom == .reality {
    gesture.numberOfTouchesRequired = 2
} else {
    gesture.numberOfTouchesRequired = 4
}
```

## Takeaways
- Most UIKit iPad apps compile and run on visionOS with only minor changes: remove deprecated/unavailable APIs, conditionalize device-specific code with `traitCollection.userInterfaceIdiom == .reality`, and switch to semantic colors and fonts.
- Glass background, hover effects on standard controls, and spatial presentations (sheets, popovers) all work automatically — effort focuses on custom views and non-semantic color usage.
- `UIHostingOrnament` is the key API for placing toolbars, palettes, and controls outside the window bounds; ornaments are the visionOS idiom for surfaces like the system tab bar and Safari's toolbar.
- `UIHostingController` bridges UIKit and SwiftUI without a full rewrite, enabling `RealityView` and other new SwiftUI APIs to be adopted incrementally in existing UIKit apps.

---
_Source: WWDC23 Session 111215 page (abstract, chapter summaries, code samples, and transcript)._
