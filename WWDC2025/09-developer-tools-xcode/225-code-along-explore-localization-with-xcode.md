# Code-Along: Explore Localization with Xcode
**WWDC25 · Session 225** · [Watch](https://developer.apple.com/videos/play/wwdc2025/225/)

_Platforms:_ iOS 26, macOS Tahoe 26, Xcode 26

## Overview
This code-along session teaches two complementary workflows for localizing a SwiftUI app using String Catalogs (`.xcstrings`). The first workflow — string extraction — uses Xcode's automatic extraction of `String(localized:)` and `Text()` calls into the String Catalog, with XLIFF export/import for translation. The second workflow — generated symbols — converts String Catalog keys into compile-time type-safe Swift symbols for strong guarantees and better refactoring support.

New in Xcode 26: the `#bundle` macro for correct bundle resolution in package targets, automatic comment generation powered by an on-device model, and the "Generate String Catalog Symbols" build setting with associated "Refactor > Convert Strings to Symbols" action.

## Key Topics

### String Catalog Basics
String Catalogs (`.xcstrings`) are Xcode's modern format for all localizable strings. Xcode automatically extracts `String(localized:)` and `Text("...", bundle:)` usages into the catalog when "Use Compiler to Extract Swift Strings" build setting is enabled. The catalog shows translation state per locale.

### XLIFF Export and Import
Translators work with XLIFF files. Export via **Product > Export Localizations** (generates one XLIFF per target/language). Import via **Product > Import Localizations** to merge completed translations back. The session demonstrates round-tripping strings for a French locale.

### #bundle Macro (NEW)
When using Swift Package Manager targets, bundle resolution requires passing the correct `bundle:` parameter. The **[NEW]** `#bundle` macro resolves to the current compilation target's bundle at compile time and works on older OS versions (back-deploys), replacing the verbose `Bundle.module` pattern. Used as: `String(localized: "key", bundle: #bundle)` or `Text("key", bundle: #bundle)`.

### Automatic Comment Generation (NEW)
Xcode 26 includes an on-device model that automatically generates translator comments for String Catalog entries. Comments explain the context of a string (which view it appears in, what it represents) to help translators produce accurate translations without manually reading source code.

### Generated Symbols (NEW)
The **[NEW]** "Generate String Catalog Symbols" build setting generates a Swift file with a type-safe symbol for each String Catalog key. Generated symbols are typed as `LocalizedStringResource`. This enables autocomplete, rename refactoring, and compile-time checking of string keys. Keys must be manually managed (not auto-extracted) when using symbols.

The **"Refactor > Convert Strings to Symbols"** action converts existing `String(localized:)` usages in source code to use the generated symbol API, updating both the call sites and the catalog.

### Plural Variants
String Catalog entries support "Vary by Plural" directly in the Xcode editor, generating the CLDR-based plural forms for each language without code changes.

## APIs & Frameworks

**String Catalogs / Xcode 26 Build System**
- `String(localized:tableName:bundle:comment:)` — standard localized string initializer
- `Text("...", bundle:, comment:)` — SwiftUI localized text
- **[NEW]** `#bundle` macro — compile-time current bundle reference; back-deploys
- **[NEW]** Automatic comment generation — on-device model in Xcode 26
- **[NEW]** "Generate String Catalog Symbols" build setting — produce type-safe Swift symbols
- **[NEW]** `LocalizedStringResource` — type of generated String Catalog symbols
- **[NEW]** "Refactor > Convert Strings to Symbols" — batch migrate string literals to symbols
- `.xcstrings` format — String Catalog file format
- XLIFF export: **Product > Export Localizations**
- XLIFF import: **Product > Import Localizations**
- Plural variants: "Vary by Plural" in String Catalog editor

## Code Highlights
Standard localized string with bundle:
```swift
Text("welcome_message", bundle: #bundle, comment: "Shown on app launch")
```

Using a generated symbol:
```swift
// Auto-generated symbol for key "welcome_message"
Text(.welcomeMessage)
// Or:
let message = String(localized: .welcomeMessage)
```

Converting to symbols — before:
```swift
Text("welcome_message", bundle: .main)
```
After Refactor > Convert Strings to Symbols:
```swift
Text(.welcomeMessage)
```

## Takeaways
- Use `#bundle` in any Swift package target instead of `Bundle.module` — it back-deploys and is less verbose.
- Enable automatic comment generation in Xcode 26 to save time manually annotating strings for translators.
- Choose **generated symbols** when you need compile-time key safety and refactoring support; choose **extraction** for rapid iteration where key names are managed by Xcode.
- Export XLIFF at translation milestones; import completed translations before each release rather than managing string files manually.

---
_Source: WWDC25 Session 225 page (abstract, chapter summaries, code samples, and resource links)._
