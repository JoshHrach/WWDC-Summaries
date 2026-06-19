# Meet PaperKit
**WWDC25 · Session 285** · [Watch](https://developer.apple.com/videos/play/wwdc2025/285/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, visionOS 26

## Overview
PaperKit is a new high-level framework that brings full-featured markup and annotation authoring to any document in any app. It builds on PencilKit's drawing engine and adds structured markup types (highlights, freehand ink, shapes, typed text, signatures) along with a standardized document model (`PaperMarkup`) and ready-to-use view controllers for every Apple platform.

Previously, adding annotation capabilities required integrating PencilKit directly, building a custom toolbar, and managing serialization. PaperKit collapses all of that into two primary APIs: `PaperMarkupViewController` (iOS, iPadOS, visionOS) and `MarkupToolbarViewController` (macOS). Both are drop-in view controllers that handle tool selection, undo/redo, and the underlying canvas.

The session also introduces the Reed tool — a new calligraphy-style brush available in PencilKit and surfaced in PaperKit's tool picker — and highlights HDR ink support via `colorMaximumLinearExposure`.

## Key Topics

### PaperMarkup Document Model
`PaperMarkup` is an opaque, serializable document type that represents all annotations on a page — ink strokes, highlights, shapes, typed text, and signatures. It is platform-independent and can be embedded in any file format as raw data or encoded to a standard representation for storage.

### PaperMarkupViewController (iOS / iPadOS / visionOS)
The primary view controller for annotation on touch-based platforms. It hosts the markup canvas, the tool picker, and gesture recognizers. Pass it a `PaperMarkup` to load existing annotations and read it back when saving.

### MarkupEditViewController and MarkupToolbarViewController (macOS)
`MarkupEditViewController` is the canvas for macOS. `MarkupToolbarViewController` provides the macOS-style tool palette (similar to the markup toolbar in Preview). Both work together and can be embedded in any AppKit view hierarchy.

### FeatureSet
`FeatureSet` controls which markup tools are available. Use `FeatureSet.latest` to expose all currently available tools including Reed. Individual tools can be enabled or disabled to constrain the experience (e.g., a note-taking app might disable the signature tool).

### HDR Ink
`colorMaximumLinearExposure` on PKToolPicker enables HDR inks that exceed SDR white, visible on ProMotion XDR displays. This is particularly useful for highlight colors that need to stand out on bright backgrounds.

### Reed Tool
The Reed tool is a new calligraphy-style brush in PencilKit with variable stroke width based on Apple Pencil tilt and azimuth. It is automatically included when using `FeatureSet.latest` in PaperKit.

### UIViewControllerRepresentable Integration
PaperKit view controllers work with SwiftUI via standard `UIViewControllerRepresentable` wrappers. The session provides a reference pattern for bridging `PaperMarkupViewController` into a SwiftUI view.

## APIs & Frameworks

- **PaperKit** **[NEW]** — high-level annotation framework
  - `PaperMarkup` **[NEW]** — serializable annotation document model
  - `PaperMarkupViewController` **[NEW]** — iOS/iPadOS/visionOS annotation canvas with integrated toolbar
  - `MarkupEditViewController` **[NEW]** — macOS markup canvas
  - `MarkupToolbarViewController` **[NEW]** — macOS tool palette view controller
  - `FeatureSet` **[NEW]** — controls which tools are enabled
  - `FeatureSet.latest` **[NEW]** — all current tools including Reed
- **PencilKit**
  - `PKToolPicker` — tool picker (extended for PaperKit integration)
  - `pencilKitResponderState.activeToolPicker` **[NEW]** — read active tool picker from responder chain
  - `colorMaximumLinearExposure` **[NEW]** — HDR ink support
  - Reed tool **[NEW]** — calligraphy brush with tilt/azimuth response

## Code Highlights

```swift
// Present PaperMarkupViewController over a document view
let markupVC = PaperMarkupViewController()
markupVC.markup = existingMarkup ?? PaperMarkup()
markupVC.featureSet = .latest
present(markupVC, animated: true)

// On dismiss, save the markup
let updatedMarkup = markupVC.markup
```

```swift
// SwiftUI integration via UIViewControllerRepresentable
struct MarkupView: UIViewControllerRepresentable {
    @Binding var markup: PaperMarkup

    func makeUIViewController(context: Context) -> PaperMarkupViewController {
        let vc = PaperMarkupViewController()
        vc.featureSet = .latest
        return vc
    }

    func updateUIViewController(_ vc: PaperMarkupViewController, context: Context) {
        vc.markup = markup
    }
}
```

## Takeaways

- PaperKit is the recommended path for any app that needs annotation or markup — it handles tool UI, undo, and serialization that previously required custom code.
- Use `FeatureSet.latest` to automatically include new tools in future OS updates without code changes.
- `PaperMarkup` is the portable annotation format; store it alongside your document data and pass it back to PaperKit for subsequent editing sessions.
- The Reed tool and HDR ink are only visible on ProMotion XDR hardware; graceful fallback is automatic.

---
_Source: WWDC25 Session 285 page (abstract, chapter summaries, code samples, and resource links)._
