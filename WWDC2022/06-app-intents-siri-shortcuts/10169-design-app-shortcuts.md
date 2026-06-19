# Design App Shortcuts
**WWDC22 · Session 10169** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10169/)

_Platforms:_ iOS 16, iPadOS 16, watchOS 9, macOS Ventura 13

## Overview
This design-focused session introduces App Shortcuts — a new App Intents capability in iOS 16 where developers define shortcuts that are **automatically available** in Siri, Spotlight, and the Shortcuts app as soon as the app is installed, with no user setup required. The session covers the complete design process: selecting the right features, naming shortcuts, visual presentation (Custom Snippets and Live Activities), collecting required input, and surfacing shortcuts for discoverability.

The session pairs with the implementation session "Implement App Shortcuts with App Intents" (10170) and the framework session "Dive into App Intents" (10032).

## Key Topics

### Selecting the Right Features
Not every app feature is a good App Shortcut candidate. The best candidates are:
- **Self-contained** — can be completed entirely outside the app without launching it into the foreground
- **Straightforward** — efficient and uncomplicated; tasks requiring many inputs or lengthy multi-step flows are poor candidates

Limit: maximum 10 App Shortcuts per app; in practice, 2–5 high-quality shortcuts is better than 10 mediocre ones. Focus on tasks people can quickly learn, remember, and grow to depend on.

### Naming App Shortcuts
The shortcut name is a **hero phrase** — it appears in the Shortcuts app, in Spotlight, and is exactly what users say to Siri.

Rules:
- **Include the app name** — required; can use the official App Store name or any submitted alternative name (e.g., "Panera" as a synonym for "Panera Bread")
- **Keep it brief** — easy to remember and say; test by saying it aloud
- **Integrate the app name naturally** — e.g., "Record Voice Memo" not "Record a Memo with Voice Memos"
- **Provide thorough synonyms** — every phrase a user might naturally say that means the same thing must be explicitly declared; Siri will only respond to declared phrases
- **Translate synonyms** for every supported App Store language

### Dynamic Parameters in Shortcut Names
A shortcut name can include one dynamic parameter to create multiple phrase variants:
- Provides flexibility: "Start Sleep Meditation", "Start Gratitude Meditation", etc.
- The parameter draws from a **finite, predictable list** (known categories, recent items, rooms); never from infinite values (times, numbers)
- The list updates any time the app is open, keeping it current
- Each parameter value + AppIntent = one unique App Shortcut variant, auto-generated in Shortcuts app and Spotlight
- Only one dynamic parameter per phrase; if the phrase sounds like it has two parameters, simplify it

### Visual Design — Custom Snippets and Live Activities
Two result presentation options in iOS 16:
- **Live Activity** — use when the content is time-continuous and benefits from persistent display on Lock Screen (order tracking, countdown timers)
- **Custom Snippet** — use for self-contained results and confirmations

Visual guidelines for Custom Snippets:
- Use a **translucent/semitranslucent material** for the background — do not fill with an opaque color
- Use **vibrant label colors** for text to guarantee contrast over translucent backgrounds and automatic Dark Mode support
- Suppress **supporting dialog** in source code when it is fully redundant with the custom visual
- **Full dialog** (covering all information) must still be provided for voice-only contexts (AirPods)
- **watchOS 9 now supports Custom Snippets** — test and adapt layout for the smaller screen (e.g., reorder elements to improve readability)

### Spotlight Integration
App Shortcuts surface in Spotlight automatically in iOS 16:
- The **first shortcut** in the App Shortcuts array appears as a Siri Suggestion under the app in search results
- Any shortcut matching the search term appears when users search by shortcut name
- When the app is a Siri Suggestion on the empty Spotlight screen, the top shortcut appears proactively
- Each shortcut shows an **SF Symbol** on the right — choose a symbol that accurately represents the shortcut's intent
- **Order matters** for which shortcut appears first; actions order is set at build time; parameter order is dynamic (updated when app is open) — use recency or frequency as the ordering heuristic

### Collecting Required Input
Three patterns for requesting missing information, in order of preference:

