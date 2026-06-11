# Unwrap PaperKit
**WWDC26 · Session 372** · [Watch](https://developer.apple.com/videos/play/wwdc2026/372/)

_Platforms:_ iOS, iPadOS, macOS, visionOS (iOS/macOS/visionOS 27)

## Overview
PaperKit is the canvas framework that powers the markup experience in Notes, Preview, and Freeform. Introduced to third-party developers at WWDC25, it gains substantial new data-model APIs in iOS/macOS/visionOS 27 that let apps read, create, and modify every element on a canvas — shapes, images, links, loupes, and PencilKit strokes — as a first-class programmatic model.

This session covers three new capabilities: the `PaperMarkup.subelements` property that exposes the full ordered set of canvas elements, element-specific APIs for controlling appearance and interaction permissions, and `MarkupAdornment` — a floating overlay system for placing interactive controls anchored to canvas coordinates without persisting them as markup.

The running example builds a comic-panel layout app: it programmatically creates a grid of shape panels, styles them, locks them as read-only templates, and then adds adornments that let users tap to generate AI images for each panel via `ImagePlaygroundViewController`.

## Key Topics

### Data Model (1:22)
`PaperMarkup` now exposes `subelements: MarkupOrderedSet`, a mutable ordered collection of every canvas element. Creating markup elements is now purely programmatic:
```swift
var markup = PaperMarkup(bounds: CGRect(...))
var subelements = markup.subelements
subelements.append(ShapeMarkup(frame: panelFrame, configuration: config))
markup.subelements = subelements
paperMarkupViewController.markup = markup
```
Write back to `paperMarkupViewController.markup` to commit changes to the canvas.

`allowedInteractions` on each element controls what the user can do: `.readOnly` prevents selection, move, and resize. This is used to lock template elements like panel outlines.

### Elements (3:41)
Each element has a concrete type with typed properties:
- `ShapeMarkup` — `frame`, `strokeColor`, `fillColor`, `configuration: ShapeConfiguration`
- `ImageMarkup` — `frame`, `image: UIImage`
- `PKStroke` — Apple Pencil strokes; PaperKit builds on PencilKit, so ink strokes appear as `PKStroke` elements in the subelements ordered set
- Links and loupes are also represented as concrete element types
- `MarkupOrderedSet.updateOrAppend(_:)` — upsert an element by identity

### Adornments (5:17)
`MarkupAdornment` is a non-persistent overlay (not stored in the `PaperMarkup` model) anchored to a canvas coordinate:
```swift
MarkupAdornment(
    id: adornmentID,
    anchor: .canvas(location: center),
    imageConfiguration: .systemImage("photo.badge.plus"),
    dragRegion: .fixed,
    scalesWithZoom: false
)
```
Set `paperMarkupViewController.adornments = adornments` to display them. Taps are delivered via `PaperMarkupViewControllerDelegate.paperMarkupViewController(_:didTapAdornmentWithID:)`. Adornments track panning and zoom but are not serialized — they must be reconstructed each session.

## APIs & Frameworks

**PaperKit** — `import PaperKit`
- `PaperMarkupViewController` — the canvas view controller
- `PaperMarkup` — the canvas data model; **[NEW]** `subelements: MarkupOrderedSet`
- `PaperMarkup.bounds` — CGRect bounding the entire canvas
- `PaperMarkup.backgroundColor` — canvas background color (CGColor)
- **[NEW]** `MarkupOrderedSet` — ordered, mutable collection of markup elements
  - `append(_:)` — add an element
  - `updateOrAppend(_:)` — upsert by element identity
- `MarkupElement` protocol / base type — base for all canvas elements
- `MarkupElement.allowedInteractions` — **[NEW]** `.readOnly`, and other interaction masks
- **[NEW]** `ShapeMarkup` — shape element
  - `init(frame:configuration:)`
  - `strokeColor: CGColor`
  - `fillColor: CGColor`
  - `allowedInteractions`
- **[NEW]** `ImageMarkup` — image element
  - `init(frame:image:)` where `image: UIImage`
- **[NEW]** `MarkupAdornment` — non-persisted floating overlay
  - `init(id:anchor:imageConfiguration:dragRegion:scalesWithZoom:)`
  - `anchor: .canvas(location:)` — world-space coordinate anchor
  - `dragRegion: .fixed` — prevents user-dragging of the adornment
  - `scalesWithZoom: Bool`
- `PaperMarkupViewController.adornments: [MarkupAdornment]` — **[NEW]**
- **[NEW]** `PaperMarkupViewControllerDelegate`
  - `paperMarkupViewController(_:didTapAdornmentWithID:)` — adornment tap callback
- `ShapeConfiguration` — shape style/type configuration

**PencilKit** (underlying layer)
- `PKStroke` — pencil stroke elements appear in `subelements`
- `PKDrawing` — the PencilKit drawing model (PaperKit builds on this)

**ImagePlayground**
- `ImagePlaygroundViewController` — used in the sample to generate images placed as `ImageMarkup`
- `ImagePlaygroundViewControllerDelegate.imageViewController(_:didCreateImageAt:)`

## Code Highlights

Create a programmatic panel layout:
```swift
var markup = PaperMarkup(bounds: CGRect(origin: .zero, size: pageSize))
var subelements: MarkupOrderedSet = markup.subelements
for frame in panelFrames {
    var shape = ShapeMarkup(frame: frame, configuration: config)
    shape.allowedInteractions = .readOnly
    subelements.append(shape)
}
markup.subelements = subelements
paperMarkupViewController.markup = markup
```

Update element styles without recreating the canvas:
```swift
for element in markup.subelements {
    guard var shape = element as? ShapeMarkup else { continue }
    shape.strokeColor = selectedColor
    shape.fillColor = selectedColor.copy(alpha: 0.15)
    subelements.updateOrAppend(shape)
}
```

Handle adornment tap to launch Image Playground:
```swift
func paperMarkupViewController(_ vc: PaperMarkupViewController, didTapAdornmentWithID id: UUID) {
    let imageVC = ImagePlaygroundViewController()
    imageVC.delegate = self
    present(imageVC, animated: true)
}
```

## Takeaways
- Use `PaperMarkup.subelements` to programmatically create and modify canvas contents — it's the primary new API surface in PaperKit for iOS/macOS 27.
- Set `allowedInteractions = .readOnly` on template elements (grids, guides, backgrounds) to lock them so users can only draw over them, not accidentally move them.
- Use `MarkupAdornment` for interactive overlays (add-image buttons, annotation badges, collaboration avatars) — they are not persisted, so reconstruct them on load and after data changes.
- PaperKit builds directly on PencilKit; `PKStroke` elements appear alongside shape and image elements in `subelements`, enabling unified handling of all canvas content.

---
_Source: WWDC26 Session 372 page (abstract, chapter summaries, code samples, and resource links)._
