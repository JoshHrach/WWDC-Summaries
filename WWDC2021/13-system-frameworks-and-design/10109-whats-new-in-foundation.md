# What's New in Foundation
**WWDC21 · Session 10109** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10109/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session covers three landmark additions to Foundation in 2021, all centered on internationalization and localization. The first is a brand-new value-type `AttributedString` designed specifically for Swift, which replaces the decades-old `NSAttributedString` reference type with a safer, Codable, and strongly-typed alternative that supports Swift string interpolation and Markdown-based attribute markup in localized strings.

The second addition is a completely rethought Formatter API that eliminates the need to create, configure, and cache formatter objects. Instead, any `Date`, `Int`, `Double`, or collection value can call `.formatted()` directly with a composable style argument, gaining type safety, autocompletion, and better performance. Ten formatter types have been updated. The third addition is Automatic Grammar Agreement — a new engine powered by the same technology as keyboard suggestions that automatically inflects localized strings to match grammatical gender, number agreement, and the user's personal term of address in supported languages (initially English and Spanish).

## Key Topics

### AttributedString (Swift-First)
`AttributedString` is a value type (`struct`) with the same Unicode scalar-counting behavior as Swift `String`. Attributes are accessed as strongly-typed properties rather than `Any`-typed dictionaries. It provides two primary views: `.characters` (a `Collection` of `Character`) and `.runs` (a `Collection` of contiguous attribute ranges). Runs can be coalesced by a single key-path (e.g., `message.runs[\.link]`) to iterate a specific attribute. Slicing with ranges and the `replaceAttributes(_:with:)` method allow targeted attribute mutations. `AttributedString` can be decoded from Markdown (standard inline styles plus a custom `^[text](key: value)` syntax for app-defined attributes), initialized from a localized string key, and encoded/decoded with `Codable`.

### Custom Attribute Definition and Scopes
Developers define custom attributes by conforming a type to `AttributedStringKey` (requires `name: String` and associated `Value` type). Adding `CodableAttributedStringKey` conformance makes the attribute serializable; adding `MarkdownDecodableAttributedStringKey` lets the attribute be specified in Markdown's custom attribute syntax using JSON 5. Attribute scopes (`AttributeScope` protocol, nested inside `AttributeScopes`) group attribute keys for use in Markdown decoding, NSAttributedString conversion, and archiving.

### New Formatter API
All formatters now expose a `.formatted(_ style:)` method directly on the value being formatted (e.g., `date.formatted(.dateTime.year().day().month(.wide))`). Format styles are composable and type-safe; field ordering does not matter. Ten formatter types are supported: dates, times, ISO 8601, date ranges, duration, relative dates, integers, decimals, percentages, and currency. Parse strategies allow round-tripping between strings and typed values. SwiftUI `TextField` accepts format styles via `TextField("Amount", value: $tip, format: .percent)`. Formatter output can be returned as `AttributedString` for post-processing (e.g., coloring the weekday portion of a formatted date).

### Automatic Grammar Agreement
Using the `^[...](inflect: true)` Markdown custom attribute syntax, localizers can mark regions of a string that the grammar engine should automatically inflect for number and gender agreement. The user's term of address (masculine/feminine/neutral) is now configurable in Language & Region settings on iOS/macOS 15/Monterey, and apps can reflect this in localized copy without writing custom switching logic.

## APIs & Frameworks

- **Foundation** framework
- `AttributedString` **[NEW]** — value-type attributed string
  - `AttributedString(_:)` — plain initializer
  - `AttributedString(localized:)` **[NEW]** — localized string initializer with Markdown support
  - `.characters` — `BidirectionalCollection<Character>` view
  - `.runs` — collection of attribute runs
  - `.runs[\.keyPath]` — coalesced run iteration by attribute key path
  - `.mergeAttributes(_:)` — merge an `AttributeContainer` into the whole string
  - `.replaceAttributes(_:with:)` — replace matching attribute container with another
  - `AttributedString.Index` — position in characters or runs views
  - Subscript with range — slice for attribute setting
- `AttributeContainer` **[NEW]** — holds attribute key-value pairs without string
- `AttributedStringKey` protocol **[NEW]** — defines a custom attribute (requires `name` and `Value`)
- `CodableAttributedStringKey` protocol **[NEW]** — adds `Codable` to an attribute key
- `MarkdownDecodableAttributedStringKey` protocol **[NEW]** — enables custom Markdown syntax decoding
- `AttributeScope` protocol **[NEW]** — groups attribute keys for decoding/conversion
- `AttributeScopes` **[NEW]** — namespace for all attribute scope types
  - `AttributeScopes.SwiftUIAttributes` — SwiftUI-defined attributes scope
  - `AttributeScopes.UIKitAttributes` — UIKit attributes scope
  - `AttributeScopes.AppKitAttributes` — AppKit attributes scope
  - `AttributeScopes.FoundationAttributes` — Foundation attributes scope
