# Localize Your SwiftUI App
**WWDC21 · Session 10220** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10220/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
This session walks through the end-to-end process of localizing a SwiftUI app using the Fruta sample project. It covers how SwiftUI automatically handles localized string lookups via `LocalizedStringKey`, Markdown-based string styling, the new declarative `Foundation` formatting APIs, automatic keyboard shortcut remapping on international keyboard layouts, and an improved Xcode 13 localization workflow that uses the Swift compiler for accurate string extraction.

## Key Topics

**LocalizedStringKey and Automatic Lookup**
`Text` accepts a `LocalizedStringKey` (not a plain `String`) when initialized with a string literal, so string lookups happen automatically at runtime from the main bundle. String interpolation is supported — Xcode 13 infers format specifier types from the variable type. For disambiguation (multiple semantic meanings of the same string), pass `tableName:` to load from a separate `.strings` file. For custom views that should participate in localization, declare properties as `LocalizedStringKey` rather than `String`.

**Xcode 13 String Extraction**
Enable the build setting **Use Compiler to Extract Swift Strings** (`SWIFT_EMIT_LOC_STRINGS = YES`) to have Xcode use the Swift compiler when exporting localizations. This resolves multiline string literals correctly and picks up all `LocalizedStringKey` usages automatically. Before exporting, use the **Accented Pseudolanguage** or **Doubled-Length Pseudolanguage** in scheme options to verify all strings are marked as localizable.

**Xcode Localization Catalog (`.xcloc`)**
Xcode 13 allows double-clicking `.xcloc` catalogs in Finder to open them directly in Xcode, showing each string's key, source, translation, and comment. This makes it easy to review strings before sending to translators and to spot issues (missing formatter usage, ambiguous strings needing comments).

**Markdown String Styling**
String literals containing Markdown syntax (e.g., `*emphasis*`, `**strong**`) passed to `Text` are automatically rendered with attributed styling. Translators can rearrange or substitute Markdown emphasis markers within translations (e.g., using `**` instead of `*` in Arabic, which lacks an italics concept).

**Declarative Formatting (iOS 15 / Foundation)**
New `FormatStyle` APIs eliminate explicit formatter construction. Pass a format style directly to `Text` or as the `format:` argument to string interpolation — Xcode extracts format specifier types automatically and the formatting is locale-aware.

**Automatic Keyboard Shortcut Remapping (iOS 15 / macOS 12)**
`View.keyboardShortcut(_:)` shortcuts are automatically remapped for the user's active keyboard layout on macOS Monterey and iPadOS 15. A shortcut that requires a key combination impossible to type on a non-US keyboard (e.g., `Command +` on a Lithuanian keyboard) is remapped to an equivalent typeable combination. No developer action required.

**RTL and Layout**
SwiftUI layouts are automatically mirrored for right-to-left languages. Use `leading`/`trailing` alignment (not `left`/`right`) in `VStack` and other containers to participate in automatic RTL flipping.

## APIs & Frameworks

- **SwiftUI** — localization-aware by default
- `Text(_ key: LocalizedStringKey, tableName:, bundle:, comment:)` — string literal lookup with optional table and comment **[UPDATED]**
- `LocalizedStringKey` — type for localizable string parameters in custom views
- `Text(_ key: LocalizedStringKey, comment:)` — `comment:` parameter for translator context **[NEW]**
- `Text` Markdown support — `*italic*`, `**bold**` inline in string literals **[NEW]**
- `View.keyboardShortcut(_ shortcut: KeyboardShortcut)` — automatic remapping for international keyboards **[UPDATED]**
- **Foundation** — new `FormatStyle` family **[NEW]**
  - `Measurement.formatted(_ style:)` — e.g., `.measurement(width: .wide, usage: .food)`
  - String interpolation `\(value, format: .measurement(...))` **[NEW]**
  - `Text("Energy: \(calories, format: .measurement(width: .wide, usage: .food))")`
- **Xcode 13** — localization tooling improvements
  - Build setting `SWIFT_EMIT_LOC_STRINGS` (`Use Compiler to Extract Swift Strings`) **[NEW]**
  - `Product > Export Localizations` / `Product > Import Localizations` — existing workflow
  - `.xcloc` (Xcode Localization Catalog) — now double-click-openable in Xcode 13 **[NEW]**
  - Pseudolanguages in scheme editor: **Accented Pseudolanguage**, **Doubled-Length Pseudolanguage** — verify localizable coverage
- `.stringsdict` file — plural forms; use for all count-based strings
- `Label` with `Text` initializer — provides context comments for tab bar items and other label-based views

## Code Highlights

Localizable Text with comment:
```swift
Button(action: done) {
    Text("Done", comment: "Button title to dismiss rewards sheet")
}
```

Custom view using `LocalizedStringKey`:
```swift
struct Card: View {
    var title: LocalizedStringKey
    var subtitle: LocalizedStringKey
    var body: some View {
        VStack { Text(title); Text(subtitle) }
    }
}
// Usage — strings are extracted at export time:
Card(title: "Thank you for your order!",
     subtitle: "We will notify you when your order is ready.")
```

Markdown styling in localized string:
```swift
Text("A refreshing blend that's a *real kick*!",
     comment: "Lemonberry smoothie description")
```

Declarative measurement formatting (iOS 15):
```swift
let calories = Measurement<UnitEnergy>(value: nutritionFact.kilocalories, unit: .kilocalories)
Text(calories.formatted(.measurement(width: .wide, usage: .food)))
// In interpolation:
Text("Energy: \(calories, format: .measurement(width: .wide, usage: .food))")
```

Keyboard shortcut (auto-remapped for international keyboards):
```swift
SmoothieFavoriteButton(smoothie)
    .keyboardShortcut("+")
```

## Takeaways

- SwiftUI's `Text` and `LocalizedStringKey` handle localized string lookup automatically; use `tableName:` for disambiguation and `comment:` to provide translator context.
- Enable **Use Compiler to Extract Swift Strings** in Xcode 13 build settings for accurate, compiler-driven string extraction that handles multiline literals and custom views correctly.
- Replace `MeasurementFormatter` and similar formatter classes with the new declarative `FormatStyle` APIs — they are locale-aware, concise, and integrate directly with `Text` interpolation.
- Keyboard shortcuts defined with `.keyboardShortcut(_:)` are automatically made typeable on international keyboard layouts in macOS Monterey / iPadOS 15 — no additional work required.

---
_Source: WWDC21 Session 10220 page (abstract, full transcript, code samples, and resource links)._
