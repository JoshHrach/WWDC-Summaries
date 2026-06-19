# Creating Great Localized Experiences with Xcode 11
**WWDC19 · Session 403** · [Watch](https://developer.apple.com/videos/play/wwdc2019/403/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session covers the full spectrum of localization improvements in iOS 13 and Xcode 11. The biggest user-facing change is the new per-app language setting in iOS 13, which lets users independently choose the language for each installed app without changing the system language — a critical feature for multilingual markets like Hong Kong and India.

On the tooling side, Xcode 11 dramatically speeds up localization export (up to 15x faster for Interface Builder-heavy projects), adds image and symbol localization directly inside Asset Catalogs, introduces a new stringsdict rule for device-specific strings, and includes Settings Bundles in the Xcode Localization Catalog workflow. The session also demonstrates using Xcode Test Plans to auto-generate localized screenshots and embed them in the Localization Catalog for translator context.

## Key Topics

- **Per-app language setting (iOS 13)** — Users can set each app's language independently via Settings; if a paired Apple Watch is present, the setting syncs to the watch app automatically. On macOS, the equivalent is Language & Region preferences.
- **Correct API usage for language detection** — `Locale.preferredLanguages` returns all user-preferred languages regardless of bundle support; `Bundle.main.preferredLocalizations` intersects user preferences with bundle-supported languages; `Bundle.preferredLocalizations(from:)` intersects with an external (e.g., server) language list.
- **State restoration on language change** — The app relaunches when the user changes its language; adopting `NSUserActivity`-based state restoration keeps the user's context intact.
- **Localization export performance** — Redesigned string extraction makes Interface Builder export ~15x faster; `xcodebuild -exportLocalizations` / `-importLocalizations` is the recommended command-line workflow.
- **Device-specific stringsdict rule** — New `NSStringDeviceSpecificRuleType` with device keys `appletv`, `applewatch`, `ipad`, `iphone`, `ipod`, `mac`; combinable with existing plural and variable-width rules.
- **Settings Bundle localization** — Settings Bundles are now included in Xcode Localization Catalogs, making them as easy to localize as any other resource.
- **Asset Catalog image and symbol localization** — Images, watch complications, Apple TV stacks, Sprite atlases, Game Center assets, and custom symbol sets can now be localized directly in the Asset Catalog Editor via a "Localized" button in the Attribute Inspector.
- **SF Symbols localization** — System SF symbols are automatically localized; custom symbols can be given directionality (RTL flip) or full locale-specific artwork.
- **Localized screenshots via Test Plans** — Enabling "Localization Screenshots" in a Test Plan's UI testing section causes screenshots to be persisted per configuration; screenshots are auto-included in the Xcode Localization Catalog export (with `--include-screenshots`).
- **Screenshot metadata** — Each localized screenshot is accompanied by a `metadata.plist` containing string IDs and bounding-frame information, enabling localization tools to highlight strings in context.

## APIs & Frameworks

### Foundation / Locale
- `Locale.preferredLanguages` — ordered list of all user-preferred languages (ignores bundle support)
- `Bundle.main.preferredLocalizations` — languages supported by the bundle, ordered by user preference **[key distinction]**
- `Bundle.preferredLocalizations(from:)` — class method; intersects an external language list with user preferences **[useful for server-side content]**

### UIKit / State Restoration (iOS 13)
- `NSUserActivity` — new state restoration APIs in iOS 13 to restore app state after language relaunch **[NEW]**

### Xcode Localization Tooling
- Xcode Localization Catalog (`.xcloc`) — container for XLIFF strings + asset catalogs + Notes directory
- `xcodebuild -exportLocalizations` — command-line export; new `-includeScreenshots` argument **[NEW]**
- `xcodebuild -importLocalizations` — command-line import
- XLIFF (`.xliff`) — strings exchange format inside the catalog

### Stringsdict
- `NSStringDeviceSpecificRuleType` — new stringsdict rule for device-specific string variants **[NEW]**
- Device keys: `appletv`, `applewatch`, `ipad`, `iphone`, `ipod`, `mac` **[NEW]**

### Asset Catalog
- Localized image sets, watch complications, Apple TV image stacks, Sprite atlases, Game Center assets **[NEW]**
- Localized symbol sets / custom SF Symbol localization **[NEW]**
- Directionality setting for RTL symbol flipping **[NEW]**

### XCTest / Test Plans
- Test Plan (`.xctestplan`) — new file type; runs tests with multiple configurations **[NEW]**
- "Localization Screenshots" option in Test Plan UI testing section **[NEW]**
- `metadata.plist` — per-screenshot file mapping string IDs to pixel-frame locations **[NEW]**
- Accessibility Identifier (`accessibilityIdentifier`) — required for language-agnostic UI test element lookup

## Code Highlights

Correct language selection for server-side content:

```swift
// Wrong: returns user preference regardless of bundle/server support
let language = Locale.preferredLanguages.first

// Right: intersect server languages with user preference
let availableServerLocalizations = fetchServerLanguages()
let language = Bundle.preferredLocalizations(from: availableServerLocalizations).first
```

Example stringsdict entry for device-specific strings:

```xml
<key>Open in Settings</key>
<dict>
    <key>NSStringDeviceSpecificRuleType</key>
    <dict>
        <key>ipad</key>  <string>Tap in Settings</string>
        <key>mac</key>   <string>Open in Preferences</string>
        <key>iphone</key><string>Tap in Settings</string>
    </dict>
</dict>
```

## Takeaways

- Never manually override the application language in code; use `Bundle.main.preferredLocalizations` (or `Bundle.preferredLocalizations(from:)` for server content) and let the system handle relaunch.
- Adopt `NSUserActivity`-based state restoration so users return to exactly where they were after a language switch.
- Localize images, symbols, and Settings Bundles directly in the Asset Catalog and Xcode Localization Catalog — no more loose files outside the catalog.
- Invest in UI tests with Accessibility Identifiers and Test Plans to auto-generate localized screenshots that give translators the visual context they need for accurate, unambiguous translations.

---
_Source: WWDC19 Session 403 page (abstract, full transcript, code samples, and resource links)._
