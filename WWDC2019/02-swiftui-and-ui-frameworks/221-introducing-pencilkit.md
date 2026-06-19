# Introducing PencilKit
**WWDC19 · Session 221** · [Watch](https://developer.apple.com/videos/play/wwdc2019/221/)

_Platforms:_ iOS 13, iPadOS 13, macOS (PKDrawing only)

## Overview
PencilKit is Apple's new framework that brings the full drawing engine from Notes, Pages, and Markup to any third-party app with just three lines of code. Previously, building a high-quality Apple Pencil drawing experience required handling estimated touches, delayed force, Metal rendering pipelines, and custom undo stacks from scratch. PencilKit encapsulates all of this into a turnkey canvas with industry-leading sub-pixel, 240 Hz latency, a rich tool palette, expressive inks, and a persistent data model.

The session is split across two presenters: Will Thimbleby covers the low-level details of Apple Pencil hardware (estimated values, delayed force, latency considerations, Pencil Tap gestures) before introducing PencilKit's architecture, and Jenny covers the `PKToolPicker` lifecycle, advanced canvas delegate callbacks, Dark Mode behavior, and the new Markup Everywhere / `UIScreenshotService` API.

The session's sample app — a small drawing app with thumbnail previews, signature pane, and Markup Everywhere adoption — demonstrates all major features.

## Key Topics

**Apple Pencil Hardware Internals**
The Pencil reports 240 touches per second. Azimuth (rotation around the iPad normal) and altitude (tilt) are calculated from a secondary touchpoint generated on the iPad surface. When that second touchpoint is obscured (edge of screen, finger overlay), azimuth/altitude are estimated and must be back-filled when correct values arrive. Force arrives over Bluetooth with a delay from the touch location stream; apps must continue listening for force updates even after the Pencil lifts, because strokes can remain in the "estimated force" region.

**Best Practices for Custom Drawing**
For lowest latency: render in Metal, use predicted touches, avoid transparent Metal layers, avoid UIVisualEffectView overlays on Metal layers, and avoid default navigation bars over drawing surfaces. For correct stroke data: listen to `actualTouchesChanged` to back-fill estimated values; use a serial queue so one stroke is updated at a time.

**PencilKit Architecture**
- `PKCanvasView` — a `UIScrollView` subclass that is the drawable region. Set its `tool` property to control the active ink. Provides `drawing: PKDrawing` for data persistence.
- `PKDrawing` — the data model; a value type. Serializes to `Data`. Available on macOS. Used to generate `UIImage` thumbnails via `PKDrawing.image(from:scale:)`.
- `PKToolPicker` — a floating UI component (not a view) that manages the tool palette. Behaves like the keyboard: its visibility is tied to first responders. Use `PKToolPicker.shared(for:)` to get the instance for a window.
- `PKTool` — base protocol; concrete types: `PKInkingTool`, `PKEraserTool`, `PKLassoTool`.

**PKInkingTool Types**
Three ink types: `.pen` (pressure-sensitive opacity and width), `.marker` (semi-transparent, wide), `.pencil` (altitude-sensitive width — wider when held flat, thinner when vertical). Each has a default width representing an average user's base; query `PKInkingTool.validWidthRange(for:)` for min/max. Set color with `UIColor`.

**PKEraserTool**
Two modes: `.bitmap` (pixel eraser — slices through stroke geometry, splits vector strokes at the cut boundary for later object erasure) and `.vector` (object eraser — removes whole strokes). This unification of bitmap and vector worlds is architecturally new in iOS 13.

**PKToolPicker Lifecycle**
1. `PKToolPicker.shared(for: window)` — get the shared instance.
2. `toolPicker.addObserver(canvasView)` — canvas auto-updates its tool when picker changes.
3. `toolPicker.setVisible(true, forFirstResponder: canvasView)` — show palette when canvas is first responder.
4. `canvasView.becomeFirstResponder()` — trigger palette display.
When a different view (e.g., a signature-only canvas) becomes first responder, the palette hides automatically. In compact size classes (iPhone), the picker is fixed-docked at the bottom; observe `toolPickerFramesObscuredDidChange` and adjust view frames or scroll insets accordingly. Undo/Redo buttons are built into the palette in regular size class only; provide your own in compact size class.

**PKCanvasView Delegate**
- `canvasViewDidBeginUsingTool(_:)` — Pencil/finger down.
- `canvasViewDidEndUsingTool(_:)` — Pencil/finger up (drawing not yet finalized).
- `canvasViewDrawingDidChange(_:)` — final force values received; drawing is complete. Update model, generate thumbnails, save here.
- `canvasViewDidFinishRendering(_:)` — all tiles loaded after `setDrawing(_:)`, scroll, or zoom.

**Dark Mode**
PencilKit strokes dynamically invert their colors to maintain legibility in Dark Mode — black ink becomes mostly-white ink over a dark background. Override with `overrideUserInterfaceStyle = .light` if drawing over static content like photos or PDFs.

**Markup Everywhere / UIScreenshotService**
New in iOS 13, the system screenshot UI (triggered by corner Pencil gesture or button combination) gains a "Full Page" segment if the app adopts `UIScreenshotService`. The app delivers PDF data representing its full content beyond the visible screen. Required for screenshot editing of scrolled content (Safari's full-page screenshot), presentations (Keynote's slide pages), and maps (Maps without chrome).

## APIs & Frameworks

**PencilKit** (iOS 13, macOS — PKDrawing only) **[NEW]**

Canvas and data model:
- `PKCanvasView: UIScrollView` **[NEW]** — drawable canvas
  - `PKCanvasView.drawing: PKDrawing` **[NEW]**
  - `PKCanvasView.tool: PKTool` **[NEW]**
  - `PKCanvasView.allowsFingerDrawing: Bool` **[NEW]**
  - `PKCanvasView.isRulerActive: Bool` **[NEW]**
  - `PKCanvasView.drawingGestureRecognizer: UIGestureRecognizer` **[NEW]**
  - `PKCanvasView.overrideUserInterfaceStyle` (inherited UIView property, used for Dark Mode opt-out)
- `PKDrawing` (value type) **[NEW]**
  - `PKDrawing.init(data:)` / `PKDrawing.dataRepresentation()` **[NEW]**
  - `PKDrawing.image(from:scale:)` **[NEW]** — generate UIImage thumbnail
- `PKCanvasViewDelegate` protocol **[NEW]**
  - `canvasViewDidBeginUsingTool(_:)` **[NEW]**
  - `canvasViewDidEndUsingTool(_:)` **[NEW]**
  - `canvasViewDrawingDidChange(_:)` **[NEW]**
  - `canvasViewDidFinishRendering(_:)` **[NEW]**

Tool Picker:
- `PKToolPicker` **[NEW]**
  - `PKToolPicker.shared(for: UIWindow) -> PKToolPicker?` **[NEW]**
  - `PKToolPicker.addObserver(_:)` **[NEW]**
  - `PKToolPicker.removeObserver(_:)` **[NEW]**
  - `PKToolPicker.setVisible(_:forFirstResponder:)` **[NEW]**
  - `PKToolPicker.frameObscured(in:) -> CGRect` **[NEW]**
  - `PKToolPickerObserver.toolPickerFramesObscuredDidChange(_:)` **[NEW]**
  - `PKToolPickerObserver.toolPickerVisibilityDidChange(_:)` **[NEW]**

Tools:
- `PKInkingTool` **[NEW]**
  - `PKInkingTool.InkType`: `.pen`, `.marker`, `.pencil` **[NEW]**
  - `PKInkingTool.init(_ inkType:color:width:)` **[NEW]**
  - `PKInkingTool.defaultWidth(for:)` **[NEW]**
  - `PKInkingTool.validWidthRange(for:)` **[NEW]**
- `PKEraserTool` **[NEW]**
  - `PKEraserTool.EraserType`: `.bitmap`, `.vector` **[NEW]**
- `PKLassoTool` **[NEW]**

Screenshot service:
- `UIScreenshotService` **[NEW]**
  - `UIWindowScene.screenshotService: UIScreenshotService?` **[NEW]**
  - `UIScreenshotServiceDelegate.screenshotService(_:generatePDFRepresentationWithCompletion:)` **[NEW]**
  - Completion block parameters: `pdfData: Data?`, `indexOfCurrentPage: Int`, `rectInCurrentPage: CGRect` (PDF coordinate space — origin bottom-left) **[NEW]**

Apple Pencil:
- `UIPencilInteraction` (existing) — double-tap gesture handler
- `UIPencilInteractionDelegate.pencilInteractionDidTap(_:)` (existing)
- `UITouch.estimatedProperties`, `UITouch.estimationUpdateIndex` (existing since iOS 9.1)
- Predicted touches: `UIEvent.predictedTouches(for:)` (existing)

## Code Highlights

Minimal three-line PencilKit setup:
```swift
let canvas = PKCanvasView(frame: view.bounds)
view.addSubview(canvas)
canvas.tool = PKInkingTool(.pen, color: .black, width: 10)
```

Generating a thumbnail in the background:
```swift
func generateThumbnail(for drawing: PKDrawing, traitCollection: UITraitCollection) -> UIImage {
    var thumbnail: UIImage!
    traitCollection.performAsCurrent {
        thumbnail = drawing.image(from: drawing.bounds, scale: UIScreen.main.scale)
    }
    return thumbnail
}
// Called from a background DispatchQueue (PKDrawing is a value type)
```

PKToolPicker setup:
```swift
guard let picker = PKToolPicker.shared(for: view.window!) else { return }
picker.addObserver(canvasView)
picker.setVisible(true, forFirstResponder: canvasView)
canvasView.becomeFirstResponder()
```

UIScreenshotService adoption:
```swift
// In UIWindowSceneDelegate
windowScene.screenshotService?.delegate = self

// Delegate method
func screenshotService(_ screenshotService: UIScreenshotService,
                       generatePDFRepresentationWithCompletion completion:
                       @escaping (Data?, Int, CGRect) -> Void) {
    let pdfData = generateFullPagePDF()
    let currentPage = currentPageIndex
    let rectInPage = currentVisibleRectInPDFCoordinates() // origin bottom-left
    completion(pdfData, currentPage, rectInPage)
}
```

## Takeaways
- PencilKit reduces a full drawing experience to three lines of code and delivers the same sub-millisecond latency engine Apple uses in Notes and Pages — don't build custom Pencil drawing from scratch unless you have requirements PencilKit cannot satisfy.
- `PKToolPicker` behaves exactly like the keyboard: visibility is first-responder driven; support compact-size-class docking by observing `toolPickerFramesObscuredDidChange` and adjusting insets or providing your own undo/redo buttons.
- `canvasViewDrawingDidChange(_:)` — not `canvasViewDidEndUsingTool` — is the correct point to save, because it fires only after delayed force values are finalized.
- Adopt `UIScreenshotService` to provide full-page PDF content for the Markup Everywhere screenshot editor, enabling users to annotate your entire content beyond the visible viewport.

---
_Source: WWDC19 Session 221 page (transcript, chapter summaries, and resource links)._
