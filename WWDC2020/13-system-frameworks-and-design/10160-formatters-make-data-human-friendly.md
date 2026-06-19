# Formatters: Make Data Human-Friendly
**WWDC20 · Session 10160** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10160/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This session is a comprehensive tour of Foundation's Formatter APIs, showing how to correctly display dates, times, measurements, names, lists, numbers, and localized strings. The core message: always use the system formatter APIs rather than constructing format strings manually, because the formatters handle locale-specific rules (decimal separators, name order, grammatical gender, plural forms) automatically and improve over time without code changes.

iOS 14 includes an improved algorithm for formatting numbers in language/region combinations — for example, a device running in French in the UAE will now use UAE-appropriate number formats rather than French ones, a change that improves hundreds of language-region combinations automatically.

A companion sample app is provided for exploring all APIs interactively.

## Key Topics

**Dates and Times**
- Predefined styles (`.short`, `.medium`, `.long`, `.full`) for common use cases
- Template-based formatting via `setLocalizedDateFormatFromTemplate(_:)` for custom combinations
- Unicode Date Field Symbol Table reference for choosing correct symbols (e.g., standalone `c` vs. format-context `e` for weekdays)
- Template field order is irrelevant — formatter assembles correctly for each locale
- Never set `dateFormatter.dateFormat` to a template string directly (use the template method)

**Duration and Interval Formatting**
- `DateComponentsFormatter` for durations (e.g., "2 hr, 26 min")
- `DateIntervalFormatter` for ranges (avoids repeating shared elements, e.g., "Jan 1–3" not "Jan 1–Jan 3")
- `RelativeDateTimeFormatter` for natural past/future language (e.g., "yesterday", "in 2 hours")

**Measurements**
- `MeasurementFormatter` handles unit conversion (metric input → imperial display for US) and localization
- Supports temperature, speed, pressure, distance, and many more via `UnitTemperature`, `UnitSpeed`, `UnitPressure`, etc.
- Supports custom unit types for app-specific measurements

**Names**
- `PersonNameComponentsFormatter` with `.default`, `.short`, `.abbreviated` styles
- Short style respects user preferences (may show nickname)
- Abbreviated style for monograms — check `.count <= 2` before using (Swift count is user-visible character count, not code points)
- Always falls back to icon when monogram does not fit
- Automatically applies correct name order for CJK names (family name first) regardless of device language

**Lists**
- `ListFormatter.localizedString(byJoining:)` handles locale-specific conjunctions
- iOS 14 update: Spanish "and" (`y` vs. `e`) now varies correctly by context **[UPDATED]**
- Works on arrays of `String` or any `CustomStringConvertible` elements

**Numbers**
- `NumberFormatter` with styles: `.decimal`, `.percent`, `.currency`, `.scientific`, `.spellOut`
- Automatically selects locale-appropriate decimal separator, grouping separator, and symbol placement
- `formatter.percentSymbol` / `formatter.decimalSeparator` for accessing locale symbols
- Percent sign position varies by locale (e.g., `71%` in English, `%71` in Turkish)

**Strings and Pluralization**
- SwiftUI `Text` with string interpolation works with `.stringsdict` files for plural-aware localization
- `.stringsdict` encodes language-specific plural rules (singular, dual, plural forms)
- Arabic has 6 plural categories; stringsdict handles all automatically
- Write `Text("\(photosCount) Photos Selected")` once; translators provide locale-specific `.stringsdict` entries

## APIs & Frameworks

### Foundation — Date/Time Formatters
- `DateFormatter` — configurable date/time formatter
  - `.dateStyle` / `.timeStyle` — predefined styles (`.none`, `.short`, `.medium`, `.long`, `.full`)
  - `setLocalizedDateFormatFromTemplate(_:)` — template-based format (locale-aware)
  - `dateFormat` property — DO NOT set template string here directly
- `DateComponentsFormatter` — formats durations from `DateComponents`
  - `.unitsStyle` — `.abbreviated`, `.short`, `.full`, `.spellOut`, `.positional`
