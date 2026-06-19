# Localization Best Practices on tvOS
**WWDC17 · Session 248** · [Watch](https://developer.apple.com/videos/play/wwdc2017/248/)

_Platforms:_ tvOS 11

## Overview
tvOS 11 adds Arabic and Hebrew as selectable system languages, making right-to-left (RTL) support on Apple TV a priority for all developers. This session covers the three pillars of a localized tvOS experience: text handling, layout and image adaptation, and the Xcode-based workflow for exporting, importing, and testing translations. The session applies to both UIKit-based tvOS apps and TVMLKit-based client-server apps.

For text, the session emphasizes wrapping all user-visible strings in `NSLocalizedString`, including meaningful translator comments on every string, and using system formatters (dates, numbers, currencies, units) instead of hand-crafted localized strings. For server-fetched content, the `Bundle` API provides the currently running language identifier and can compute the best language match from a server-provided set.

Layout adaptation must handle both variable string lengths (translations can be significantly shorter or longer than the source language) and script directionality. UIStackView backed by Auto Layout with leading/trailing constraints handles both automatically. TVMLKit templates gain new leading/trailing position and alignment APIs for RTL languages, along with media query support for directional margins.

## Key Topics
- **New in tvOS 11** — Arabic and Hebrew added as selectable system languages; new TVMLKit RTL APIs
- **NSLocalizedString** — wrap all code strings; include translator comments on every string; Storyboard strings are localizable by default
- **Formatter APIs** — use system formatters for dates, numbers, currencies, units; these handle regional formatting variants automatically
- **Server-side language matching** — `Bundle.main.preferredLocalizations.first` returns the running language identifier; `Bundle.preferredLocalizations(from:)` computes best match from a server-provided set
- **Language fallbacks and regional variants** — bundle APIs automatically resolve `es-MX` → `es-419`, `zh-CN` → `zh-Hans`, etc.
- **UIStackView + Auto Layout** — leading/trailing constraints automatically reverse for RTL without extra code; nested stack views support complex layouts
- **TVMLKit RTL APIs (new)** — new leading/trailing position/alignment properties for TVMLKit elements; new media queries for directional margins
- **Image directionality** — three categories: universal (no change), mirror-able (Xcode asset catalog "Mirrors" setting), and locale-specific (separate assets for each direction)
- **Xcode Localization workflow** — add languages in Project Editor; Export for Localization (XLIFF); Import Localizations; Static Analyzer for missing localizable string markers and comments
- **Testing** — Scheme Editor > Options > Application Language to run in a specific locale; pseudo-language options including RTL pseudo-language to test direction without knowing a RTL language

## APIs & Frameworks

### Foundation
- **`NSLocalizedString(_:comment:)`** — marks string for export to XLIFF; comment parameter displayed to translators
- **`Bundle.main.preferredLocalizations`** — ordered array of language identifiers the app is running in; `.first` gives the active language
- **`Bundle.preferredLocalizations(from:forPreferences:)`** — computes best language match from a set (e.g., server-supported languages)
- **`DateFormatter`** — locale-aware date/time formatting; respects 12/24-hour preference, calendar, etc.
- **`NumberFormatter`** — locale-aware number and currency formatting
- **`MeasurementFormatter`** — locale-aware unit formatting (length, mass, etc.)
- **`Locale`** — current locale; `Locale.current.languageCode`, `Locale.current.regionCode`

### UIKit
- **`UIStackView`** — `axis`, `distribution`, `alignment`; automatically respects leading/trailing for RTL
- **Auto Layout leading/trailing constraints** — `NSLayoutConstraint` with `.leading`/`.trailing` attributes; reverse automatically in RTL
- **`UIView.semanticContentAttribute`** — `.forceLeftToRight`, `.forceRightToLeft`, `.unspecified` for overriding default RTL behavior on specific views
- **`UIApplication.userInterfaceLayoutDirection`** — `.leftToRight` or `.rightToLeft`; query current direction at runtime

### TVMLKit **[NEW RTL APIs in tvOS 11]**
- **Leading/trailing position attributes** — new TVMLKit element attributes for directional positioning (replaces left/right)
- **Leading/trailing alignment** — new TVMLKit element attributes for directional text alignment
- **Media queries for layout direction** — `@media` conditions for `direction: ltr` / `direction: rtl` to specify directional margins and padding

### Xcode Tools
- **Export for Localization** (Editor menu) — generates XLIFF files from all localizable strings and Storyboard text
- **Import Localizations** (Editor menu) — imports translated XLIFF back into the project
- **Static Analyzer** — detects unlocalizable strings passed to UI and missing translator comments
- **Scheme Editor Application Language** — run app in any supported language without changing system language
- **Pseudo-languages** — Right-to-Left Pseudolanguage, Accented Latin, etc. for layout testing
- **Asset Catalog Localization Direction** — per-asset setting: Fixed, Mirrors, or locale-specific variants

## Code Highlights

```swift
// Mark string as localizable with translator context
let label = NSLocalizedString("subscribe",
    comment: "Button label in content detail screen; subscribes user to channel")

// Format date locale-appropriately (no NSLocalizedString needed)
let formatter = DateFormatter()
formatter.timeStyle = .short
let displayTime = formatter.string(from: Date())

// Get running language to send to server
let language = Bundle.main.preferredLocalizations.first ?? "en"
// e.g., "es-419" for Latin American Spanish

// Get best match from server-supported languages
let serverLanguages = ["en", "es", "fr", "de"]
let best = Bundle.preferredLocalizations(from: serverLanguages).first ?? "en"
```

## Takeaways
- tvOS 11's addition of Arabic and Hebrew makes RTL support non-optional; UIStackView with Auto Layout leading/trailing constraints provides RTL adaptation essentially for free.
- Always include translator comments on every `NSLocalizedString` call; poor comments lead to unnatural translations that undermine the localization investment.
- Use system formatters (DateFormatter, NumberFormatter, MeasurementFormatter) instead of hand-crafted localized strings for dates, numbers, and units — they handle dozens of regional formatting variations automatically.
- The Xcode XLIFF export/import workflow and the pseudo-language scheme options let teams localize and validate their apps without needing native speakers during development.

---
_Source: WWDC17 Session 248 page (abstract, chapter summaries, code samples, and resource links)._
