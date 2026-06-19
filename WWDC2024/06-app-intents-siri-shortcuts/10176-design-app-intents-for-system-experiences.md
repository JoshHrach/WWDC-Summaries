# Design App Intents for System Experiences
**WWDC24 · Session 10176** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10176/)

_Platforms:_ iOS 18, iPadOS 18, macOS 15, watchOS 11

## Overview
App Intents in iOS 18 expands far beyond Siri and Shortcuts into a broader set of system integration points — Spotlight, the Action Button, Control Center, the Lock Screen, and Apple Intelligence. This session is a design-focused guide to structuring app intents so they work well in all of these contexts simultaneously, with emphasis on discoverability, disambiguation, and graceful behavior when the app is not running in the foreground.

The talk introduces the concept of an intent's "personality" — how its title, description, parameter labels, and result presentation make it legible to users across Siri, Shortcuts automation, Spotlight search, and widget configuration — and shows how small wording changes have outsized impact on adoption.

## Key Topics
- **Intent discoverability** — Spotlight and Apple Intelligence surface app intents automatically; a well-named intent with a clear `title` and `description` appears in search results and Siri suggestions without extra work.
- **`ControlWidget` and Control Center** — new in iOS 18, intents can back `ControlWidget` controls that appear in Control Center and the Lock Screen; requires an intent that runs without a `foregroundApplication` requirement.
- **`AppShortcutsProvider`** — the mechanism for surfacing curated shortcuts in Siri Suggestions and the Shortcuts app; each `AppShortcut` wraps an intent with natural-language phrases.
- **Parameter design** — parameters should be narrow and well-typed (use `@Parameter` with a custom `AppEntity` type rather than `String`); disambiguation dialogs must be concise for Siri.
- **Result presentation** — `ReturnsValue` / `OpensIntent` / `showsNotificationOnCompletion` — choose the right result strategy for the surface (Siri reads the result aloud; Spotlight shows a snippet; widgets update silently).
- **Background execution** — intents running from Control Center or Shortcuts automation must not require a live UI session; audit `openAppWhenRun` usage and remove it for automation-safe intents.

## APIs & Frameworks

**App Intents**
- `AppIntent` protocol — core conformance; `func perform() async throws -> some IntentResult`
- `@Parameter` property wrapper — declare intent parameters; specify `title`, `description`, `requestValueDialog`, `optionsProvider`
- `IntentResult` — return type from `perform()`; compose with `ReturnsValue`, `OpensIntent`, `showsNotificationOnCompletion`
- **[NEW]** `ControlWidget` / `ControlWidgetButton` / `ControlWidgetToggle` — new widget kinds backed by `AppIntent` that appear in Control Center and the Lock Screen
- **[NEW]** `ControlCenter` integration — `ControlConfigurationIntent` protocol for intents that configure a Control Center control
- `AppShortcutsProvider` — declare `appShortcuts: [AppShortcut]`; each `AppShortcut(intent:phrases:shortTitle:systemImageName:)` surfaces in Siri and Shortcuts
- `AppShortcut` — `init(intent:phrases:shortTitle:systemImageName:)`; `phrases` drive Siri voice recognition
- `AppEntity` protocol — model objects exposed to App Intents; define `query` conformance for entity resolution
- `EntityQuery` — resolves entity identifiers; `func entities(for:)` and `func suggestedEntities()`
- `IntentParameter` disambiguation — `requestDisambiguation(among:dialog:)` within `perform()` for run-time disambiguation
- `ForegroundContinuableIntent` — intent that can start in the background and ask to continue in the foreground if needed
- `openAppWhenRun` — `Bool` property on `AppIntent`; set to `false` for intents that must run from Control Center or automation
- `IntentDonation` — `AppIntent.donate()` — donate intent executions for Siri Suggestions
- **[NEW]** `AssistantSchema` — opt in to Apple Intelligence action descriptions; annotate intents with `.perform` role descriptions for Siri and Writing Tools integration
- `ShortcutsLink` — SwiftUI view linking to the intent in the Shortcuts app

## Code Highlights
A Control Center toggle backed by an App Intent:

```swift
struct ToggleFocusModeIntent: AppIntent {
    static var title: LocalizedStringResource = "Toggle Focus Mode"
    static var openAppWhenRun: Bool = false  // must be false for Control Center

    func perform() async throws -> some IntentResult {
        FocusManager.toggle()
        return .result()
    }
}

struct FocusModeControl: ControlWidget {
    var body: some ControlWidgetConfiguration {
        StaticControlConfiguration(kind: "com.app.focus", provider: FocusProvider()) { value in
            ControlWidgetToggle("Focus", isOn: value, action: ToggleFocusModeIntent())
        }
    }
}
```

## Takeaways
- Remove `openAppWhenRun = true` from intents used in Control Center, Shortcuts automation, and Siri; those surfaces cannot open the app — the intent must run entirely in the background process.
- Write `AppShortcut` phrases that match how users would naturally voice a request to Siri, including parameter placeholder syntax like `"Open \(\.$note) in \(.applicationName)"`.
- Use narrow `AppEntity` parameters rather than `String` — entity resolution gives Siri and Shortcuts meaningful disambiguation dialogs and improves intent matching accuracy.
- Annotate key intents with `AssistantSchema` descriptions to make them eligible for Apple Intelligence action inference in iOS 18.

---
_Source: WWDC24 Session 10176 page (abstract, chapter summaries, code samples, and resource links)._
