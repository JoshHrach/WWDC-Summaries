# Design for Arabic · صمّم بالعربي
**WWDC22 · Session 110441** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110441/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
This is the Arabic-language edition of session 10034 "Design for Arabic." It presents identical content delivered entirely in Arabic, making it the primary resource for Arabic-speaking designers and developers who want to learn best practices for designing or optimizing apps and games for Arabic-speaking audiences.

The session covers UI directionality (right-to-left layouts), Arabic script characteristics, Apple's SF Arabic and SF Arabic Rounded typefaces, iconography considerations using SF Symbols RTL variants, and the two Arabic numeral systems. The goal is to help creators reach the approximately 660 million Arabic script users across more than 22 countries.

Related English-language companion sessions include "Design for Arabic" (10034) and "Get it Right (to Left)" (10107) for the development implementation perspective.

## Key Topics

### UI Directionality
- All layout elements (titles, paragraphs, carousels, navigation bars) flow right-to-left in Arabic UIs.
- Culturally neutral content (images, videos, backgrounds such as a sunrise) should NOT be mirrored.
- Carousels, swipeable elements, animations, and interactions must be inverted.
- Temperature/progress scales, pagination dots, and date/time progression reverse direction.
- Charts with time components (days, weeks, months) should progress right-to-left; non-time charts depend on country conventions.
- Toggles, segmented controllers, and battery usage graphs all mirror in Arabic.
- Islamic Lunar Calendar month indicators are relevant cultural additions for Arabic/Islamic users.

### Arabic Script Features
- Arabic is a connected, cursive script; each letter has up to four contextual forms (isolated, initial, medial, final).
- Words are more compact in width but taller due to dots, vocalization marks (tashkeel), and diacritic marks.
- Heavy vocalization usage requires additional vertical spacing to prevent clipping.

### Typography
- **SF Arabic** — system Arabic typeface with optical sizing, matching the Latin SF family aesthetics; available in Ultralight to Black weights.
- **SF Arabic Rounded** — **[NEW]** introduced at WWDC22; all weights from Ultralight to Black; gives a softer, more approachable look.
- Large optical sizes (display) suit titles; small optical sizes prioritize legibility with added angularity and contrast.
- Mix with uppercase Latin: increase Arabic font size by ~10% to compensate for optical imbalance.
- Letter tracking/spacing: set to 0 for Arabic; the system font adds Kashida (letter extension) for organic spacing.
- Opacity applied per word/phrase (not per letter) by the system font to prevent visible glyph joints.

### Iconography
- 300+ Arabic/RTL-localized symbols in SF Symbols.
- Directionality-sensitive icons (text alignment, writing direction, calendar progress, speaker direction) should be mirrored.
- Direction-neutral icons (magnifying glass, clock hands) should NOT be mirrored.
- SF Symbols app: Localization section in the Info panel shows Arabic/RTL variants.
- All RTL and local symbols appear automatically via Apple system APIs.

### Arabic Numerals
- Western Arabic numerals (0–9): used in North/West Africa (Morocco, Algeria, Tunisia).
- Eastern Arabic numerals: used in Levant and Gulf countries.
- Many countries (Egypt, Saudi Arabia) use both; system selects automatically by region.
- Apps using numerals must support both forms.

## APIs & Frameworks
- **SwiftUI** — automatic RTL layout mirroring
- **SF Arabic** — system Arabic typeface (optical sizing, all weights)
- **SF Arabic Rounded** — new rounded Arabic typeface **[NEW]**
- **SF Symbols** — 300+ Arabic/RTL localized symbol variants; Localization panel in SF Symbols app
- `layoutDirection` environment value — inspect or override layout direction
- **Human Interface Guidelines: Right to Left** — canonical design reference

## Code Highlights
This is a design-focused session presented in Arabic. No unique code samples beyond those in the companion session 10034. Key implementation principle:

```swift
// SwiftUI handles RTL automatically for Arabic locales.
// SF Arabic Rounded is available via standard Font API:
Text("مرحبا").font(.system(.body, design: .rounded))
```

## Takeaways
- This Arabic-language session makes WWDC22 design guidance directly accessible to Arabic-speaking designers and developers without requiring English proficiency.
- The same RTL design principles apply: mirror layouts and interactive components, but preserve culturally neutral imagery and direction-neutral icons.
- SF Arabic Rounded (new in 2022) gives apps a softer aesthetic option while maintaining full weight range and optical sizing.
- Supporting both Western and Eastern Arabic numeral forms is essential for reaching the full Arabic-speaking market across different regions.

---
_Source: WWDC22 Session 110441 page (abstract, chapter summaries, code samples, and resource links)._