- `NSAttributedString(AttributedString)` — conversion from new type to legacy type
- Markdown inline styles supported in `AttributedString(localized:)`: `**bold**`, `_italic_`, `[link](url)`, `~~strikethrough~~`, `` `code` ``
- Custom Markdown attribute syntax: `^[text](attributeName: value)` (JSON 5 values)
- `AttributedString.runs[\.dateField]` — run iteration by Foundation date field attribute **[NEW]**
- `AttributeContainer.dateField(_:)` **[NEW]** — container for date field attributes
- `Date.formatted()` **[NEW]** — format with default style
- `Date.formatted(_ style: FormatStyle)` **[NEW]** — format with explicit style
- `Date.FormatStyle` **[NEW]** — composable date format style (`.dateTime`, `.iso8601`)
  - `.year()`, `.month(_:)`, `.day()`, `.weekday(_:)`, `.hour()`, `.minute()`, `.second()`
  - `.month(.wide)` — full month name
  - `.weekday(.wide)` — full weekday name
  - `.dateSeparator(.dash)` — ISO 8601 separator
  - `.locale(_:)` — override locale
  - `.attributed` — return `AttributedString` instead of `String`
- `Date.ISO8601FormatStyle` **[NEW]** — ISO 8601-specific format style
- `Date.ParseStrategy` **[NEW]** — parse `Date` from a `String`
  - `Date(string, strategy:)` **[NEW]** — throwing initializer
- Date range formatting: `(date..<date).formatted()`
- `Duration.formatted(.timeDuration)` **[NEW]**
- `Duration.formatted(.components(style: .wide))` **[NEW]**
- `Date.RelativeFormatStyle` **[NEW]** — `date.formatted(.relative(presentation:unitsStyle:))`
- `Int.formatted()` / `Double.formatted()` **[NEW]** — format number with default style
- `Int.formatted(.percent)`, `.formatted(.number.notation(.scientific))`, `.formatted(.currency(code:))` **[NEW]**
- `[T].formatted(.list(memberStyle:type:))` **[NEW]** — localized list format
- `TextField(_:value:format:)` **[NEW]** SwiftUI — bind text field to typed value with format style
- Automatic grammar agreement: `^[...](inflect: true)` Markdown attribute **[NEW]**
- `InflectionRule` — controls inflection behavior **[NEW]**
- `Morphology` — describes grammatical gender/number used by agreement engine **[NEW]**
- Term of address preference: Language & Region settings, iOS 15 / macOS Monterey **[NEW]**
- `Date.now` — static property returning current date/time
- JSON 5 support in `JSONDecoder` **[NEW]**

## Code Highlights

**Creating and mutating an AttributedString:**
```swift
var thanks = AttributedString("Thank you!")
thanks.font = .body.bold()

var website = AttributedString("Please visit our website.")
website.link = URL(string: "http://www.example.com")

var container = AttributeContainer()
container.foregroundColor = .red
thanks.mergeAttributes(container)
```

**Coalesced run iteration for a specific attribute:**
```swift
for (value, range) in message.runs[\.link] {
    if let v = value, v.scheme != "https" { /* flag insecure link */ }
}
```

**New date formatting:**
```swift
let formatted = Date.now.formatted(.dateTime.year().day().month(.wide))
// "June 7, 2021" (locale-sensitive)
let isoDate = Date.now.formatted(.iso8601.year().month().day().dateSeparator(.dash))
// "2021-06-07"
```

**Automatic grammar agreement:**
```swift
let message = AttributedString(localized:
    "Add ^[\(quantity) \(size) \(food)](inflect: true) to your order")
// Engine auto-inflects for number and gender in Spanish, English, etc.
```

**Custom attribute with Markdown:**
```swift
enum RainbowAttribute: CodableAttributedStringKey, MarkdownDecodableAttributedStringKey {
    enum Value: String, Codable { case plain, fun, extreme }
    static var name = "rainbow"
}
let header = AttributedString(localized: "^[Fast & Delicious](rainbow: 'extreme') Food",
                               including: \.caffeApp)
```

## Takeaways

- `AttributedString` replaces `NSAttributedString` with a safe, value-type, Codable, Markdown-capable attributed string that integrates directly into SwiftUI's `Text` view.
- The new `.formatted()` API eliminates formatter caching boilerplate and brings type safety and autocompletion to all ten Foundation formatter types, including dates, numbers, currencies, and lists.
- Automatic grammar agreement with `^[...](inflect: true)` dramatically reduces the number of localized strings needed for languages with grammatical gender and number agreement.
- The new term-of-address preference (iOS 15 / macOS Monterey) allows apps to address users in their preferred grammatical form with no custom logic — just the `inflect` attribute.

---
_Source: WWDC21 Session 10109 page (abstract, chapter summaries, code samples, and resource links)._
