# Implement App Shortcuts with App Intents
**WWDC22 · Session 10170** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10170/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9

## Overview
App Shortcuts is a new capability built on the App Intents framework that allows developers to expose app functionality to Siri, Spotlight, and the Shortcuts app with zero user setup. Unlike the previous SiriKit intents model, App Shortcuts are available as soon as the app is installed — no "Add to Siri" button or manual configuration by the user is required.

The App Intents framework is Swift-only and defines intents entirely in source code rather than in a separate metadata file. Intents are Swift structs conforming to `AppIntent`, and shortcut phrases are registered through an `AppShortcutsProvider`. This approach integrates cleanly with code review and merge workflows and eliminates code generation steps.

The session walks through building a Meditation app that uses App Shortcuts to start meditation sessions via Siri voice phrases, Spotlight search, and the Shortcuts app, including adding parameterized phrases so users can specify a session type in their initial utterance.

## Key Topics
- **App Shortcuts with zero user setup** — intents become available immediately after app install via `AppShortcutsProvider`
- **Defining App Intents** — `AppIntent` protocol with `title` and `perform()` async method; returning `IntentResult` with optional dialog
- **Custom snippet views** — returning SwiftUI views alongside dialog from `perform()` using `ShowsSnippetView`; same constraints as widgets (no interactivity or animations)
- **App Entities and parameters** — `AppEntity` protocol for custom parameter types; `EntityQuery` for lookup by identifier; `@Parameter` property wrapper on intent structs
- **Value prompts** — `$parameter.requestDisambiguation(among:dialog:)`, value prompts, and confirmation prompts during `perform()`
- **Parameterized shortcut phrases** — `suggestedEntities()` on `EntityQuery`; `AppShortcutsProvider.updateAppShortcutParameters()` to notify the system when entities change
- **Discoverability** — `SiriTipView` (SwiftUI and UIKit), `ShortcutsLink`, Spotlight integration, phrasing best practices, app name synonyms
- **Phrase design** — including `.applicationName` token; maximum of 10 app shortcuts per app; first phrase is the primary/label phrase

## APIs & Frameworks
**App Intents (new framework)** **[NEW]**
- `AppIntent` protocol **[NEW]** — base protocol for all intents; requires `title: LocalizedStringResource` and `perform() async throws -> some IntentResult`
- `AppShortcutsProvider` protocol **[NEW]** — provides `appShortcuts: [AppShortcut]` static getter
- `AppShortcut` **[NEW]** — wraps an intent with an array of spoken `phrases` and metadata
- `AppShortcut.phrases` **[NEW]** — array of `LocalizedStringResource` with `.applicationName` and `\(\.$parameter)` interpolation tokens
- `.applicationName` phrase token **[NEW]** — inserts app name and configured synonyms into a phrase
- `AppShortcutsProvider.updateAppShortcutParameters()` **[NEW]** — notifies the system to refresh parameterized shortcut phrases
- `IntentResult` protocol **[NEW]** — return type from `perform()`
- `ProvidesDialog` protocol **[NEW]** — result conforms to this to return spoken/displayed dialog
- `ShowsSnippetView` protocol **[NEW]** — result conforms to this to return a custom SwiftUI snippet view
- `.result(dialog:)` **[NEW]** — factory on `IntentResult` to return a dialog string
- `.result(dialog:view:)` **[NEW]** — factory on `IntentResult` to return dialog plus a SwiftUI view
- `IntentDialog` **[NEW]** — typed wrapper for dialog strings used in prompts
- `AppEntity` protocol **[NEW]** — marks a Swift type as usable as an intent parameter; requires `id`, `typeDisplayName`, `displayRepresentation`, and `defaultQuery`
- `DisplayRepresentation` **[NEW]** — describes how an entity is shown in UI
- `EntityQuery` protocol **[NEW]** — requires `entities(for identifiers:) async throws -> [Entity]`; optional `suggestedEntities() async throws -> [Entity]`
- `@Parameter` property wrapper **[NEW]** — declares a property on an `AppIntent` struct as a parameter; accepts `title:` metadata
- `$parameter.requestDisambiguation(among:dialog:)` **[NEW]** — async prompt that asks the user to pick from a list of entities
- `SiriTipView` **[NEW]** — SwiftUI view for surfacing App Shortcut discoverability (replaces Add to Siri button)
- `ShortcutsLink` **[NEW]** — SwiftUI view that links to the app's shortcuts list in the Shortcuts app

**SwiftUI** (used for snippet views and tip views)
- Standard SwiftUI view composition for snippet views (widget-like constraints apply)

## Code Highlights
Minimal App Intent:
```swift
struct StartMeditationIntent: AppIntent {
    static let title: LocalizedStringResource = "Start Meditation Session"

    func perform() async throws -> some IntentResult & ProvidesDialog {
        await MeditationService.startDefaultSession()
        return .result(dialog: "Okay, starting a meditation session.")
    }
}
```

App Shortcuts provider with multiple phrases and parameterized phrase:
```swift
struct MeditationShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: StartMeditationIntent(),
            phrases: [
                "Start a \(.applicationName)",
                "Begin \(.applicationName)",
                "Start a \(\.$session) session with \(.applicationName)"
            ]
        )
    }
}
```

Returning a custom snippet view:
```swift
return .result(
    dialog: "Okay, starting a meditation session.",
    view: MeditationSnippetView()
)
```

Prompting for a parameter via disambiguation:
```swift
let sessionToRun = self.session ?? try await $session.requestDisambiguation(
    among: SessionManager.allSessions,
    dialog: IntentDialog("What session would you like?")
)
```

Notifying the system when entity list changes:
```swift
self.cancellable = $sessions.sink { _ in
    MeditationShortcuts.updateAppShortcutParameters()
}
```

## Takeaways
- App Shortcuts require zero user setup — they're available immediately after app install, surfaced in Siri, Spotlight, and the Shortcuts app.
- The App Intents framework is Swift-only and code-centric, replacing SiriKit's metadata-file approach with plain Swift structs and protocols.
- Parameterized phrases allow users to specify parameter values inline (e.g., "Start a calming meditation"), but only for predefined entity sets — not open-ended strings.
- Discoverability is the developer's responsibility: use `SiriTipView` contextually and `ShortcutsLink` to help users discover and engage with App Shortcuts.

---
_Source: WWDC22 Session 10170 page (abstract, chapter summaries, code samples, and resource links)._