1. **Parameter Confirmation** — infer a likely value (from prior selection or popularity) and present it for quick confirmation; most efficient
2. **Disambiguation** — short list of options (5 or fewer); read aloud in voice-only contexts ("What kind of meditation? Focus, Gratitude, Walking, Compassion, or Sleep?"); avoid for large option sets
3. **Open-ended request** — for values with no finite list (numbers, locations, strings); use App Intents built-in types (numerical values, dates, time values) when possible for built-in dialog patterns and NLU; use custom entities otherwise

**Intent Confirmation** (final step): confirm the entire action before executing. Use only for consequential, destructive, financial, or high-risk actions. Confirmation buttons must contain a verb reiterating the action (e.g., "Delete", "Send") not ambiguous words like "Confirm". App Intents provides standard default verbs with synonyms; use a custom string + synonyms when none match.

### Discoverability
Two surfaces for teaching users about App Shortcuts:
- **In-app tip** — replaces the old "Add to Siri" button; surface at moments when users have just completed or are about to complete a repeatable action; make it dismissible
- **Shortcuts app link** — a provided button that links directly to the app's shortcuts landing page in the Shortcuts app

## APIs & Frameworks

### App Intents (New in iOS 16)
- `AppShortcutsProvider` protocol **[NEW]** — provides the `appShortcuts` array; replaces manual "Add to Siri" flows; shortcuts become available automatically on app install
- `AppShortcut(intent:phrases:shortTitle:systemImageName:)` **[NEW]** — defines a single App Shortcut with its invocation phrases and Spotlight icon
- `AppIntent` protocol **[NEW]** — the action type underlying each shortcut
- Dynamic parameter in phrase: `"\(.applicationName) Start \(\.$meditationType)"` syntax embeds an entity parameter in the invocation phrase
- `AppEntity` / `EntityQuery` **[NEW]** — provides the finite list of values for dynamic parameters; query updates dynamically when app is open

### Snippet and Dialog APIs
- `IntentResultValue.dialog(_:)` / `IntentResultValue.snippet(_:)` **[NEW]** — return dialog and Custom Snippet views as shortcut results
- `SiriTipView` **[NEW]** — in-app tip UI component to promote a shortcut to users; replaces `INUIAddVoiceShortcutButton`
- `ShortcutsLink` **[NEW]** — button that links to the app's page in the Shortcuts app

### Parameter Confirmation and Intent Confirmation
- `needsValueConfirmation` on `IntentParameter` — triggers Parameter Confirmation flow
- `requestConfirmation(result:)` on `AppIntent.perform()` — triggers Intent Confirmation flow
- `IntentDialog` — text spoken by Siri and shown in snippet for confirmations

### Existing Related APIs
- `INUIAddVoiceShortcutButton` — deprecated; replaced by `SiriTipView` and automatic App Shortcuts
- `AppShortcutsProvider.updateAppShortcutParameters()` — call when dynamic parameter values change to update Spotlight suggestions

## Code Highlights
No code samples in this session (design-focused). See companion sessions for implementation:
- "Implement App Shortcuts with App Intents" (10170)
- "Dive into App Intents" (10032)

App Shortcuts definition pattern:
```swift
struct MeditationShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: StartMeditationIntent(),
            phrases: [
                "Start \(\.$meditationType) with \(.applicationName)",
                "Begin \(\.$meditationType) \(.applicationName)",
            ],
            shortTitle: "Start Meditation",
            systemImageName: "figure.mind.and.body"
        )
    }
}
```

Intent Confirmation with action verb:
```swift
await requestConfirmation(
    result: .result(value: session),
    dialog: "Are you ready to start \(session.type) meditation?"
)
// Confirmation button shows "Start" (built-in App Intents verb)
```

## Takeaways
- App Shortcuts are automatically available in Siri, Spotlight, and the Shortcuts app the moment the app is installed — no user setup required.
- Focus on 2–5 self-contained, straightforward features; the phrase name must include the app name, be brief, and have thorough synonyms covering all natural language variations.
- A single dynamic parameter in the phrase creates multiple discoverable shortcut variants from a finite, predictable list; the list updates dynamically with app open.
- Custom Snippets use translucent backgrounds with vibrant label colors; suppress redundant supporting dialog; provide full dialog for voice-only contexts; test on watchOS 9.
- Collect missing information using Parameter Confirmation, Disambiguation (≤5 options), or Open-ended Requests in that order of preference; reserve Intent Confirmation for consequential or destructive actions only.

---
_Source: WWDC22 Session 10169 page (abstract, transcript, and resource links)._
