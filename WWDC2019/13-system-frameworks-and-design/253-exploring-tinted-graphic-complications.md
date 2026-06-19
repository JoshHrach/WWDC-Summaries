# Exploring Tinted Graphic Complications
**WWDC19 · Session 253** · [Watch](https://developer.apple.com/videos/play/wwdc2019/253/)

_Platforms:_ watchOS 6

## Overview
watchOS 6 introduces tinted mode for graphic complications on any watch face that supports them, including Infograph and the new watch faces shipping with watchOS 6. Tinted mode allows customers to personalize complications with their chosen color, creating a cohesive watch face appearance. Developers need to understand how ClockKit modifies gauges, text, and images in tinted contexts, and use the new `tintedImageProvider` API to supply high-quality alternate images when automatic desaturation is not desirable.

The session explains how each of the three graphic data types (gauges, text, images) behaves under tinting, distinguishes between `CLKFullColorImageProvider` and `CLKImageProvider`, and walks through four image presentation scenarios developers can choose from.

## Key Topics

**How Tinting Affects Complication Data Types**
- Gauges: rendered as a solid color (determined by system from customer's color selection) rather than a color gradient; complications that use color to convey information within a gauge should revisit their design
- Text: displayed in a single system-determined color; multicolor text providers are collapsed to one color
- Images: desaturated by default (approximately grayscale); new API allows providing an alternate template image instead

**CLKFullColorImageProvider vs. CLKImageProvider**
- `CLKFullColorImageProvider` — holds the full-color image used in full-color contexts; not a subclass of `CLKImageProvider`
- `CLKImageProvider` — manages template images (one-piece and two-piece) for classic and tinted contexts; has been available since watchOS 2

**Tinted Image Scenarios**
1. Let the full-color image be automatically desaturated (default behavior when `tintedImageProvider` is nil)
2. Supply a one-piece template image via `CLKImageProvider` as the `tintedImageProvider` — system applies tint color
3. Supply the full-color image itself inside `CLKImageProvider` — it gets templatized rather than desaturated (Apple's own complications use this)
4. Supply a two-piece template image via `CLKImageProvider` — system picks one-piece or two-piece based on context; two-piece takes priority when applicable

**Two-Piece Images**
- Composed of `twoPieceImageBackground` and `twoPieceImageForeground` (both template images)
- In tinted graphic complications the system determines foreground/background colors from the customer's selection; `tintColor` property is ignored for graphic complications (only used in classic complications)

## APIs & Frameworks

**ClockKit**
- `CLKFullColorImageProvider` (existing, watchOS 5)
  - `var image: UIImage` — full color image for full-color contexts
  - `var accessibilityLabel: String?` — label for accessibility
  - `var tintedImageProvider: CLKImageProvider?` **[NEW]** — optional image provider for tinted contexts; if nil, full color image is desaturated
  - `init(fullColorImage:tintedImageProvider:)` **[NEW]** convenience initializer
- `CLKImageProvider` (existing, watchOS 2)
  - `var onePieceImage: UIImage` — required template image
  - `var tintColor: UIColor?` — applied in classic complications only; ignored in tinted graphic complications
  - `var twoPieceImageBackground: UIImage?` — background template image for two-piece display
  - `var twoPieceImageForeground: UIImage?` — foreground template image for two-piece display
- Complication families supporting tinted mode **[NEW]**: all families with graphic complications on Infograph and new watchOS 6 watch faces
- Data providers (existing): `CLKGaugeProvider`, `CLKTextProvider`, `CLKSimpleTextProvider`, `CLKRelativeDateTextProvider`, `CLKTimeTextProvider`, `CLKDateTextProvider`

## Code Highlights

Default desaturation (no changes needed):
```swift
let provider = CLKFullColorImageProvider(fullColorImage: myFullColorImage)
// tintedImageProvider is nil → system desaturates in tinted context
```

Alternate one-piece template image in tinted context:
```swift
let tintedProvider = CLKImageProvider(onePieceImage: myTemplateImage)
let provider = CLKFullColorImageProvider(fullColorImage: myFullColorImage,
                                         tintedImageProvider: tintedProvider)
```

Templatize the full-color image (Apple's preferred approach for own complications):
```swift
let tintedProvider = CLKImageProvider(onePieceImage: myFullColorImage) // templatized, not desaturated
let provider = CLKFullColorImageProvider(fullColorImage: myFullColorImage,
                                         tintedImageProvider: tintedProvider)
```

Two-piece image for maximum control:
```swift
let tintedProvider = CLKImageProvider(onePieceImage: myOnepiece)
tintedProvider.twoPieceImageBackground = myBackground
tintedProvider.twoPieceImageForeground = myForeground
let provider = CLKFullColorImageProvider(fullColorImage: myFullColorImage,
                                         tintedImageProvider: tintedProvider)
```

## Takeaways
- Any watch face with graphic complications supports tinted mode in watchOS 6; all existing graphic complications will be automatically desaturated unless you supply a `tintedImageProvider`.
- Never rely on your chosen `tintColor` in graphic complications — tint color is always system-determined in graphic contexts.
- If your full-color image or logo desaturates poorly, supply a clean template image via `tintedImageProvider` to ensure a good appearance at any customer-selected color.
- Two-piece images give the most expressive tinted result; ClockKit automatically selects one-piece or two-piece based on the complication layout.

---
_Source: WWDC19 Session 253 page (abstract, chapter summaries, code samples, and resource links)._
