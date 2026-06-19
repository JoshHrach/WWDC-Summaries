# Discover String Catalogs
**WWDC23 · Session 10155** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10155/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1 (deployable to earlier OS versions at build time)

## Overview
Marina and Matt from Apple's Localization team introduce String Catalogs (`*.xcstrings`) — the new Xcode 15 file format that consolidates an entire string table (previously scattered across multiple `.strings` and `.stringsdict` files inside per-language `.lproj` directories) into a single JSON-backed file. Xcode 15 automatically extracts localizable strings from Swift, Objective-C, C, SwiftUI, Interface Builder, and Info.plist sources into the catalog on every build, keeping the catalog in sync with source code without manual intervention.

String Catalogs are backward-compatible: they compile to `.strings` and `.stringsdict` at build time, so no deployment target bump is required. Existing projects can migrate at their own pace, file by file.

## Key Topics

### Extraction and Sync
Xcode 15 automatically discovers localizable strings from:
- **SwiftUI**: any string literal passed to a parameter of type `LocalizedStringKey`.
- **Swift**: `String(localized:)`, `AttributedString(localized:)`, `LocalizedStringResource` initializers. Requires the **Use Compiler to Extract Swift Strings** build setting enabled.
- **Objective-C**: `NSLocalizedString` and any custom macros declared in the **Localized String Macro Names** build setting.
- **C**: `CFCopyLocalizedString` and custom macros.
- **Interface Builder**: string-valued properties in Storyboard/xib files.
- **Info.plist**: known localizable keys, via a dedicated `InfoPlist.xcstrings` file.
- **App Shortcut phrases**: `AppShortcutsProvider` phrases (see "Spotlight your app with App Shortcuts").

On each build Xcode adds new strings, updates changed source values, and marks removed strings as **Stale** (if already translated) or deletes them (if not yet translated).

### Translation States
Each string in each language has one of four states:
- **New** — string added but not yet translated.
- **Needs Review** — source string changed after translation existed; translator attention required.
- **Stale** — string no longer found in code but has existing translations; confirm deletion or keep manually.
- **Translated** (green checkmark) — fully localized, no action needed.

The sidebar shows per-language progress percentages, providing a first-class way to verify 100% localization before App Store submission.

### String Variations (Plurals and Device)
The editor supports built-in string variation workflows:
- **Vary by plural**: right-click → "Vary by plural." Xcode generates the correct plural form keys for each language (e.g., one/other for English; zero/one/two/few/many/other for Ukrainian). No stringsdict required.
- **Vary by device**: right-click → "Vary by device." Choose device types (applewatch, mac, etc.) and provide device-specific translations.
- **Substitutions**: for strings with multiple interpolated arguments each requiring independent plural agreement, substitutions (prefixed with `@`) let each argument vary independently. Xcode generates the correct permutations at runtime.

### Export and Import (XLIFF)
- **Product → Export Localizations** generates one Localization Catalog per language containing an XLIFF file.
- String Catalog variations use a new human-readable XLIFF key format: `key|==|plural.one`, `key|==|device.applewatch`, `key|==|substitution.birds.one`, etc. (previously, stringsdict keys were opaque plist paths).
- Translators can add variation structure directly in the XLIFF; importing will update the catalog accordingly.
- Set **Localization Prefers String Catalogs** build setting to **Yes** to ensure XLIFF export uses the new format.

### Build Process
String Catalogs are JSON files in the project. At build time, Xcode compiles them to `.strings` / `.stringsdict` binaries. Source strings are not included in the final build (disk space savings). No deployment target changes needed.

### Migration from Legacy Formats
Right-click any `.strings` or `.stringsdict` file → **Migrate to String Catalog**. The Migration Assistant lists all migratable files per target. Xcode builds the project post-migration to populate the catalog. Legacy formats can coexist with String Catalogs during gradual migration.

## APIs & Frameworks

