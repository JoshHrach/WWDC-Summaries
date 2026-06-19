# How to Pick a Custom Font
**WWDC17 · Session 815** · [Watch](https://developer.apple.com/videos/play/wwdc2017/815/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, tvOS 11, watchOS 4

## Overview
Choosing a custom typeface is both a functional and stylistic decision. This design-focused session from Apple's design team introduces two fundamental typographic concepts — structure (ductus) and contrast — and explains how understanding them enables more confident, principled font selection. Structure describes the path a calligraphic tool follows to create a letterform; contrast describes the variation in stroke width produced by the pressure applied to that tool. Together these two properties largely determine how a typeface reads and feels.

The session distinguishes between display type (large sizes, attention-grabbing, higher contrast and more decorative detail tolerated) and text type (comfortable reading at small sizes, lower contrast, looser spacing). Selecting the wrong category for the context — e.g., using a high-contrast display face at small body text sizes — leads to legibility problems as thin strokes disappear. Developers and designers are shown how to evaluate typeface scale and proportion to avoid needing to compensate with manual point-size adjustments when pairing faces.

Font pairing strategies are discussed: leveraging weight and italic variants within a single family for the safest consistent look; creating contrast via structural difference (e.g., geometric sans-serif headline + classical serif body); and mixing the system font San Francisco as a workhorse with a single expressive custom face. The session closes with resources for continuing typographic education.

## Key Topics
- **Structure (ductus)** — the path traced by a calligraphic tool; cursive (single stroke, dynamic) vs. constructed (multiple strokes, static)
- **Contrast** — variation in stroke width; high-contrast faces look sharp at large sizes but lose thin strokes at small sizes; low-contrast faces are more versatile at smaller sizes
- **Display vs. text type** — display: bigger sizes, higher contrast, tighter spacing, more elaborate details; text: smaller sizes, lower contrast, looser spacing, avoid extremes
- **Proportions and scale** — fonts drawn to different apparent scales within the same point size; match proportions when pairing to avoid manual size compensation
- **Stylistic impression and expectations** — how users associate typeface history and cultural context with mood (classical book faces feel literary; geometric sans-serif feels modern/simple but is ~100 years old)
- **Font pairing strategies** — single-family weight/italic pairing (safest); weight contrast only; stylistic contrast (different structure and contrast); system font + one expressive custom face; avoid more than two faces initially
- **Typographic contrast** — achieved via size, weight, or style; must be strong enough to be immediately obvious; key to hierarchy in news, books, dictionaries, and other text-heavy layouts
- **Avoiding LTypI (Lack of Typographic Imagination)** — don't pick a font solely by its name matching your app's theme; ensure style is functionally appropriate
- **Spacing** — headlines benefit from tight spacing; body text benefits from looser spacing; spacing expectations differ by usage context

## APIs & Frameworks
This session is design education focused; no new APIs are introduced. The relevant Apple developer surfaces for implementing custom fonts are:

- **`UIFont`** / **`NSFont`** — load and use custom fonts embedded in the app bundle
- **`UIFontDescriptor`** / **`NSFontDescriptor`** — query font traits (weight, design, symbolic traits) programmatically
- **`CTFont`** / **`CTFontDescriptor`** (Core Text) — lower-level font access; `CTFontCopyName` for family/style metadata
- **Info.plist `UIAppFonts` key** — register custom font files (TTF/OTF) embedded in the app bundle so `UIFont(name:size:)` can load them
- **`NSAttributedString`** — apply mixed fonts, weights, and styles within a single text view
- **Dynamic Type** — `UIFont.preferredFont(forTextStyle:)` with `UIFontMetrics` for scaling custom fonts to user accessibility preferences
- **San Francisco (system font)** — `UIFont.systemFont(of:weight:)` / `.monospacedDigitSystemFont`; use as typographic workhorse alongside one expressive custom face

## Code Highlights
No code samples were shown; this is a design principles session. For implementing custom fonts:

```swift
// Register custom font in Info.plist under UIAppFonts, then:
let customFont = UIFont(name: "MyCustomFont-Regular", size: 17)

// Scale custom font with Dynamic Type
let metrics = UIFontMetrics(forTextStyle: .body)
let scaledFont = metrics.scaledFont(for: customFont!)
label.font = scaledFont
label.adjustsFontForContentSizeCategory = true
```

## Takeaways
- Understand structure and contrast before picking any font; these two properties determine where and how legibly a typeface can be used.
- Match the category to the use: text faces for body copy (low contrast, loose spacing), display faces for headlines (high contrast, tight spacing).
- When pairing typefaces, align apparent scale to avoid manual size compensation; typographic contrast must be strong and obvious, not subtle.
- Start with one custom face paired with San Francisco; add additional typefaces only when truly necessary.

---
_Source: WWDC17 Session 815 page (abstract, chapter summaries, code samples, and resource links)._
