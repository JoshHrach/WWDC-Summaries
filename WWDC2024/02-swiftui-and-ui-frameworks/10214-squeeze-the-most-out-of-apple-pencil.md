# Squeeze the Most Out of Apple Pencil
**WWDC24 · Session 10214** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10214/)

_Platforms:_ iOS 18, iPadOS 18, visionOS 2

## Overview
New in iOS 18, iPadOS 18, and visionOS 2, PencilKit's tool picker has been significantly enhanced to support completely custom tools alongside the built-in system tools. Developers can now define the exact set of tools displayed in the picker—including multiple instances of the same tool type—and integrate their own drawing tools with matching controls for color, width, and custom properties.

Apple Pencil Pro introduces a rich set of new hardware features that apps can now integrate with: squeeze gesture detection, barrel roll angle sensing, and haptic feedback output. These capabilities allow apps to build more expressive drawing experiences and more intuitive gesture-based workflows without the user ever releasing the pencil.

The session walks through building a custom stamp tool integrated into PKToolPicker, then covers how to respond to squeeze events in both UIKit and SwiftUI, use roll angle for stroke variation, and generate canvas haptic feedback using the new UICanvasFeedbackGenerator and SwiftUI SensoryFeedback APIs.

## Key Topics

### Configuring the Tool Picker
`PKToolPicker` now accepts a `toolItems` initializer that lets you specify which tools appear and in what order. You can include multiple inking tools of the same type (e.g., two different colored pens), a ruler item, a Scribble item, and custom items. A new accessory `UIBarButtonItem` can be added to the picker's trailing edge for canvas-level actions like inserting a text box.

### Custom Tools in the Tool Picker
`PKToolPickerCustomItem` and its `Configuration` struct let you build fully custom tool items that integrate visually with the picker. You supply a rendered image closure (called whenever color, width, or opacity change), and optionally a view controller for extra property controls. PencilKit handles tool selection state; your app handles the actual rendering when the custom tool is active.

### Apple Pencil Pro Features and APIs
Apple Pencil Pro exposes squeeze gestures via `UIPencilInteraction` (UIKit) and the `.onPencilSqueeze` modifier (SwiftUI). The hover pose attached to squeeze events provides precise pencil location above the screen (position, z-offset, azimuth, altitude, roll). Barrel roll angle is now available on `UITouch` and `UIHoverGestureRecognizer` via `rollAngle`. New `UICanvasFeedbackGenerator` methods signal alignment snapping and shape completion with haptics; SwiftUI equivalents are `.sensoryFeedback(.alignment, ...)` and `.sensoryFeedback(.pathComplete, ...)`.

## APIs & Frameworks

- `PKToolPicker` **[NEW]** `toolItems:` initializer for custom tool sets
- `PKToolPickerItem` — base type for all tool picker items
- `PKToolPickerInkingItem` — inking tool item (pen, marker, etc.)
- `PKToolPickerEraserItem` — eraser tool item
- `PKToolPickerLassoItem` — lasso selection item
- `PKToolPickerRulerItem` — ruler toggle item
- `PKToolPickerScribbleItem` — Scribble mode item
- `PKToolPickerCustomItem` **[NEW]** — custom app-defined tool item
- `PKToolPickerCustomItem.Configuration` **[NEW]** — defines supported properties (color, width, opacity, custom VC)
- `PKToolPickerCustomItem.reloadImage()` **[NEW]** — triggers image closure re-execution on custom attribute change
- `PKToolPicker.accessoryItem` **[NEW]** — trailing `UIBarButtonItem` on the picker
- `UIPencilInteraction.Squeeze` **[NEW]** — squeeze gesture event type
- `UIPencilInteraction.Squeeze.phase` — gesture phase (began, ended, etc.)
- `UIPencilInteraction.Squeeze.hoverPose` **[NEW]** — pencil hover location and angles at squeeze time
- `UIPencilInteraction.preferredSqueezeAction` — system preference for squeeze behavior
- `UIPencilInteractionDelegate.pencilInteraction(_:didReceiveSqueeze:)` **[NEW]**
- `.onPencilSqueeze` **[NEW]** — SwiftUI modifier for squeeze events
- `@Environment(\.preferredPencilSqueezeAction)` **[NEW]** — SwiftUI environment value
- `UIHoverGestureRecognizer.rollAngle` **[NEW]** — barrel roll angle during hover
- `UITouch.rollAngle` **[NEW]** — barrel roll angle on touch events
- `UICanvasFeedbackGenerator` **[NEW]** — UIKit haptic feedback for canvas drawing
- `UICanvasFeedbackGenerator.alignmentOccurred(at:)` **[NEW]**
- `UICanvasFeedbackGenerator.pathCompleted(at:)` **[NEW]**
- `UIFeedbackGenerator` — updated to accept `view:` and location on all subclasses
- `SensoryFeedback.alignment` **[NEW]** — SwiftUI sensory feedback type
- `SensoryFeedback.pathComplete` **[NEW]** — SwiftUI sensory feedback type
- `.sensoryFeedback(_:trigger:)` — SwiftUI modifier
- `PKCanvasView` — PencilKit canvas; drawing disabled when custom tool is selected
- `UILabel.drawText(in:)` — recommended for rendering text on custom tool images

## Code Highlights

Respond to squeeze in UIKit and present a contextual palette at the hover location:
```swift
func pencilInteraction(_ interaction: UIPencilInteraction,
           didReceiveSqueeze squeeze: UIPencilInteraction.Squeeze) {
    if UIPencilInteraction.preferredSqueezeAction == .showContextualPalette &&
       squeeze.phase == .ended {
       let anchorPoint = squeeze.hoverPose?.location ?? myDefaultLocation
       presentMyContextualPaletteAtPosition(anchorPoint)
    }
}
```

SwiftUI equivalent using `.onPencilSqueeze`:
```swift
var body: some View {
    MyView()
        .onPencilSqueeze { phase in
            if preferredAction == .showContextualPalette, case let .ended(value) = phase {
                if let anchorPoint = value.hoverPose?.anchor {
                    contextualPaletteAnchor = .point(anchorPoint)
                }
                contextualPalettePresented = true
            }
        }
}
```

Canvas haptic feedback in SwiftUI:
```swift
MyView()
    .sensoryFeedback(.alignment, trigger: dragAlignedToGuide)
    .sensoryFeedback(.pathComplete, trigger: snappedToShape)
```

## Takeaways

- `PKToolPicker` now supports fully custom tool items via `PKToolPickerCustomItem`, enabling any app to integrate its own drawing tools alongside PencilKit's built-in tools.
- Apple Pencil Pro's squeeze gesture delivers hover pose data (location, roll, azimuth, altitude) that can be used to present context-sensitive palettes exactly where the pencil is hovering.
- `rollAngle` on `UITouch` and `UIHoverGestureRecognizer` enables expressive, orientation-aware strokes and previews; combining roll and azimuth provides a good fallback on non-Pro devices.
- Use `UICanvasFeedbackGenerator` or SwiftUI's `.sensoryFeedback` with `.alignment` and `.pathComplete` to deliver haptic feedback that makes snapping and shape recognition feel tactile.

---
_Source: WWDC24 Session 10214 page (abstract, chapter summaries, code samples, and resource links)._
