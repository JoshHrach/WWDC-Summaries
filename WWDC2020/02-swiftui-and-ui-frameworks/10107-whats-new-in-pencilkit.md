# What's New in PencilKit
**WWDC20 · Session 10107** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10107/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11 (Mac Catalyst — PKCanvasView only)

## Overview
PencilKit received notable enhancements in iOS 14, including zero-adoption improvements to the drawing experience (smart selection, insert space, new color picker), a redesigned finger-vs-pencil drawing policy API, independent PKToolPicker instances per canvas, and — most significantly — programmatic access to the internal stroke data of a PKDrawing.

Many improvements require no changes to existing apps: smart handwriting selection, insert-space gestures, and the new UIKit system color picker are automatically available to all PencilKit clients. The drawing policy API replaces the deprecated `allowsFingerDrawing` property with a richer `PKCanvasViewDrawingPolicy` enum that respects the new system-wide "Prefer Only Pencil Drawing" setting.

Mac Catalyst now supports `PKCanvasView` (though not the tool picker), and latency restrictions on visual effect views were lifted, allowing blur materials and bars anywhere in a PencilKit app.

## Key Topics

### Zero-Adoption Improvements
- **Smart Selection** — rich selection of handwritten content: double-tap for word, double-tap again for line, grab handles for range; velocity-sensitive tap-and-pan for noncontiguous selection
- **Insert Space** — tap whitespace, choose "Insert Space," drag handle to adjust, write new content in the gap
- **New Color Picker** — system `UIColorPickerViewController`; spectrum, sliders, saved colors, eye dropper
- **Latency Improvements** — visual effect views, blur materials, and bars no longer impact drawing latency

### Finger vs. Pencil Drawing Policy
The old `allowsFingerDrawing` property is deprecated. A new global system setting ("Prefer Only Pencil Drawing") in the Apple Pencil settings pane is accessible via `UIPencilInteraction.prefersPencilOnlyDrawing`. PencilKit surfaces this toggle in the tool picker as "Draw with Finger." The new `PKCanvasViewDrawingPolicy` enum replaces the deprecated property.

### Independent PKToolPicker Instances
Previously a single shared tool picker per `UIWindow`; now each canvas can have its own `PKToolPicker` instance with independent state (selected tool, drawing policy controls, color UI style). Apps must retain a strong reference to each tool picker instance.

### Stroke Data Access
iOS 14 exposes the internal structure of `PKDrawing` — strokes, inks, paths, and individual touch points — enabling annotation tools, animation, handwriting recognition, and machine learning on drawing content. Covered in depth in Session 10148.

## APIs & Frameworks

### PencilKit Framework
- `PKCanvasView` — main drawing canvas
  - `drawingPolicy: PKCanvasViewDrawingPolicy` **[NEW]** — replaces deprecated `allowsFingerDrawing`
  - Mac Catalyst support **[NEW]** (PKToolPicker not supported on Mac)
- `PKCanvasViewDrawingPolicy` **[NEW]** — enum with cases:
  - `.anyInput` — finger and Pencil both draw
  - `.pencilOnly` — only Pencil draws; finger scrolls/selects
  - `.default` — respects tool picker visibility: if shown, respects `UIPencilInteraction.prefersPencilOnlyDrawing`; if hidden, pencil only
- `PKToolPicker` **[UPDATED]**
  - Now instantiable independently (no longer just shared per window) **[NEW]**
  - `showsDrawingPolicyControls: Bool` **[NEW]** — show/hide "Draw with Finger" toggle in picker UI
  - `selectedTool: PKTool` — set default tool per picker instance
- `PKInkingTool` — ink tool configuration (pen, marker, pencil ink types)
- `PKDrawing` — drawing model; now exposes `strokes: [PKStroke]` **[NEW]**
- `PKStroke` **[NEW]** — individual stroke with ink, path, and points
- `PKStrokePath` **[NEW]** — path of a stroke
- `PKStrokePoint` **[NEW]** — individual point with location, force, altitude, azimuth, speed
- `allowsFingerDrawing` **[DEPRECATED]** — replaced by `drawingPolicy`

### UIKit Integration
- `UIPencilInteraction.prefersPencilOnlyDrawing` — system-wide "Prefer Only Pencil Drawing" setting; PencilKit reads this automatically; custom drawing engines should query this manually
- `UIColorPickerViewController` — system color picker now used by PencilKit tool picker

## Code Highlights

Setting drawing policy per canvas:
```swift
var drawingPolicy: PKCanvasViewDrawingPolicy
```

Hiding drawing policy controls in the tool picker (for Pencil-only apps):
```swift
PKToolPicker.showsDrawingPolicyControls  // set to false for pencil-only canvases
```

Configuring independent tool pickers per canvas:
```swift
notesCanvas.drawingPolicy = .default
notesToolPicker.showsDrawingPolicyControls = true
notesToolPicker.selectedTool = PKInkingTool(.pen, color: .black, width: 2)

drawingCanvas.drawingPolicy = .anyInput
drawingToolPicker.showsDrawingPolicyControls = false
drawingToolPicker.selectedTool = PKInkingTool(.marker, color: .purple, width: 20)
```

## Takeaways

- Smart selection, insert space, and the new color picker are free for all PencilKit clients — no code changes required.
- Migrate from `allowsFingerDrawing` to `PKCanvasViewDrawingPolicy` to correctly support the system-wide "Prefer Only Pencil Drawing" preference introduced in iOS 14.
- Each canvas can now have its own independent `PKToolPicker` with distinct state; a strong reference must be held by the app.
- Stroke data access in iOS 14 (`PKStroke`, `PKStrokePath`, `PKStrokePoint`) unlocks annotation, animation, recognition, and ML use cases on PencilKit drawings.

---
_Source: WWDC20 Session 10107 page (abstract, transcript, code samples, and resource links)._
