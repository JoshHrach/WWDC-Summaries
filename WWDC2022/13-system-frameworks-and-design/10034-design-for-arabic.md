# Design for Arabic
**WWDC22 · Session 10034** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10034/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
This session guides designers and developers through best practices for creating or optimizing apps and games for Arabic-speaking audiences. With roughly 660 million people using Arabic script across more than 22 countries, reaching even a fraction of this audience requires attention not only to language translation but also to UI directionality, typography, iconography, and numerals.

Arabic is written right-to-left, which means layouts, navigation flows, carousels, and entire app mental models must be mirrored. Apple's native frameworks (SwiftUI) handle the bulk of this automatically, but designers need to actively manage specific components such as charts, toggles, segmented controls, and pagination indicators.

The session also introduces the new SF Arabic Rounded typeface for WWDC22, and covers how SF Symbols provides over 300 Arabic and right-to-left localized variants to ensure cultural relevance.

## Key Topics

### UI Directionality
- All layout elements (titles, paragraphs, carousels, navigation bars) flow right-to-left in Arabic UIs.
- Images and culturally neutral content (e.g., a sunrise) should NOT be mirrored.
- Carousel interactions and animations must be inverted.
- Temperature scales, progress bars, and date/time progressions reverse direction.
- Pagination dots flow right-to-left; primary pages anchor on the far right.
- Charts with time components (days, weeks) should progress right-to-left; non-time charts depend on country conventions.
- Toggles, segmented controllers, and battery graphs all mirror in Arabic.

### Arabic Script Features
- Arabic is a connected script; each letter can have multiple glyphs depending on its position (isolated, initial, medial, final).
- Words tend to be more compact width-wise but taller due to dots, vocalization marks, and diacritics.
- If heavy vocalization marks are used, increase vertical spacing in the UI to prevent clipping.

### Typography
- Apple provides **SF Arabic**, a system typeface designed for legibility and consistency with the Latin SF family, available in all weights from Ultralight to Black.
- **SF Arabic** uses optical sizing: large sizes favor contemporary grotesque style; small sizes add angularity and contrast for legibility.
- **SF Arabic Rounded** introduced this year, available in all weights (Ultralight to Black).
- Uppercase Latin paired with Arabic: increase Arabic font size by ~10% to compensate for optical imbalance.
- Arabic is not case-sensitive; letter-spacing (tracking) should be set to 0 to avoid broken letter connections.
- Transparency in fonts can expose visible joints; the system font applies opacity at the word/phrase level to prevent distortion.

### Iconography
- Over 300 right-to-left and Arabic-specific symbols exist in SF Symbols.
- Directionality-sensitive icons (text alignment indicators, writing tools, speaker direction, calendar progress dots) should be mirrored.
- Direction-neutral icons (magnifying glass reflecting right-hand use, clock hands matching physical clocks) should NOT be mirrored.
- SF Symbols app: inspect the Localization section of the Info panel to find Arabic/RTL variants.
- RTL and local symbols appear automatically when using Apple system APIs.

### Arabic Numerals
- Western Arabic numerals (0–9) are used in North/West Africa; Eastern Arabic numerals used in Levant and Gulf countries; some countries (Egypt, Saudi Arabia) use both.
- The system selects the appropriate numeral form automatically based on user region and preferences.
- Apps using numerals should support both forms.

## APIs & Frameworks
- **SwiftUI** — automatic RTL layout mirroring **[NEW behavior in iOS 16]**
- **SF Arabic** — system Arabic typeface (all weights, optical sizes)
- **SF Arabic Rounded** — new Arabic rounded typeface **[NEW]**
- **SF Symbols** — 300+ Arabic/RTL localized symbol variants
- `.environment(\.layoutDirection, .rightToLeft)` — layout direction environment value
- **Human Interface Guidelines: Right to Left** — design guidance resource

## Code Highlights
No specific code samples were shown in this design-focused session. The key implementation note is:

```swift
// SwiftUI automatically mirrors layout for RTL locales.
// Avoid forcing layout direction unless necessary.
// Use environment value to inspect or override if needed:
@Environment(\.layoutDirection) var layoutDirection
```

For SF Arabic Rounded, use standard font APIs — the system selects the appropriate typeface variant automatically based on locale.

## Takeaways
- Arabic UI design requires full right-to-left layout mirroring for navigation, carousels, scales, and charts — but culturally neutral imagery stays as-is.
- Use SF Arabic and the new SF Arabic Rounded typeface at the correct optical sizes; set letter tracking to 0 and apply a ~10% size increase when mixing with uppercase Latin.
- SF Symbols provides 300+ Arabic/RTL-localized icons that appear automatically via system APIs — review each icon for directionality relevance rather than blindly mirroring all symbols.
- Support both Western and Eastern Arabic numeral forms; the system chooses automatically by region but apps must not hardcode a single form.

---
_Source: WWDC22 Session 10034 page (abstract, chapter summaries, code samples, and resource links)._