### Xcode 15 String Catalog Format **[NEW]**
- `.xcstrings` file — new String Catalog file format (JSON under the hood) **[NEW]**
- `Localizable.xcstrings` — default table name (matches `.strings` convention) **[NEW]**
- `InfoPlist.xcstrings` — dedicated catalog for Info.plist localizable keys **[NEW]**
- String variation editor — built-in plural/device variation UI in Xcode **[NEW]**
- Substitutions — multi-argument plural variation system **[NEW]**
- Per-language progress indicator — percentage display and checkmark in sidebar **[NEW]**
- Localization Migration Assistant — right-click context menu migration tool **[NEW]**
- **Localization Prefers String Catalogs** build setting — controls XLIFF export format **[NEW]**
- **Use Compiler to Extract Swift Strings** build setting — required for Swift string extraction

### Foundation — Localizable String Types
- `LocalizedStringResource` — recommended type for passing localizable strings through code; supports `comment`, `table`, `defaultValue` **[NEW in earlier release, now central to String Catalogs]**
- `String(localized:comment:table:bundle:)` — initializer for localized strings in Swift
- `AttributedString(localized:)` — localized attributed string initializer
- `LocalizedStringKey` — SwiftUI type; any string literal in a SwiftUI view is `LocalizedStringKey`

### SwiftUI
- `Text(_ key: LocalizedStringKey, comment:)` — localizable text view with optional comment
- `Label(_ titleKey: LocalizedStringKey, systemImage:)` — localizable label
- `Button(_ titleKey: LocalizedStringKey)` — localizable button
- Custom views using `LocalizedStringResource` properties — Xcode extracts strings at call sites

### Legacy (for reference / migration)
- `.strings` files — replaced by `.xcstrings`
- `.stringsdict` files — replaced by plural variation in `.xcstrings`
- `NSLocalizedString(_:comment:)` — Objective-C localized string macro
- `CFCopyLocalizedString(_:_:)` — C localized string macro

## Code Highlights

```swift
// Swift: localizable string variants
String(localized: "Welcome to WWDC!")
String(localized: "WWDC_NOTIFICATION_TITLE", defaultValue: "Welcome to WWDC!")
String(localized: "Welcome to WWDC!", comment: "Notification banner title")
String(localized: "Welcome to WWDC!", table: "WWDCNotifications", comment: "Banner title")
```

```swift
// LocalizedStringResource — preferred for passing strings before lookup
let resource = LocalizedStringResource("Title", comment: "Section header")
// Pass resource around; resolve when needed:
let resolved = String(localized: resource)
```

```swift
// SwiftUI: automatic extraction from LocalizedStringKey parameters
struct CardView: View {
    let title: LocalizedStringResource     // Xcode extracts from call-site string literal
    let subtitle: LocalizedStringResource
    var body: some View {
        VStack { Text(title); Text(subtitle) }
    }
}
// Usage — Xcode extracts "Recent Purchases" and "Items you've ordered…"
CardView(title: "Recent Purchases", subtitle: "Items you've ordered in the past week.")
```

```swift
// App Shortcut phrases — localized via String Catalog automatically
struct FoodTruckShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: ShowTopDonutsIntent(),
            phrases: [
                "\(.applicationName) Trends for \(\.$timeframe)",
                "Show trending donuts for \(\.$timeframe) in \(.applicationName)"
            ]
        )
    }
}
```

## Takeaways
- String Catalogs replace `.strings` + `.stringsdict` with a single, Xcode-managed file that auto-syncs with source on every build — no more manually hunting for missing localizations.
- Migration is opt-in and file-by-file: right-click → **Migrate to String Catalog** whenever you're ready; old formats coexist peacefully during transition.
- Use `LocalizedStringResource` as the standard type for all localizable strings passed through model/business logic code; SwiftUI's `LocalizedStringKey` handles the view layer automatically.
- The plural variation and substitution editors eliminate the need for hand-crafted stringsdict plists, with correct plural forms generated automatically per language.

---
_Source: WWDC23 Session 10155 page (abstract, chapter summaries, code samples, and resource links)._
