# Spotlight your app with App Shortcuts
**WWDC23 · Session 10102** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10102/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 9.2+, HomePod 16.2+

## Overview
App Shortcuts (introduced in iOS 16) let apps surface key features in Spotlight, Siri, and the Shortcuts app without any user setup — they work as soon as the app is installed. This session covers the full implementation pattern (App Intents + AppShortcutsProvider), major iOS 17 improvements including Flexible Matching (on-device ML for natural-language phrase recognition), the new App Shortcuts Preview tool in Xcode 15, String Catalog support for unlimited locale-specific phrases, new visual APIs (colors, entity thumbnails, short titles), and expansion to Apple Watch and HomePod.

The core message is that App Shortcuts dramatically lower friction for habitual use of an app's most important features — making them accessible hands-free, in Spotlight, and on additional devices — with minimal code.

## Key Topics

### App Shortcut Architecture
Every App Shortcut requires two Swift structures:
1. **AppIntent** — the action; `perform()` contains the actual logic.
2. **AppShortcutsProvider** — declares all shortcuts with trigger phrases, short title, and system image name. One provider per app; maximum 10 shortcuts per app, 1,000 total trigger phrase combinations.

### Entities and Queries
For shortcuts involving dynamic data (e.g., specific to-do lists), define:
- **AppEntity** — a domain object conforming to `AppEntity` with `typeDisplayRepresentation`, `displayRepresentation`, and a `defaultQuery`.
- **EntityQuery** — resolves entities by identifier (`entities(for:)`) and provides suggested values (`suggestedEntities()`).

### Parameter Phrases with Entity Expansion
Phrases can embed parameter placeholders (e.g., `"\(.list) with \(.applicationName)"`). The system calls `suggestedEntities()` to expand these into all recognized phrases. After any change to entities' `displayRepresentation`, call `AppShortcutsProvider.updateAppShortcutParameters()` to refresh the index.

### iOS 17: Flexible Matching (Semantic Similarity Index)
New on-device ML automatically recognizes phrases *similar* to the ones declared in code, without exact wording. Enabled by rebuilding with Xcode 15; opt out via "Enable App Shortcuts Flexible Matching" build setting.

### iOS 17: App Shortcuts Preview (Xcode 15)
Product > App Shortcuts Preview in Xcode 15 on macOS Sonoma. Enter arbitrary phrases to immediately test whether they match an App Shortcut — no device or Siri interaction required. Also supports locale switching for multi-language testing.

### iOS 17: String Catalog for Unlimited Locale Phrases
Previously limited to the same phrase count across all locales. With a `AppShortcuts.strings` String Catalog, each locale can have unlimited phrases. Rebuild triggers auto-population; migration assistant upgrades existing `.strings` files.

### iOS 17: Visual Identity APIs
New APIs make App Shortcuts more visually distinctive in Spotlight and Shortcuts:
- **App accent colors** — up to two colors defined in Info.plist
- **Entity thumbnail images** — added to `DisplayRepresentation` via URL, Data, named image, or SF Symbol
- **Short title + system image** — required for all App Shortcuts (was previously optional)

### iOS 17: Synonyms for AppEntities and AppEnums
`DisplayRepresentation` now accepts additional synonyms. These widen the recognized phrase variations for both direct invocation and Siri prompts. If synonyms change, call `updateAppShortcutParameters()` again.

### iOS 17: Negative Phrases
`AppShortcutsProvider.negativePhrase` array prevents specific phrases from incorrectly triggering an App Shortcut when Flexible Matching overfires.

### Apple Watch Support
App Shortcuts work on watchOS if the watchOS app is installed on the watch. Flexible Matching is not available on watchOS — phrases must match exactly. Introduced in watchOS 9.2.

### HomePod Support
App Shortcuts work on HomePod when the companion iOS/iPadOS app is installed. App cannot be launched from HomePod — dialog must be voice-complete. `IntentDialog` has a `full:supporting:` initializer where `full` is used on HomePod (voice-only) and `supporting` on visual devices. Available from HomePod 16.2+.

### OpenIntent Protocol
App intents that open the app when triggered can conform to `OpenIntent` to be eligible for Spotlight listing alongside entity search results.

## APIs & Frameworks

