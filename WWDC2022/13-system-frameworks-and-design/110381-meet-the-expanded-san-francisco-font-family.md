# Meet the expanded San Francisco font family
**WWDC22 · Session 110381** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110381/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
The San Francisco (SF) font family received its most significant expansion to date with three new width styles added to SF Pro: Condensed, Compressed, and Expanded. These join the existing Regular width and, combined with all available weights (Ultralight through Black), dramatically increase the typographic palette available to designers building Apple-platform interfaces.

The session also covers the linguistic expansion of SF, highlighting SF Arabic — introduced the prior year — and announcing SF Arabic Rounded as a new addition to the family. SF Arabic brings all the advanced font technologies of SF Pro (full weight range, optical sizes) to the Arabic writing system, supporting Arabic, Persian, Pashto, Sindhi, and many more languages written in the Arabic script.

The presentation uses real-world examples from Photos, News, and Maps to illustrate how width styles create typographic hierarchy, improve space efficiency, and deliver expressive headlines, while maintaining consistent vertical proportions for predictable font mixing.

## Key Topics

### San Francisco Family Overview
- **SF Pro / SF Pro Rounded** — primary system fonts for iOS, iPadOS, macOS, tvOS
- **SF Compact / SF Compact Rounded** — optimized for narrow columns and small point sizes; default for watchOS
- **SF Mono** — monospaced variant for coding environments (Xcode, Swift Playgrounds)
- **New York** — the serif companion system font
- All families share common weight ranges (Ultralight to Black) and optical size technology

### New SF Pro Width Styles
- **Condensed** — narrower than Regular, still comfortable for body text; space-efficient for headlines
- **Compressed** — most compact style; flat-sided shapes; highly graphical; best for Display sizes
- **Expanded** — wide, open proportions; works for headlines, labels, and cartographic text
- Vertical proportions remain identical across all width styles → safe to mix widths without scale discrepancies
- Only horizontal glyph proportions change; stem thickness is relatively consistent while counter (negative) space varies dramatically

### Design Usage Guidance
- Photos: contrasting Compressed + Expanded widths in Memory Titles for visual impact
- News: Condensed and Compressed for space-efficient multi-line headlines; Expanded for bylines/captions
- Maps: Expanded with loose tracking for large geographical labels; full width system for cartographic hierarchy
- Styles near center of Weight × Width map = more neutral; perimeter styles = more expressive
- Recommend no more than two or three styles in a single hierarchy; always consider legibility at small sizes

### SF Arabic and SF Arabic Rounded
- **SF Arabic** — introduced in 2021; contemporary Naskh style; rational and flexible design
  - Full weight range (Ultralight to Black), nine weights
  - Optical sizes tailored for Arabic script: Text (higher stroke contrast, wider spacing) and Display (simpler geometry)
  - Supports Arabic, Persian, Pashto, Sindhi, and many other Arabic-script languages
  - Extensive character set: vocalization marks, tone marks, Quranic annotations, honorifics, extended vowel signs
- **SF Arabic Rounded** — **[NEW]** 2022 addition; same full weight range and optical sizes as SF Arabic; used in Reminders and Fitness app

### Optical Sizes and Variable Font Technology
- Fonts automatically adjust design features based on point size: dot spacing on "i", aperture openness, tracking
- Display vs. Text optical size variants
- Variable font axes: Weight and Optical Size (covered in depth in the WWDC20 "Details of UI Typography" session)

## APIs & Frameworks

**UIKit / AppKit / SwiftUI (Typography)**
- `UIFont` / `NSFont` — system font access
- `Font` (SwiftUI) — system font access
- `UIFontDescriptor.SymbolicTraits` — font width traits
- Font width attributes via `UIFontDescriptor` / `NSFontDescriptor`:
  - `UIFontDescriptorTraitCondensed` — condensed width trait
  - `UIFontDescriptorTraitExpanded` — expanded width trait
- Dynamic Type text styles — continue to work across all SF Pro widths
- `UIFontMetrics` — scaling custom fonts with Dynamic Type

**Font Assets (available for download at developer.apple.com)**
- SF Pro (Regular, Condensed, Compressed, Expanded) — **[NEW widths]**
- SF Pro Rounded
- SF Compact / SF Compact Rounded
- SF Mono
- New York (serif)
- SF Arabic — (introduced 2021)
- SF Arabic Rounded — **[NEW]**

## Code Highlights

No code samples in this session (design/typography focus). To apply a condensed or expanded system font variant programmatically:

```swift
// UIKit — request a condensed system font
let descriptor = UIFontDescriptor.preferredFontDescriptor(withTextStyle: .headline)
    .withSymbolicTraits(.traitCondensed)!
let condensedFont = UIFont(descriptor: descriptor, size: 0)

// SwiftUI — use width variant via Font.width (if available in target OS)
```

## Takeaways
- SF Pro now covers four width axes — Regular, Condensed, Compressed, and Expanded — each in nine weights, providing an unprecedented level of typographic control for Apple-platform UI design.
- Width styles share vertical proportions, making font mixing predictable; they are safe to pair with any weight and with other SF families (Rounded, Mono, New York).
- For space-constrained layouts, Condensed and Compressed let text be set much larger within the same footprint; Expanded works well for cartographic labels, bylines, and Display headlines.
- SF Arabic Rounded is a new 2022 addition to the SF family, bringing rounded Arabic system typography with full weights and optical sizes to complement SF Arabic.

---
_Source: WWDC22 Session 110381 page (abstract, chapter summaries, and resource links)._
