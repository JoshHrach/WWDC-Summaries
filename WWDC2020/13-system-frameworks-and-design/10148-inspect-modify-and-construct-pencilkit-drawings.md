# Inspect, Modify, and Construct PencilKit Drawings
**WWDC20 · Session 10148** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10148/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
iOS 14 opens the PencilKit data model to developers, enabling inspection, modification, and programmatic construction of drawings. This session walks through every layer of the model: `PKDrawing` (a collection of strokes), `PKStroke` (path + ink + transform + mask), `PKStrokePath` (a cubic B-spline of control points), and `PKStrokePoint` (appearance and touch data at a location).

The session demonstrates a handwriting practice app that synthesizes PencilKit drawings from text, animates a marker dot along stroke paths using parametric interpolation, and scores the user's attempt against a stored template — illustrating the full range of capabilities unlocked by data model access: inspection, recognition, animation, and procedural construction.

## Key Topics

**PKDrawing**
- A drawing is an ordered array of `PKStroke` objects — ordered by the sequence the user drew them
- `drawing.strokes: [PKStroke]` — access all strokes
- New drawing from a slice of strokes: `PKDrawing(strokes: someSlice)` **[NEW]**
- `PKDrawing.image(from:scale:)` — render to `UIImage` for image-based recognition

**PKStroke**
- Primary components: `path: PKStrokePath`, `ink: PKInk`, `transform: CGAffineTransform`, `mask: UIBezierPath?`
- `renderBounds: CGRect` — bounding box of the fully rendered stroke (accounts for path, ink, transform, mask)
- `maskedPathRanges: [ClosedRange<CGFloat>]` — parametric ranges of the path that survive the mask; used for recognition and animation of partially erased strokes

**PKInk**
- `inkType: PKInk.InkType` — `.pen`, `.pencil`, `.marker`
- `color: UIColor` — stroke color
- Width is not stored on ink — it varies along the path via `PKStrokePoint.size`

**PKStrokePath**
- A uniform cubic B-spline; stored as control points (not points on the curve)
- `count: Int` — number of control points
- Subscript `path[i]` — returns control point `i` as `PKStrokePoint` (not on the curve)
- `interpolatedPoints(strideBy:)` — returns a sequence of `PKStrokePoint` values on the actual curve
  - `.distance(CGFloat)` — uniform spacing by distance in drawing coordinate points
  - `.time(TimeInterval)` — uniform spacing by time in seconds
  - `.parametricValue(CGFloat, offsetBy:)` — non-uniform stepping; useful for time-based animation
- `interpolatedPoint(at parametricValue: CGFloat)` — single point at given parametric value
- `parametricValue(_ parametricValue: CGFloat, offsetBy: PKStrokePath.InterpolatedSlice.Stride) -> CGFloat` — advance a parametric value by a distance or time amount

**PKStrokePoint**
- Appearance attributes: `location: CGPoint`, `size: CGSize`, `azimuth: CGFloat` (rotation), `opacity: CGFloat`
- Touch attributes: `force: CGFloat`, `altitude: CGFloat` — same as `UITouch` values when drawn
- `timeOffset: TimeInterval` — seconds since the stroke path's creation date
- Stored in lossy compressed format — set values will not round-trip with perfect precision

**Parametric Values**
- Integer parametric values (0, 1, 2 … count-1) produce points equivalent to the control points but ON the curve
- Non-integer values (e.g., 2.4) interpolate between control points — full floating point flexibility
- Use `parametricValue(_:offsetBy:)` to advance position by exact time elapsed between animation frames — handles non-uniform frame timing correctly

**Masked Strokes**
- Masks created when pixel eraser partially erases a stroke
- `maskedPathRanges` is an array of `ClosedRange<CGFloat>` parametric ranges representing the surviving portions
- May have zero ranges (entire path is erased but stroke object still exists) or multiple ranges (stroke has holes)
- When iterating a masked stroke for recognition or animation, always use `maskedPathRanges` — iterating the full path will include erased portions

**Splitting and Constructing Drawings**
- Split a drawing by slicing `drawing.strokes` array into per-letter segments
- Construct a new `PKDrawing` by initializing with a `[PKStroke]` array
- Compose multi-letter drawings by concatenating slices and constructing new drawings
- Procedurally create strokes with custom `PKStrokePath`, `PKInk`, and `CGAffineTransform`

**Recognition Patterns**
- Spline-based recognition: iterate `maskedPathRanges`, interpolate points, use `timeOffset`/`force`/`location` for comparison
- Image-based recognition: call `PKDrawing.image(from:scale:)` then pass result to Vision or other image classifiers

