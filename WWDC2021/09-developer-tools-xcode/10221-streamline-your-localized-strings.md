# Streamline Your Localized Strings
**WWDC21 · Session 10221** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10221/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
This session presents a comprehensive, end-to-end guide to building a streamlined localization workflow in Apple platforms. It covers how to declare localizable strings in SwiftUI, UIKit, and AppKit; how to organize strings across bundles and string tables; and how Xcode 13 can automatically extract, export, and import all localizable content — eliminating the need to manually maintain `.strings` files.

The session also covers new iOS 15 and macOS Monterey additions: the Swift-native `String(localized:)` API replacing `NSLocalizedString`, Markdown support in `AttributedString` for inline formatting, and automatic grammar agreement via the `inflect:` Markdown attribute. These new APIs let developers write correct, expressive, and culturally inclusive strings with minimal code.

## Key Topics

### Declaring Localizable Strings
- **SwiftUI**: any `Text`, `Button`, `Label`, or other view that accepts a `LocalizedStringKey` is automatically localizable — no extra work required. Use `Text(verbatim:)` for non-localizable mock/preview strings to avoid generating unnecessary translator work.
- **UIKit / AppKit / model code**: use `String(localized:comment:)` **[NEW in iOS 15/macOS Monterey]** — the Swift-native replacement for `NSLocalizedString`. Supports string interpolation with variables, proper pluralization, RTL variable isolation, and user-preferred digit formats automatically.
- Avoid `String(format:)` for localized strings — it does not handle bidirectional text, locale-specific digits, or plurals correctly.
- Avoid concatenating strings; use full sentences with interpolated variables instead.

### Comments Are Mandatory
- Every string must have a `comment:` parameter — comments are the primary context source for translators, who do not see the live app.
- A good comment states: the UI element type (button, label, VoiceOver text), the action or meaning (ordering tickets vs. sorting a list), and what each variable represents (example values help).
- Storyboard/XIB strings have a corresponding comment field in Xcode's Identity Inspector.

### Organizing Strings: Tables and Bundles
- By default all strings go into the `Localizable` table (`Localizable.strings`). Use a custom `tableName:` parameter to route strings into feature- or screen-specific files for large projects.
- The `bundle:` parameter controls which bundle's string file is loaded. Default is `.main`. For app extensions sharing strings with their host app, specify the host app's bundle. For frameworks vending localized strings, pass `Bundle(for: AnyClassInFramework.self)` to load from the framework's bundle.
- Use `Bundle.preferredLocalizations(from:).first` to determine the best language when requesting strings from a server.

### Xcode 13 Export/Import Localizations
- **Export Localizations**: Xcode reads all Swift and Foundation string declarations and generates `.xcloc` (localization catalog) packages — one per language — containing `.strings`, `.stringsdict`, and other assets. New in Xcode 13: Swift string extraction uses compiler support; workspaces are fully supported.
- **Import Localizations**: the Product menu's Import Localizations command reads a translated `.xcloc` and creates/updates all `.strings`, `.stringsdict`, and asset files automatically. Command-line equivalents (`xcodebuild -exportLocalizations` / `-importLocalizations`) enable CI automation.
- **In-Xcode editing** **[NEW in Xcode 13]**: exported `.xcloc` catalogs can be opened and edited directly in Xcode with a structured table UI showing each string, its comment, screenshots, and a translation field.
- UI test screenshots are now included in exported catalogs, giving translators visual context and providing App Store-ready localized screenshots.
- Custom wrapper methods around `String(localized:)` can be registered under the **Localized String Macro Names** build setting for extraction.

### Stringsdict for Plurals
- Use a `.stringsdict` file (opt-in, created via Xcode template, mark as Localized) for any string that pluralizes a number: e.g., "Order 1 Ticket" vs "Order 3 Tickets."
- The stringsdict format uses a token-substitution mechanism with `NSStringPluralRuleType` entries (`one`, `other`, and language-specific cases like `few`/`many` for Russian). Xcode adds language-specific cases automatically at export time.
- Do **not** use stringsdict for singular/plural based on logic other than a numeric count (e.g., "this/both/all tickets") — use simple `if`/`else` with separate `String(localized:)` calls in that case.

