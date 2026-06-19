# What's New in PDFKit
**WWDC22 · Session 10089** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10089/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, Mac Catalyst

## Overview
PDFKit is Apple's framework for viewing, editing, and saving PDF documents, available on iOS, macOS, and Mac Catalyst. iOS 16 and macOS Ventura bring four significant new capabilities: Live Text support for scanned PDFs (on-demand OCR as pages are viewed), improved form recognition, a new image-to-PDF-page API, and the long-requested overlay view system enabling PencilKit and other interactive content on top of PDF pages.

The new `PDFPageOverlayViewProvider` protocol is the highlight feature — it enables any UIView/NSView to be overlaid on PDF pages with automatic lifecycle management. PDFKit handles view creation timing intelligently to avoid creating hundreds of views for large documents. Combined with an updated annotation save mechanism and new write options, round-trip editing of overlay content is fully supported.

## Key Topics

### Live Text in PDFs
PDFKit now performs on-demand OCR on scanned PDF pages (pages that contain only bitmap images, no text). Users can select and search text in scanned PDFs. OCR runs in-place without copying the document, on demand as pages are interacted with. An option to save extracted text for the whole document is available when saving.

### Improved Form Handling
PDF form fields are now automatically recognized even in documents without built-in text field annotations. Users can tab through fields and enter text as expected.

### PDF Pages from Images
New API `PDFPage(image:options:)` creates a PDF page from a `CGImageRef`. Options include:
- `mediaBox` — page size (fit image exactly or use paper sizes like Letter)
- `rotation` — portrait or landscape
- `upscaleIfSmaller` — whether small images are upscaled to fill the page

Images are compressed using high-quality JPEG encoding. No intermediate format conversions needed.

### Overlay Views (PDFPageOverlayViewProvider)
New protocol enabling interactive UIView/NSView overlays on PDF pages. PDFKit calls the provider when it needs to display or reclaim views (handling the large-document lifecycle automatically). Key protocol methods:
- `pdfView(_:overlayViewFor:)` — return the view for a given page
- `pdfView(_:willDisplayOverlayView:for:)` — optional; install gesture recognizers
- `pdfView(_:willEndDisplayingOverlayView:for:)` — optional; save view data back to the page model

### Saving Overlay Content as Annotations
Overlay content is saved by creating a `PDFAnnotation` subclass that overrides `draw(with:in:)` to render the overlay (e.g., PencilKit drawing) into a PDF appearance stream. Custom data (e.g., PKDrawing bytes) is stored in the annotation as a private key/value pair for round-trip editing. Use `PDFDocumentWriteOption.burnInAnnotationsOption` to flatten annotations into the page.

### New PDFDocumentWriteOptions
- `burnInAnnotationsOption` **[NEW]** — flatten annotations into page content on save
- `saveAllImagesAsJPEGOption` **[NEW]** — encode all images as JPEG
- `optimizeImagesForScreenOption` **[NEW]** — downsample images to HiDPI screen resolution
- `createLinearizedPDFOption` **[NEW]** — create linearized PDF optimized for web streaming

## APIs & Frameworks

**PDFKit**
- `PDFPage(image:options:)` **[NEW]** — create PDF page from `CGImageRef`
- `PDFPageImageInitializationOption` **[NEW]** — option keys for image-to-page API
  - `.mediaBox` — specify page size
  - `.rotation` — page rotation
  - `.upscaleIfSmaller` — upscale small images
- `PDFPageOverlayViewProvider` protocol **[NEW]** — protocol for overlay view lifecycle
  - `pdfView(_:overlayViewFor:) -> PDFKitPlatformView?` **[NEW]**
  - `pdfView(_:willDisplayOverlayView:for:)` **[NEW]** — optional
  - `pdfView(_:willEndDisplayingOverlayView:for:)` **[NEW]** — optional
- `PDFView.pageOverlayViewProvider` **[NEW]** — assign an overlay view provider
- `PDFAnnotation` — existing class; override `draw(with:in:)` for custom appearance stream
- `PDFAnnotation.setValue(_:forAnnotationKey:)` — store custom data in annotation
- `PDFAnnotationKey` — annotation dictionary key
- `PDFDocumentWriteOption.burnInAnnotationsOption` **[NEW]** — flatten annotations
- `PDFDocumentWriteOption.saveAllImagesAsJPEGOption` **[NEW]** — JPEG image encoding
- `PDFDocumentWriteOption.optimizeImagesForScreenOption` **[NEW]** — downsample images
- `PDFDocumentWriteOption.createLinearizedPDFOption` **[NEW]** — linearized PDF output
- `PDFDocument.dataRepresentation(options:)` — save with write options
- Live Text support in `PDFView` **[NEW]** — automatic on-demand OCR for scanned pages
- Form field recognition **[NEW improved]** — auto-recognize form fields

## Code Highlights

```swift
// Install PencilKit overlay views on PDF pages
class Coordinator: NSObject, PDFPageOverlayViewProvider {
    var pageToViewMapping = [PDFPage: UIView]()
    
    func pdfView(_ view: PDFView, overlayViewFor page: PDFPage) -> UIView? {
        if let existing = pageToViewMapping[page] { return existing }
        let canvasView = PKCanvasView(frame: .zero)
        canvasView.backgroundColor = .clear
        pageToViewMapping[page] = canvasView
        if let drawing = (page as? MyPDFPage)?.drawing {
            canvasView.drawing = drawing
        }
        return canvasView
    }
    
    func pdfView(_ pdfView: PDFView, willEndDisplayingOverlayView overlayView: UIView,
                 for page: PDFPage) {
        (page as? MyPDFPage)?.drawing = (overlayView as? PKCanvasView)?.drawing
        pageToViewMapping.removeValue(forKey: page)
    }
}

// Save with burn-in and image optimization options
let options: [PDFDocumentWriteOption: Any] = [
    .burnInAnnotationsOption: true,
    .saveAllImagesAsJPEGOption: true,
    .optimizeImagesForScreenOption: true
]
let data = pdfDocument.dataRepresentation(options: options)
```

## Takeaways

- Live Text in PDFKit provides on-demand OCR for scanned PDFs — no document copying, no full-document preprocessing, just works as users interact with pages.
- `PDFPageOverlayViewProvider` is the correct way to add PencilKit or any interactive view on top of PDF pages; PDFKit handles all view lifecycle timing for large documents automatically.
- Save overlay content as a `PDFAnnotation` subclass with a custom appearance stream; use private annotation key/value pairs to store raw data for round-trip editing.
- New write options (`burnInAnnotationsOption`, `saveAllImagesAsJPEGOption`, `optimizeImagesForScreenOption`, `createLinearizedPDFOption`) give fine-grained control over PDF output size and fidelity.

---
_Source: WWDC22 Session 10089 page (abstract, chapter summaries, code samples, and resource links)._
