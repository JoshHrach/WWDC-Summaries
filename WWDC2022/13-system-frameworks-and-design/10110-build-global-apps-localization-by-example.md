# Build Global Apps: Localization by Example
**WWDC22 · Session 10110** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10110/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9

## Overview
This session uses Apple's own Weather and Mail apps as concrete case studies to teach localization best practices. It covers four interconnected topics: declaring and commenting localized strings correctly (including the new `defaultValue` parameter for disambiguation), downloading and serving server-side content in the user's preferred language using `Bundle.preferredLocalizations(from:)`, combining formatters with plural rules via stringsdict for grammatically correct sentences, and exporting/importing localizations for Swift packages — now a first-class workflow in Xcode 14.

A second major theme is layout: localized text is often significantly longer or taller than its English equivalent, and the session presents `Grid` (new in SwiftUI) and `ViewThatFits` (new in SwiftUI) as the recommended tools for building layouts that accommodate varying text lengths without hiding UI elements or clipping strings.

## Key Topics

### Declaring & Disambiguating Localized Strings
- `String(localized:comment:)` — the standard API for declaring localizable strings in Swift
- **New**: `String(localized:defaultValue:comment:)` — separate the lookup key from the display value **[NEW]**
  - Use when the same English word appears in two different UI contexts and other languages need different words (e.g., "Archive" as a folder name vs. "Archive" as a menu action)
  - Set `key` to a unique descriptor (e.g., `"Archive.label"`, `"Archive.menuItem"`) and `defaultValue` to the English string
- Always write descriptive `comment` arguments: which UI element, where it appears, what variables mean at runtime

### Downloading Localized Remote Content
- `Bundle.preferredLocalizations(from:) -> [String]` — matches a list of server-supported language codes against the user's language preferences
- Returns sorted candidates; use `.first` as the language tag for subsequent server requests
- Pattern: server sends available language list → app calls `preferredLocalizations` → app requests content in the matched language
- Ensures all user-visible server content is in the user's preferred language

### Formatters & Plural Rules
- Use `.formatted()` or `Text(value, format:)` for all numeric, date, measurement, and currency values — formatters automatically respect the user's locale, numbering system, and unit preferences
- Key formatter usage patterns:
  - `humidity.formatted(.percent)` — percent with locale-appropriate symbol placement
  - `amountOfRain.formatted(.measurement(width: .narrow, usage: .rainfall))` — measurement with usage hint
  - `date.formatted(.dateTime.year().month())` — component-selective date
  - `price.formatted(.currency(code: "EUR"))` — currency
  - `bytesCopied.formatted(.byteCount(style: .file))` — file size
- `UnitLength(forLocale:usage:)` — get the user's preferred unit for a specific usage (e.g., `.rainfall`)
- For strings that embed a formatted value AND need plural variation, combine a formatter with a stringsdict:
  - Format the value to a `String` using a formatter
  - Embed both the integer count and formatted string as `stringsdict` parameters
  - stringsdict uses `NSStringPluralRuleType` to select the correct grammatical form per language
- Automatic Grammar Agreement (`^[\(count) word](inflect: true)` syntax) — simpler alternative for some plural cases

### Swift Package Localization (New Xcode 14 Workflow)
- `Package.defaultLocalization: LanguageTag` — declare the development language for a Swift Package **[NEW Xcode 14 support]**
- Once set, Xcode 14 enables "Export Localizations" and "Import Localizations" from the Product menu for packages
- Load strings from a package using `String(localized:bundle:.module)` — `.module` is the package's bundle
- Workflow is now identical to localizing an app target

### Layout for Localization: Grid and ViewThatFits
- `Grid` (SwiftUI) — new 2D grid layout for aligning heterogeneous content; use `GridRow` for each row; column widths expand to fit the widest content in that column **[NEW in SwiftUI]**
  - Use `grid(alignment:)` with `.leading` for proper RTL adaptation
  - `gridColumnAlignment(.trailing)` on individual cells
- `ViewThatFits` (SwiftUI) — provide multiple layout alternatives; SwiftUI picks the first one that fits without clipping **[NEW in SwiftUI]**
  - Use for horizontal→vertical stack transitions when a translated label is too long
  - Never hide elements; always provide a layout that accommodates longer text
- Do not give text containers a fixed height — script height varies by language (e.g., Hindi is taller)
- Do not assume spacing/padding that works for English will work for other languages