## APIs & Frameworks

### PencilKit — Data Model (New in iOS 14)
- `PKDrawing.strokes: [PKStroke]` **[NEW]** — ordered array of all strokes in the drawing
- `PKDrawing.init(strokes: [PKStroke])` **[NEW]** — construct a drawing from strokes
- `PKDrawing.image(from: CGRect, scale: CGFloat) -> UIImage` — render drawing region to image

- `PKStroke` **[NEW]** — value type representing a single stroke
  - `.path: PKStrokePath` — the stroke's B-spline path
  - `.ink: PKInk` — color and type
  - `.transform: CGAffineTransform` — orientation and position
  - `.mask: UIBezierPath?` — clipping mask (nil for unmasked strokes)
  - `.renderBounds: CGRect` — full rendered bounding box
  - `.maskedPathRanges: [ClosedRange<CGFloat>]` **[NEW]** — parametric ranges surviving the mask

- `PKInk` **[NEW]** — value type describing stroke appearance
  - `.inkType: PKInk.InkType` — `.pen`, `.pencil`, `.marker`
  - `.color: UIColor`

- `PKStrokePath` **[NEW]** — cubic B-spline path
  - `count: Int` — control point count
  - `subscript(Int) -> PKStrokePoint` — control point at index
  - `interpolatedPoints(strideBy:) -> PKStrokePath.InterpolatedSlice` — on-curve point sequence
  - `interpolatedPoint(at: CGFloat) -> PKStrokePoint` — single interpolated point
  - `parametricValue(_ parametricValue: CGFloat, offsetBy: PKStrokePath.InterpolatedSlice.Stride) -> CGFloat` — advance parametric value

- `PKStrokePath.InterpolatedSlice.Stride` — enum cases:
  - `.distance(CGFloat)` — distance in drawing coordinate points
  - `.time(TimeInterval)` — time in seconds
  - `.parametricValue(CGFloat)` — raw parametric step

- `PKStrokePoint` **[NEW]** — atomic drawing data point
  - `.location: CGPoint`
  - `.size: CGSize`
  - `.azimuth: CGFloat`
  - `.opacity: CGFloat`
  - `.force: CGFloat`
  - `.altitude: CGFloat`
  - `.timeOffset: TimeInterval`

## Code Highlights

Splitting a drawing into per-letter drawings:
```swift
let alphabetDrawing: PKDrawing  // master drawing with all letters
let strokesPerLetter = alphabetDrawing.strokes.count / 26

var letterDrawings: [PKDrawing] = []
for i in 0..<26 {
    let start = i * strokesPerLetter
    let end = start + strokesPerLetter
    let letterStrokes = Array(alphabetDrawing.strokes[start..<end])
    letterDrawings.append(PKDrawing(strokes: letterStrokes))
}
```

Iterating on-curve points at uniform distance:
```swift
for point in stroke.path.interpolatedPoints(strideBy: .distance(50)) {
    // point.location is ON the curve, spaced ~50 pts apart
    draw(point.location)
}
```

Iterating a masked stroke using maskedPathRanges:
```swift
for range in stroke.maskedPathRanges {
    let slice = stroke.path.interpolatedPoints(in: range, strideBy: .distance(10))
    for point in slice {
        draw(point.location)
    }
}
```

Animating a marker along a stroke by elapsed time:
```swift
var currentParametricValue: CGFloat = 0

func updateAnimation(deltaTime: TimeInterval) {
    currentParametricValue = stroke.path.parametricValue(
        currentParametricValue,
        offsetBy: .time(deltaTime)
    )
    let point = stroke.path.interpolatedPoint(at: currentParametricValue)
    markerView.center = point.location
}
```

## Takeaways
- Always use `maskedPathRanges` when iterating stroke paths for recognition or animation — iterating the full path includes erased regions that are invisible to the user.
- The subscript `path[i]` returns B-spline control points which are NOT on the curve; use `interpolatedPoints(strideBy:)` or `interpolatedPoint(at:)` for points that lie on the actual rendered stroke.
- `parametricValue(_:offsetBy:)` with `.time(deltaTime)` is the correct way to animate along a stroke at the user's original drawing velocity, handling non-uniform frame timing naturally.
- Procedural drawing construction — building `PKStroke` objects from `PKStrokePath`, `PKInk`, and `CGAffineTransform` and combining them into a `PKDrawing` — enables server-generated templates, synthetic ink effects, and guided drawing experiences.

---
_Source: WWDC20 Session 10148 page (abstract, transcript, and resource links)._