### New iOS 15 / macOS Monterey String APIs
- **`AttributedString(localized:comment:)`** **[NEW]**: localizable attributed string with Markdown support. Use `**bold**`, `_italic_`, and `[link](url)` inline — no risky manual character manipulation.
- **Automatic grammar agreement** **[NEW]**: use `^[\(count) Ticket](inflect: true)` inside `AttributedString(localized:)` to get correct pluralization and morphology (number agreement) computed at runtime in supported languages.
- **Term of address inflection** **[NEW]**: use the `inflect:` attribute with term-of-address intent to produce gender-inclusive greetings (e.g., "Bienvenida"/"Bienvenido" in Spanish) based on the user's device language settings.
- **New formatters**: use `Array.formatted(.list(type:))` and `Text("\(price, format: .currency(code:))")` for locale-aware data formatting inline in SwiftUI.

## APIs & Frameworks

- `Foundation` framework
- `String(localized:comment:)` **[NEW]** — Swift-native localized string (replaces `NSLocalizedString`)
- `String(localized:tableName:bundle:comment:)` **[NEW]** — with table and bundle routing
- `NSLocalizedString(_:tableName:bundle:value:comment:)` — Objective-C / older Swift API
- `Text(_:comment:)` (SwiftUI) — `LocalizedStringKey` with comment
- `Text(verbatim:)` (SwiftUI) — non-localized literal string
- `Button(_:action:)` with `LocalizedStringKey` — auto-localizable
- `Label(_:systemImage:)` — auto-localizable label
- `AttributedString(localized:comment:)` **[NEW]** — localized attributed string with Markdown
- `AttributedString` Markdown `inflect:` attribute **[NEW]** — automatic grammar agreement
- `Bundle.preferredLocalizations(from:)` — determine best server language
- `Bundle(for:)` — access a framework's or extension's bundle
- Xcode **Export Localizations** (Product menu) — generates `.xcloc` catalogs
- Xcode **Import Localizations** (Product menu) — applies translated catalogs
- `xcodebuild -exportLocalizations` / `-importLocalizations` **[NEW workspace support]**
- Xcode **Localization Catalog editor** **[NEW in Xcode 13]** — in-IDE string review and translation
- **Localized String Macro Names** build setting — register custom extraction wrappers
- `.stringsdict` file — plural rules via `NSStringPluralRuleType`
- `NSStringPluralRuleType` plist keys — `NSStringFormatSpecTypeKey`, `one`, `other`, `few`, `many`, `zero`
- `Localizable.strings` — default string table file
- `fr.lproj/` / `Base.lproj/` — language and base resource directories
- `FormatStyle` / `.currency(code:)` — `Text` inline currency formatting **[NEW]**
- `ListFormatStyle` / `Array.formatted(.list(type:))` — locale-aware list formatting **[NEW]**

## Code Highlights

Swift-native localized string with variable and comment:
```swift
// SwiftUI
Text("Order \(count) Tickets",
     comment: "Button: confirms booking of the specified number of concert tickets")

// Swift / UIKit
button.title = String(localized: "Order \(count) Tickets",
                      comment: "Button: confirms booking of the specified number of concert tickets")
```

String from a specific table and bundle (framework example):
```swift
// Inside TicketKit framework
String(localized: "Complete",
       bundle: Bundle(for: AnyClassInTicketKit.self),
       comment: "Standalone ticket status: order finalized")
```

Localized attributed string with Markdown bold:
```swift
AttributedString(localized: "Your order is **complete**!",
                 comment: "Ticket order confirmation title")
```

Automatic grammar agreement (plural inflection):
```swift
AttributedString(localized: "Order ^[\(ticketsCount) Ticket](inflect: true)")
```

CI export/import via command line:
```swift
xcodebuild -exportLocalizations -workspace VacationPlanet.xcworkspace -localizationPath ~/Documents
xcodebuild -importLocalizations -workspace VacationPlanet.xcworkspace -localizationPath ~/Documents/de.xcloc
```

## Takeaways
- Adopt `String(localized:comment:)` and `AttributedString(localized:comment:)` for all new Swift code — they handle bidirectionality, locale-specific digits, and Markdown formatting that `NSLocalizedString` and `String(format:)` do not.
- Always write a descriptive `comment:` — it is the only context a translator has; omitting it leads to incorrect translations.
- Let Xcode's Export/Import Localizations handle `.strings` file generation and updates — never edit them manually, and use `xcodebuild` commands to automate this on CI.
- Use `inflect: true` in `AttributedString` for automatic grammar agreement rather than maintaining separate plural string variants per language.

---
_Source: WWDC21 Session 10221 page (abstract, chapter summaries, code samples, and resource links)._
