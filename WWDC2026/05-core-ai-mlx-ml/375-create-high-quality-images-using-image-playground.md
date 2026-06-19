# Create High-Quality Images Using Image Playground
**WWDC26 · Session 375** · [Watch](https://developer.apple.com/videos/play/wwdc2026/375/)

_Platforms:_ iOS, iPadOS, macOS (Apple Intelligence required)

## Overview
Image Playground now supports a new generative model running on Private Cloud Compute, enabling photorealistic image generation in virtually any style — a significant upgrade from the illustrated styles available in previous versions. Developers can integrate this capability into their apps using the `ImagePlayground` framework with a sheet-based UI that handles all generation and editing interactions.

This session walks through adopting Image Playground in a greeting card app, covering the full API surface: presenting the sheet with seed concepts from app content, providing a reference photo, using PencilKit drawings as visual suggestions, specifying image dimensions for non-square use cases, controlling style presets including a new external provider style, generating Adaptive Image Glyphs for compact icon use, disabling personalization, and handling availability gracefully on non-supported devices.

## Key Topics

### Capabilities
The new model supports photorealistic generation alongside illustrated styles. Users can generate images with people, specify styles, use custom aspect ratios, and modify images through natural language and touch. The model runs on Private Cloud Compute, not on-device.

### Adopting Image Playground
Use the `.imagePlaygroundSheet(isPresented:concepts:onCompletion:)` SwiftUI modifier — or `ImagePlaygroundViewController` for UIKit/AppKit — to present the generation UI. The `onCompletion` callback returns a `URL` to the generated image file.

### Seeding with Concepts
Pre-populate the sheet with context using `ImagePlaygroundConcept` values:
- `.text(_:)` — a literal text hint
- `.extracted(from:title:)` — extract relevant concepts from a text string (e.g., card message)
- `.drawing(_:)` — a `PKDrawing` used as a visual suggestion (PencilKit integration **[NEW]**)

### Source Image
Pass a `sourceImage: Image?` to provide a reference photo the user can transform or use as a starting point.

### Options
`ImagePlaygroundOptions` controls generation behavior:
- `.sizeSpecification = .closest(to:)` — request the output size closest to a specified `CGSize`
- `.personalization = .disabled` — opt out of personalization features

### Styles
Use `.imagePlaygroundGenerationStyle(_:in:)` to control which styles are available and set a default:
- `.externalProvider` — **[NEW]** style sourced from a third-party provider
- `.emoji` — emoji/genmoji style for compact icon generation
- Pass an array for `allowedStyles` to restrict or expand the style picker

### Adaptive Image Glyphs
Use the `onAdaptiveImageGlyphCreation` callback alongside `.imagePlaygroundGenerationStyle(.emoji, in: [.emoji])` to receive an `NSAdaptiveImageGlyph` suitable for compact icon use (e.g., card thumbnails, app icons).

### Availability
Use the `@Environment(\.supportsImageGeneration)` environment value to check if the device supports Image Playground, and adapt your UI accordingly — showing a full editor for supported devices and a simpler picker for unsupported ones.

## APIs & Frameworks

**ImagePlayground** **[Updated]**
- `.imagePlaygroundSheet(isPresented:concepts:sourceImage:onCompletion:onCancellation:)` — SwiftUI view modifier **[Updated: new parameters]**
  - `onAdaptiveImageGlyphCreation: (NSAdaptiveImageGlyph) -> Void` — **[NEW]** callback for glyph generation
- `.imagePlaygroundOptions(_:)` — SwiftUI modifier to configure generation options **[NEW]**
- `.imagePlaygroundGenerationStyle(_:in:)` — SwiftUI modifier to set default style and allowed styles **[NEW]**
- `ImagePlaygroundViewController` — UIKit/AppKit view controller
  - `.concepts` — array of `ImagePlaygroundConcept`
  - `.delegate` — `ImagePlaygroundViewControllerDelegate`
  - `imagePlaygroundViewController(_:didCreateImageAt:)` — delegate callback with generated image URL
- `ImagePlaygroundConcept` — seed concept enum **[Updated]**
  - `.text(_:)` — literal text hint
  - `.extracted(from:title:)` — extract concepts from a string
  - `.drawing(_:)` — **[NEW]** `PKDrawing` as visual suggestion
- `ImagePlaygroundOptions` — **[NEW]** generation options struct
  - `.sizeSpecification` — `ImagePlaygroundSizeSpecification`
    - `.closest(to:)` — nearest supported size to a `CGSize`
  - `.personalization` — `.disabled` to opt out of personalization
- `ImagePlaygroundGenerationStyle` — style enum
  - `.externalProvider` — **[NEW]** third-party provider style
  - `.emoji` — emoji/glyph style
  - `.animation` — animated style (existing)
  - `.illustration` — illustrated style (existing)
  - `.realistic` — **[NEW]** photorealistic style
- `@Environment(\.supportsImageGeneration)` — **[NEW]** environment value indicating device support
- `NSAdaptiveImageGlyph` — compact adaptive icon type (existing, new generation path)

**PencilKit**
- `PKDrawing` — drawing input for `.drawing(_:)` concept
- `PKDrawing.strokes` — check if drawing has content

**Related Resources**
- [ImagePlayground documentation](https://developer.apple.com/documentation/ImagePlayground) (implied)
- Related session: "Build with the new Apple Foundation Model on Private Cloud Compute" (Session 319)
- Related session: "Read between the strokes with PencilKit" (Session 203)

## Code Highlights

Adopt with concepts and source image (SwiftUI):
```swift
.imagePlaygroundSheet(
    isPresented: $showingPlayground,
    concepts: [
        .text(card.theme),
        .extracted(from: card.message, title: card.theme),
        .drawing(drawing)   // PencilKit drawing as visual suggestion
    ],
    sourceImage: sourceImage,
    onCompletion: { url in store.saveImage(url, for: &card) }
)
.imagePlaygroundOptions(options)
.imagePlaygroundGenerationStyle(
    pendingStylePreset.defaultStyle,
    in: pendingStylePreset.allowedStyles + [.externalProvider]
)
```

Generate an Adaptive Image Glyph for thumbnail:
```swift
Color.clear
    .imagePlaygroundSheet(
        isPresented: $showingIconPlayground,
        concepts: concepts,
        onCompletion: { _ in },
        onAdaptiveImageGlyphCreation: { glyph in
            store.saveIcon(glyph, for: &card)
        }
    )
    .imagePlaygroundGenerationStyle(.emoji, in: [.emoji])
```

Check availability:
```swift
@Environment(\.supportsImageGeneration) private var supportsImageGeneration
// Use to show/hide Image Playground UI
```

## Takeaways
- The new Private Cloud Compute model enables photorealistic image generation — update any app that previously limited users to illustrated styles only.
- Use `.extracted(from:)` to automatically derive concepts from existing text content in your app (messages, descriptions, titles) rather than requiring users to type a prompt from scratch.
- The `onAdaptiveImageGlyphCreation` callback plus `.emoji` style is the correct path for generating compact icons/thumbnails — it produces `NSAdaptiveImageGlyph` which renders correctly across all sizes and contexts.
- Always check `\.supportsImageGeneration` and provide a graceful fallback; the feature requires Apple Intelligence support which is not available on all devices.

---
_Source: WWDC26 Session 375 page (abstract, chapter summaries, code samples, and resource links)._
