# The Practice of Inclusive Design
**WWDC21 · Session 10275** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10275/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session presents six concrete practices for designing apps and games that welcome people from diverse backgrounds and perspectives. Presented by Linda Dong (Design Evangelist) and Sam Iglesias (Apple Design Team prototyper), the session frames inclusive design not just as usability across ability levels, but as making all people — regardless of gender, culture, language, age, and background — feel respected and represented.

The session covers how content (words, imagery, marketing materials) communicates to potential users before they even download an app. App Store screenshots, example data, placeholder icons, and copy all signal who an app was designed for. Small, thoughtful changes — diverse names in examples, culturally varied food choices, gender-neutral icons — can meaningfully widen the audience that feels welcomed.

Technical accessibility is framed as an inseparable part of inclusive design: Dynamic Type, VoiceOver labels, contrast ratios, and Reduce Motion are not optional extras but requirements for truly inclusive apps.

## Key Topics

### 1. Tell Diverse Stories
- App Store screenshots, names, and scenarios signal whom the app is for
- Use names and activities that represent diverse cultures, holidays, and communities
- Avoid defaulting to culturally narrow or familiar-only examples

### 2. Avoid Stereotypes
- Gender binary assumptions: use gender-neutral pronouns ("they") and non-gendered placeholder icons
- SF Symbols provides gender-neutral human glyphs
- Ableist language (e.g., "insane," "crazy" used negatively): replace with neutral alternatives
- Represent people with disabilities in photography, emoji, and game characters
- Avoid functionality assumptions (e.g., fixed family role assignments) that exclude non-standard family structures

### 3. Adopt Accessibility
- Dynamic Type: use SwiftUI or UIKit layout APIs to adapt automatically across text size preferences
- Bold Text: manually thicken non-text elements that convey information (e.g., indicators, circles)
- VoiceOver: add descriptive accessibility labels beyond SF Symbol names; set correct traversal order
- Testing: use Accessibility Shortcut and Control Center toggles (VoiceOver, Increase Contrast, Reduce Transparency, Reduce Motion, Grayscale Color Filter, Text Size)
- Grayscale testing reveals over-reliance on color; silent testing reveals over-reliance on audio

### 4. Localize for Culture
- Translation is necessary but insufficient: content must also be culturally relevant
- Avoid idioms and slang; prefer plain, direct language
- Respect cultural norms (dietary customs, holidays, locally relevant content)
- Avoid cultural caricatures in games

### 5. Use Color Mindfully
- Color associations differ by culture (e.g., red for fortune in many Asian cultures vs. danger in others)
- Stocks app shows green for gains in most regions, red for gains in China
- Color blindness affects ~5% of world population: do not rely solely on color to convey meaning
- Increase Contrast mode: aim for minimum 4.5:1 luminosity ratio between text and backgrounds
- System colors (iOS palette) include high-contrast variants automatically; custom palettes require manual calculation

### 6. Encourage Self-Expression
- Accept diverse name formats: hyphens, accents, single names, multiple family names, varying lengths
- Prefer a single "full name" field over separate first/last name fields
- Offer preferred name alongside legal name
- Gender selection: provide a broad spectrum of options, allow custom input, allow privacy choice
- Character creation in games: offer diverse appearances across skin tone, age, body type, hair, clothing without gender-gating options

## APIs & Frameworks

- `SwiftUI` font system (automatic Dynamic Type scaling)
- `UIKit` text layout APIs (Dynamic Type support)
- `UIAccessibility` (VoiceOver labels, ordering)
- `accessibilityLabel` (SwiftUI / UIKit)
- `accessibilityHint` (SwiftUI / UIKit)
- `accessibilitySortPriority` / `accessibilityElements` (traversal order)
- `UIFont.preferredFont(forTextStyle:)` (Dynamic Type)
- `UITraitCollection.preferredContentSizeCategory`
- `UIAccessibility.isBoldTextEnabled`
- SF Symbols (gender-neutral glyphs for diverse representation)
- `UIColor` system color palette with automatic high-contrast variants
- Human Interface Guidelines: Inclusion (referenced resource)
- Apple Style Guide: Writing Inclusively (referenced resource)

## Code Highlights

No code samples are included in this session. The session is design-focused with no code demonstrations.

Relevant API patterns mentioned:
- Setting VoiceOver labels in SwiftUI: `.accessibilityLabel("Account settings")`
- Setting VoiceOver labels in UIKit: `button.accessibilityLabel = "Account settings"`
- Setting VoiceOver traversal order: `.accessibilitySortPriority(1)` or `UIAccessibilityContainer` protocol

## Takeaways

- Inclusive design is not just about accessibility features — it is about making every person feel the app was designed with them in mind, through language, imagery, functionality, and content.
- Test your app with VoiceOver, Grayscale, Increase Contrast, and various Dynamic Type sizes before shipping; even temporary disabilities affect users.
- Avoid gender-binary and ableist assumptions in both copy and functionality (e.g., do not hardcode family roles or restrict name formats).
- Color choices carry cultural meaning and can unintentionally exclude or offend — validate with people from the cultures you are designing for.

---
_Source: WWDC21 Session 10275 page (abstract, chapter summaries, code samples, and resource links)._
