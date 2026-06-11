# Read Between the Strokes with PencilKit
**WWDC26 · Session 203** · [Watch](https://developer.apple.com/videos/play/wwdc2026/203/)

_Platforms:_ iOS, iPadOS (iOS 27)

## Overview
This session unlocks handwriting recognition in third-party apps using the same on-device recognition engine that powers Notes and Freeform. The new `PKStrokeRecognizer` API converts a `PKDrawing` into recognized text, indexable content, and search results — entirely on-device, with no network calls. Alongside recognition, the session covers path conversion (bidirectional Bézier ↔ `PKStrokePath`), improved drawing model access (stable stroke identities, controllable selection, wet-ink render groups), and two stroke-slicing techniques for erasing and animation.

## Key Topics

### Handwriting Recognition (3:25)
The new `PKStrokeRecognizer` class is the public surface for on-device handwriting recognition, supporting a wide range of alphabets and languages (the same set used by Notes):
- `recognizedText()` — returns a plain `String` of recognized handwriting
- `indexableContent` — returns a structured representation suitable for Spotlight indexing
- `search(_:)` — returns an array of `PKStrokeRecognizer.SearchResult` with `bounds` for each match, enabling highlight-on-canvas search

Usage pattern: call `await recognizer.updateDrawing(drawing)` first, then call any recognition method. All recognition is async and on-device.

### Path Conversion (8:38)
New bidirectional conversion between `PKStrokePath` and `UIBezierPath`/`CGPath`:
- `PKStrokePath.init(bezierPath:)` — converts a Bézier path to a `PKStrokePath`
- `PKStrokePath.bezierPath` (or equivalent) — converts a `PKStrokePath` to a `UIBezierPath` without loss of fidelity

This enables apps that store strokes as Bézier paths (CAShapeLayer, custom renderers) to construct `PKDrawing` objects and run handwriting recognition on any canvas, not just `PKCanvasView`.

### Improved Model Access (10:21)
Three improvements to the `PKDrawing` / `PKStroke` model:
- **Stable stroke IDs** — `PKStroke` now conforms to `Identifiable` with a stable `id` that survives edits and undo operations, enabling reliable stroke tracking and diffing
- **Controllable canvas selection** — `PKCanvasView` exposes a selection change delegate so apps can observe and programmatically control which strokes are selected
- **Adjustable wet-ink render groups** — apps can configure the render group used for in-progress (wet-ink) strokes, useful for custom blending or rendering pipelines

### Stroke Slicing (11:25)
Two ways to slice a stroke into parts:
1. **Programmatic erasing** — cuts a `PKStroke` into independent `PKStroke` segments along an erase path, useful for programmatic erase tools
2. **Substroke extraction** — extract a sub-range using parametric `ClosedRange<PKStrokePoint.ParametricValue>`, useful for progress animations or partial-stroke effects

Performance consideration: slicing is O(n) in stroke complexity; avoid slicing thousands of strokes per frame in complex drawings.

## APIs & Frameworks

**PencilKit** — `import PencilKit`

_Handwriting Recognition_
- **[NEW]** `PKStrokeRecognizer` — on-device handwriting recognizer
  - `updateDrawing(_ drawing: PKDrawing) async`
  - `recognizedText() async -> String`
  - `indexableContent: PKStrokeRecognizer.IndexableContent? async` — Spotlight-indexable representation
  - `search(_ query: String) async -> [PKStrokeRecognizer.SearchResult]`
  - `PKStrokeRecognizer.SearchResult.bounds: CGRect`

_Path Conversion_
- **[NEW]** `PKStrokePath.init(bezierPath: UIBezierPath)` — Bézier → PKStrokePath
- **[NEW]** `PKStrokePath.bezierPath: UIBezierPath` — PKStrokePath → Bézier

_Drawing Model_
- **[NEW]** `PKStroke: Identifiable` — stable `id` across edits and undo
- **[NEW]** `PKCanvasView` selection change delegate — observe/control selection state
- **[NEW]** Wet-ink render group configuration on `PKCanvasView`

_Stroke Slicing_
- **[NEW]** Programmatic erasing API — cuts strokes along an erase path
- **[NEW]** Substroke extraction with parametric range (`ClosedRange<PKStrokePoint.ParametricValue>`)
- `PKStrokePath` — underlying stroke geometry
- `PKStrokePoint` — individual point with pressure, azimuth, altitude, etc.

_Existing Core Types_
- `PKDrawing` — the drawing model (collection of `PKStroke`)
- `PKStroke` — a single continuous ink stroke
- `PKCanvasView` — the ink canvas view
- `PKInkingTool`, `PKEraserTool`, `PKLassoTool`

## Code Highlights

Recognize handwriting in a drawing:
```swift
let recognizer = PKStrokeRecognizer()
await recognizer.updateDrawing(drawing)
myLabel.text = await recognizer.recognizedText()
```

Index drawing content for Spotlight:
```swift
await recognizer.updateDrawing(drawing)
if let indexedContent = await recognizer.indexableContent {
    index(text: indexedContent)
}
```

Search within a drawing:
```swift
let results = await recognizer.search("apple")
for result in results {
    highlight(bounds: result.bounds)
}
```

## Takeaways
- Adopt `PKStrokeRecognizer` to add multilingual handwriting recognition, search, and accessibility support without any server-side infrastructure — it is fully on-device.
- Use `PKStrokePath.init(bezierPath:)` to bring existing canvas drawings (stored as Bézier paths) into the PencilKit model, enabling recognition on canvases that don't use `PKCanvasView`.
- Track strokes by their stable `Identifiable.id` rather than by array index — the ID survives undo, redo, and remote sync operations.
- Use substroke extraction sparingly in complex drawings; profile with Instruments to ensure slicing stays well within frame budget for animation use cases.

---
_Source: WWDC26 Session 203 page (abstract, chapter summaries, code samples, and resource links)._