## APIs & Frameworks

**Foundation — Strings** **[NEW]**
- `String(localized:defaultValue:comment:)` — key/value split for disambiguation **[NEW]**
- `String(localized:bundle:comment:)` — load from a specific bundle (required for Swift packages)
- `Bundle.preferredLocalizations(from: [String]) -> [String]` — match server language list to user prefs

**Foundation — Formatters**
- `Numeric.formatted(.percent)` — locale-aware percent
- `Measurement.formatted(.measurement(width:usage:))` — measurement with usage hint
- `Date.formatted(.dateTime...)` — component-selective date formatting
- `Numeric.formatted(.currency(code:))` — currency
- `Numeric.formatted(.byteCount(style:))` — file sizes
- `UnitLength(forLocale: .current, usage: .rainfall)` — preferred unit for a usage

**SwiftUI — Layout** **[NEW]**
- `Grid(alignment:) { GridRow { ... } }` **[NEW]**
- `GridRow` — one row in a `Grid`
- `.gridColumnAlignment(_ alignment: HorizontalAlignment)` **[NEW]**
- `ViewThatFits(in:) { View1; View2; ... }` **[NEW]** — first layout that fits without clipping

**Xcode 14 — Localization**
- `Package.defaultLocalization: LanguageTag` in `Package.swift`
- Product > Export Localizations / Import Localizations now works for Swift Packages **[NEW]**

## Code Highlights

Disambiguating same English word in two contexts:
```swift
// Two strings for the same English word "Archive" in different contexts
let archiveFolderName = String(
    localized: "Archive.label",
    defaultValue: "Archive",
    comment: "Name of the Archive folder in the sidebar")

let archiveMenuAction = String(
    localized: "Archive.menuItem",
    defaultValue: "Archive",
    comment: "Menu item title for moving an email into the Archive folder")
```

Matching server languages to user preferences:
```swift
let allServerLanguages = ["bg", "de", "en", "es", "kk", "uk"]
if let preferredLanguage = Bundle.preferredLocalizations(from: allServerLanguages).first {
    // Use preferredLanguage in subsequent server requests
}
```

Combining a formatter with a stringsdict for plurals:
```swift
func expectedPrecipitationIn24Hours(for valueInMillimeters: Measurement<UnitLength>) -> String {
    let preferredUnit = UnitLength(forLocale: .current, usage: .rainfall)
    let converted = valueInMillimeters.converted(to: preferredUnit)
    let formattedValue = converted.formatted(.measurement(width: .narrow, usage: .asProvided))
    let integerValue = Int(converted.value.rounded())
    return String(
        localized: "EXPECTED_RAINFALL",
        defaultValue: "\(integerValue) \(formattedValue) expected in next \(24)h.",
        comment: "How much precipitation is expected in the next 24 hours")
}
```

Grid layout for 10-day forecast:
```swift
Grid(alignment: .leading) {
    ForEach(rows) { row in
        GridRow {
            Text(row.dayOfWeek)
            Image(systemName: row.weatherCondition).symbolRenderingMode(.multicolor)
            Text(row.minimumTemperature).gridColumnAlignment(.trailing)
            Capsule().fill(Color.orange).frame(height: 4)
            Text(row.maximumTemperature)
        }
    }
}
```

ViewThatFits for layout adaptation:
```swift
ViewThatFits {
    HStack { button1; button2; button3; button4 }  // preferred horizontal layout
    VStack {                                         // fallback for longer translations
        HStack { button1; button2 }
        HStack { button3; button4 }
    }
}
```

## Takeaways
- Use `String(localized:defaultValue:comment:)` whenever the same English word appears in multiple UI contexts — other languages often need different words, and a unique key lets translators provide the correct one.
- Always match server-provided content language to the user's preference using `Bundle.preferredLocalizations(from:)` rather than assuming English or the device region.
- For strings containing measured values with plural variation, combine a `Measurement` formatter with a stringsdict file — the formatter handles locale-appropriate unit display while the stringsdict handles grammatical number agreement.
- Use `ViewThatFits` and `Grid` (both new in SwiftUI) to build layouts that gracefully accommodate longer or taller localized text without hiding UI elements.

---
_Source: WWDC22 Session 10110 page (abstract, transcript, and code samples)._
