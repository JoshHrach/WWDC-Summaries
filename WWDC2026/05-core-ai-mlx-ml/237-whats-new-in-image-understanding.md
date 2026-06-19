# What's New in Image Understanding
**WWDC26 · Session 237** · [Watch](https://developer.apple.com/videos/play/wwdc2026/237/)

_Platforms:_ iOS, iPadOS, macOS, watchOS

## Overview
This session covers four new image understanding capabilities spanning Vision and Foundation Models: a tap-to-segment API for interactive object isolation, direct image input to the on-device large language model, image-based tool calling that lets LLMs trigger Vision operations, and Vision framework support on watchOS.

The session is practical and code-forward, using a botanist assistant app as the running example. It clarifies when to use Vision (precise, deterministic image analysis — segmentation, OCR, barcodes) versus Foundation Models (open-ended semantic understanding — captions, scene descriptions, creative suggestions). The two approaches compose naturally through the tool-calling pattern.

## Key Topics

### Tap-to-Segment (Vision)
The new `GenerateIterativeSegmentationRequest` enables interactive object isolation via seed points (taps) or lasso strokes. An `ImageRequestHandler` is initialized with the image, and the request can be refined iteratively by adding included or excluded points. The segmentation model must be downloaded on-device; the API provides download management hooks. Lasso strokes should use a stroke width relative to the image's smaller dimension for consistent results.

### Image Inputs for Foundation Models
The Foundation Models framework now supports passing images directly to the on-device LLM via the `Attachment` type inside a `Prompt` builder. This enables caption generation, scene understanding, recipe suggestions from food photos, and interior design ideas. Use Foundation Models when you need open-ended semantic reasoning; use Vision for precise, structured analysis.

### Image-Based Tool Calling
Tools conforming to the `Tool` protocol can now accept image arguments via `ImageReference`. The `PlantIdentifierTool` example shows how to: define a `@Generable Arguments` struct with an `ImageReference` field, resolve the reference back to pixel data from the session's `Transcript` history, and pass that data to Vision or a custom classifier. Built-in Vision tools — including a `BarcodeReaderTool` and a saliency tool — can be added directly to a `LanguageModelSession` without any custom implementation.

### Vision on watchOS
Vision is now available on watchOS. Demonstrated using `GenerateObjectnessBasedSaliencyImageRequest` to automatically crop wildlife photos to the most prominent subject, ensuring watch UI always displays the most relevant part of an image in the compact screen.

## APIs & Frameworks

**Vision** **[Updated]**
- `ImageRequestHandler(image:)` — initializes a handler for processing image requests
- `GenerateIterativeSegmentationRequest(seed:)` — **[NEW]** tap-to-segment request; `seed` is a normalized `CGPoint`
  - `addIncludedPoint(_:)` — refine mask by adding an included point
  - `addExcludedPoint(_:)` — refine mask by adding an excluded point (implied)
  - Returns observation with `.pixelBuffer` segmentation mask
- `GenerateObjectnessBasedSaliencyImageRequest` — saliency analysis; returns prominent object rects
  - `.salientObjects` — array of `NormalizedRect` for prominent subjects
  - Now available on **watchOS** **[NEW]**
- `handler.perform(_:)` — async request execution; returns typed observation

**FoundationModels** **[Updated]**
- `Prompt` — multimodal prompt builder using result builder syntax **[NEW multimodal support]**
- `Attachment(_:)` — wraps a `CGImage`, `UIImage`, or `NSImage` for inclusion in a `Prompt` **[NEW]**
  - `.label(_:)` — assigns a label to the attachment for tool reference
- `LanguageModelSession.respond(to:)` — now accepts `Prompt` with image attachments **[NEW]**
- `LanguageModelSession.respond(generating:)` accepting a `Prompt` closure **[NEW]**
- `Tool` protocol — tool conformance for LLM tool calling
- `@Generable` — macro for structured output and tool argument types
- `ImageReference` — **[NEW]** argument type representing an image passed in a prompt
  - `imageReference.resolve(in:)` — resolve back to `Attachment` from a `Transcript`
  - `imageAttachment.pixelBuffer()` — extract `CVPixelBuffer` from resolved attachment
- `Transcript` — session message history, used to resolve image references
- `@SessionProperty(\.history)` — property wrapper to access session history inside a tool
- `BarcodeReaderTool` — **[NEW]** built-in Vision-backed tool for barcode reading by the LLM
- `LanguageModelSession(model:tools:)` — session with tool array
- `SystemLanguageModel` — the on-device Apple Foundation Model

**Related Resources**
- [Segmenting objects using taps, scribbles or rectangles](https://developer.apple.com/documentation/Vision/segmenting-objects-using-taps-scribbles-or-rectangles)
- [Implementing saliency-based image cropping in iOS and watchOS](https://developer.apple.com/documentation/Vision/implementing-saliency-based-image-cropping-in-iOS-and-watchOS)

## Code Highlights

Tap-to-segment with iterative refinement:
```swift
let handler = ImageRequestHandler(image)
let request = GenerateIterativeSegmentationRequest(seed: point)
let observation = try await handler.perform(request)
let mask = observation?.pixelBuffer
// Refine:
request.addIncludedPoint(newPoint)
let refined = try await handler.perform(request)
```

Image input to Foundation Models:
```swift
let prompt = Prompt {
    "Generate a caption for this image"
    Attachment(image)
}
let response = try await session.respond(to: prompt)
```

Image-based tool calling:
```swift
struct PlantIdentifierTool: Tool {
    @SessionProperty(\.history) var history
    @Generable struct Arguments { var image: ImageReference }
    func call(arguments: Arguments) async throws -> String {
        let transcript = Transcript(history)
        guard let attachment = arguments.image.resolve(in: transcript) else { throw AppError.imageNotFound }
        let image = try attachment.pixelBuffer()
        return classifyPlant(image)
    }
}
```

Using built-in Vision tools:
```swift
let session = LanguageModelSession(model: model, tools: [BarcodeReaderTool()])
let response = try await session.respond(generating: EventInfo.self) {
    "Get the date, location, and website from this flyer"
    Attachment(image).label("flyer")
}
```

Saliency crop on watchOS:
```swift
let request = GenerateObjectnessBasedSaliencyImageRequest()
let observation = try await request.perform(on: image)
let prominentRect = observation.salientObjects.first
```

## Takeaways
- `GenerateIterativeSegmentationRequest` enables Photoshop-style tap-to-select in any app; pair it with `addIncludedPoint` / `addExcludedPoint` for multi-tap refinement.
- Pass images directly to `LanguageModelSession` via `Attachment` inside a `Prompt` for semantic tasks (captions, descriptions); use Vision requests for precise structural tasks (OCR, barcodes, segmentation masks).
- The `BarcodeReaderTool` and saliency tool are drop-in additions to any `LanguageModelSession` — the LLM decides when to call them based on the conversation context.
- Vision is now on watchOS — saliency analysis is the recommended way to auto-crop images for compact watch displays without manual coordinate math.

---
_Source: WWDC26 Session 237 page (abstract, chapter summaries, code samples, and resource links)._