- **App Intents** framework (Swift only)
  - `AppIntent` protocol — defines an intent; `perform() async throws -> some IntentResult`
  - `AppShortcutsProvider` protocol — declares all App Shortcuts **[ONE per app]**
    - `var appShortcuts: [AppShortcut]` — array of shortcuts
    - `static func updateAppShortcutParameters()` — signals parameter changes to system
    - `negativePhrase: [AppShortcutPhrase]` — phrases to exclude from flexible matching **[NEW]**
  - `AppShortcut` — struct with intent, phrases, title, systemImageName
    - `AppShortcutPhrase` — phrase string with parameter interpolation e.g. `"\(.list) with \(.applicationName)"`
  - `@Parameter` property wrapper — declares intent input parameters
  - `AppEntity` protocol — domain object usable as parameter; `typeDisplayRepresentation`, `displayRepresentation`, `defaultQuery`
    - `DisplayRepresentation` — title, subtitle, image; now accepts `synonyms: [String]` **[NEW]**
    - `DisplayRepresentation.Image` — URL, Data, named image, or system image **[NEW thumbnail support]**
  - `EntityQuery` protocol — `entities(for:)`, `suggestedEntities()`
  - `AppEnum` — fixed-set enum usable as parameter
  - `IntentDialog` — dialog returned from `perform()`; `init(full:supporting:)` for HomePod vs visual device **[NEW init]**
  - `OpenIntent` protocol — conform to make open-app intents visible in Spotlight **[NEW]**
  - `SiriTipView` (SwiftUI) and UIKit equivalent — contextual tip view for in-app App Shortcut discovery
- **Spotlight** — surfaces App Shortcuts in Top Hits and title-based search results
- **Shortcuts app** — features App Shortcuts prominently; used in Automations **[NEW design in iOS 17]**
- **App Shortcuts Preview** (Xcode 15, macOS Sonoma) — phrase testing tool **[NEW]**
  - Product > App Shortcuts Preview menu item
  - Locale switching without device language change
- **String Catalog** (`AppShortcuts.strings` String Catalog format) — unlimited per-locale phrases **[NEW in iOS 17]**
  - Auto-populated from Swift source on rebuild
  - Migration assistant for existing `.strings` files
- **Semantic Similarity Index** — on-device ML powering Flexible Matching **[NEW]**
- Info.plist accent color keys — up to two colors for Spotlight/Shortcuts visual identity **[NEW]**
- **Siri** — flexible phrase recognition on iPhone/iPad; exact matching on Apple Watch
- **HomePod** — App Shortcuts via companion iOS device; voice-only, no app launch (HomePod 16.2+) **[NEW]**
- **Apple Watch / watchOS** — App Shortcuts from watchOS app only (watchOS 9.2+)

## Code Highlights

Defining an App Intent and App Shortcut:
```swift
import AppIntents

struct SummarizeListIntent: AppIntent {
    static var title: LocalizedStringResource = "Summarize List"

    @Parameter(title: "List")
    var list: ToDoList

    func perform() async throws -> some IntentResult & ReturnsValue<String> {
        let summary = await myApp.summarize(list)
        return .result(
            value: summary,
            dialog: IntentDialog(full: "Your \(list.title) has \(summary).",
                                 supporting: "Here's a summary.")
        )
    }
}

struct DemoShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: SummarizeListIntent(),
            phrases: [
                "Summarize my \(.list) list with \(.applicationName)",
                "Summarize my \(.applicationName) list"
            ],
            shortTitle: "Summarize List",
            systemImageName: "list.bullet.clipboard"
        )
    }
}
```

Conforming an entity type with a thumbnail image:
```swift
struct ToDoList: AppEntity {
    static var typeDisplayRepresentation = TypeDisplayRepresentation(name: "To-Do List")
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(
            title: "\(title)",
            image: .init(named: imageName),
            synonyms: [alternateName]
        )
    }
    static var defaultQuery = ToDoListQuery()
}
```

## Takeaways

- App Shortcuts require zero user setup and immediately appear in Spotlight and Siri after app installation; implement `AppIntent` + `AppShortcutsProvider` to expose core features.
- iOS 17 Flexible Matching (Semantic Similarity Index) dramatically improves phrase recognition naturally — no code changes needed, just rebuild with Xcode 15.
- Use the new App Shortcuts Preview tool (Product > App Shortcuts Preview in Xcode 15) to rapidly test phrase matching across locales without device or Siri interaction.
- Add colors to Info.plist, entity thumbnails to `DisplayRepresentation`, and required `shortTitle` + `systemImageName` to make shortcuts visually distinctive in Spotlight and Shortcuts.

---
_Source: WWDC23 Session 10102 page (abstract, chapter summaries, and resource links)._