- `DateIntervalFormatter` — formats start/end date ranges without repetition
  - `.dateTemplate` — template string for interval display
- `RelativeDateTimeFormatter` **[NEW in iOS 13, expanded]** — natural-language relative dates
  - `.dateTimeStyle` — `.named` ("yesterday"), `.numeric` ("1 day ago")
  - `localizedString(from:)` — format from `DateComponents`

### Foundation — Measurement Formatter
- `MeasurementFormatter` — formats `Measurement<Unit>` values with automatic unit conversion
- `Measurement<UnitTemperature>`, `Measurement<UnitSpeed>`, `Measurement<UnitPressure>` — typed measurement values
- Supports all `Dimension` subclasses and custom `Unit` types

### Foundation — Name Formatter
- `PersonNameComponentsFormatter` — formats `PersonNameComponents` for display
  - `.style` — `.default`, `.short`, `.medium`, `.long`, `.abbreviated`
- `PersonNameComponents` — struct with `familyName`, `givenName`, `middleName`, `namePrefix`, `nameSuffix`, `nickname` properties

### Foundation — List Formatter
- `ListFormatter` — formats arrays into natural-language lists **[UPDATED in iOS 14]**
  - `localizedString(byJoining:)` — class method, simplest usage
  - `.locale` — override default locale
  - `.itemFormatter` — formatter applied to each item before joining

### Foundation — Number Formatter
- `NumberFormatter` — formats numeric values
  - `.numberStyle` — `.decimal`, `.percent`, `.currency`, `.scientific`, `.spellOut`, `.ordinal`
  - `.maximumFractionDigits` / `.minimumFractionDigits` — precision control
  - `.percentSymbol` — locale's percent symbol
  - `.decimalSeparator` — locale's decimal separator

### SwiftUI
- `Text` with string interpolation + `.stringsdict` — plural-aware localized strings
- Works with `LocalizedStringKey` interpolation for automatic stringsdict lookup

## Code Highlights

Date with medium style and time:
```swift
let df = DateFormatter()
df.dateStyle = .medium
df.timeStyle = .short
df.string(from: Date())  // "Jan 5, 2020 at 9:41 AM"
```

Template-based weekday abbreviation:
```swift
let df = DateFormatter()
df.setLocalizedDateFormatFromTemplate("ccccc")  // "M", "T", "W" etc.
df.string(from: Date())
```

Relative date:
```swift
let f = RelativeDateTimeFormatter()
f.dateTimeStyle = .named
f.localizedString(from: DateComponents(day: -1))  // "yesterday"
```

Measurement with automatic unit conversion:
```swift
let f = MeasurementFormatter()
f.numberFormatter.maximumFractionDigits = 0
let temp = Measurement<UnitTemperature>(value: 16, unit: .celsius)
f.string(from: temp)  // "61°F" for US locale
```

Monogram with length guard:
```swift
let f = PersonNameComponentsFormatter()
f.style = .abbreviated
let monogram = f.string(from: nameComponents)
if monogram.count <= 2 { /* use monogram */ } else { /* use icon */ }
```

Locale-aware list:
```swift
ListFormatter.localizedString(byJoining: ["English", "Spanish"])
// "English and Spanish" (en) / "Inglés y español" (es)
```

Plural-aware SwiftUI text (pairs with .stringsdict):
```swift
Text("\(photosCount) Photos Selected")
```

## Takeaways
- Always use system formatters for dates, numbers, measurements, names, lists, and pluralized strings — they handle hundreds of locale-specific rules that manual string construction would miss.
- Use `setLocalizedDateFormatFromTemplate(_:)` (not `.dateFormat`) for custom date templates; field order in the template does not matter.
- iOS 14 improves number formatting for hundreds of language-region combinations and updates `ListFormatter` with grammatically correct conjunctions in several languages — apps get these improvements for free.
- Use `.stringsdict` with SwiftUI `Text` interpolation for plural forms; stringsdict encodes the full set of plural rules for every supported language including Arabic's six categories.

---
_Source: WWDC20 Session 10160 page (abstract, chapter summaries, code samples, and resource links)._
