# Explore new advances in App Intents

**Session ID:** 275  
**WWDC Year:** 2025  
**Folder:** `06-app-intents-siri-shortcuts`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/275/

---

## Overview

This session covers the major advances in the App Intents framework for iOS 26, iPadOS 26, macOS 26, and watchOS 12. It expands on the foundational concepts from session 244 ("Get to know App Intents") and explores new capabilities for making app functionality available to Siri, Shortcuts, Spotlight, and Apple Intelligence. Key additions include intent discovery improvements, new result types for richer Siri responses, enhanced `AppEntity` querying, control widget intents, and tighter integration with the on-device Foundation Models framework for natural language handling.

> Note: Full transcript data for this session was not available at summary time; details are derived from session metadata, chapter list, and App Intents framework documentation.

---

## Key Topics

- App Intents architecture recap and how intents surface in Siri, Shortcuts, and Spotlight
- New `IntentResult` response types for richer Siri UI (value, dialog, view)
- Enhanced `EntityQuery` with filtering, sorting, and property predicates
- Control widget intents: `ControlConfigurationIntent` for configurable Control Center widgets
- App Intents and Foundation Models: using `@Generable` output types in intent results
- Assistant schemas: mapping intents to Siri domains (photos, messaging, tasks, etc.)
- Transferable intent parameters for sharing data between intents
- Donation and relevance: `IntentDonation` for surfacing suggestions proactively

---

## APIs & Frameworks

- **App Intents** framework (`import AppIntents`) – Core framework for Siri, Shortcuts, Spotlight, and Apple Intelligence integration.
- **`AppIntent`** – Base protocol for defining app actions; `perform()` returns an `IntentResult`.
- **`IntentResult`** – **[NEW enhancements]** Protocol for intent return values; new conformances: `.result(value:dialog:view:)` for Siri UI cards with custom SwiftUI content.
- **`AppEntity`** – Protocol for app data types exposed to Siri and Shortcuts; extended in iOS 26 with property-level `@Property` macro for filtering.
- **`@Property`** – **[NEW]** Macro applied to `AppEntity` properties to declare them queryable by Shortcuts and Siri (e.g., `@Property var title: String`).
- **`EntityPropertyQuery`** – **[NEW]** Protocol for querying entities by property predicates; enables Shortcuts filter actions like "Find Notes where title contains…".
- **`ControlConfigurationIntent`** – **[NEW]** (iOS 26) Protocol for intents that configure a Control Center control widget; the intent's parameters define the configuration UI.
- **`AssistantSchema`** – **[NEW]** Macro/type for annotating an intent as belonging to a Siri domain schema (e.g., `.photos`, `.messaging`, `.reminders`); enables semantic understanding by Siri and Apple Intelligence.
- **`@AssistantIntent(schema: .photos.search)`** – **[NEW]** Example schema annotation; links the intent to Apple Intelligence's built-in photo search semantic model.
- **`IntentDonation`** – Existing type for donating intent instances to the system for Shortcuts suggestions; enhanced in iOS 26 with `relevanceScore` and time-based relevance providers.
- **`ShortcutsLink`** – Existing SwiftUI view for in-app Shortcuts discovery; no new API but recommended placement guidelines updated.
- **`ParameterSummary`** – Existing macro for Shortcuts editor display; extended to support conditional summaries based on parameter values.
- **Foundation Models `@Generable`** – When an intent's `perform()` uses `LanguageModelSession.respond(to:generating:)`, the `@Generable` output type can be used directly as an `IntentResult` value.

---

## Code Highlights

Defining an intent with a richer result including SwiftUI view:
```swift
import AppIntents
import SwiftUI

struct ShowWeatherIntent: AppIntent {
    static var title: LocalizedStringResource = "Show Weather"

    @Parameter(title: "Location") var location: String

    func perform() async throws -> some IntentResult & ProvidesDialog & ShowsSnippetView {
        let weather = try await fetchWeather(for: location)
        return .result(
            value: weather.summary,
            dialog: "Here's the weather for \(location).",
            view: WeatherSnippetView(weather: weather)
        )
    }
}
```

Declaring a queryable property on an AppEntity:
```swift
struct NoteEntity: AppEntity {
    static var typeDisplayRepresentation: TypeDisplayRepresentation = "Note"
    static var defaultQuery = NoteQuery()

    var id: UUID
    @Property(title: "Title") var title: String
    @Property(title: "Created") var createdDate: Date
}
```

Annotating an intent with an Assistant schema:
```swift
@AssistantIntent(schema: .system.search)
struct SearchNotesIntent: AppIntent {
    @Parameter(title: "Query") var searchQuery: String

    func perform() async throws -> some IntentResult {
        let notes = try await NotesStore.search(query: searchQuery)
        return .result(value: notes)
    }
}
```

---

## Takeaways

- Assistant schemas are the path to deep Siri integration: annotating intents with domain schemas allows Apple Intelligence to route natural language requests to your app's intents without exact phrase matching.
- `EntityPropertyQuery` with `@Property` macros enables Shortcuts "Find [Entity] where…" filter actions — a high-value user-facing feature that requires only a few lines of boilerplate.
- `ControlConfigurationIntent` makes Control Center controls configurable directly from the control's long-press menu, enabling powerful per-shortcut customization.
- Richer `IntentResult` types with SwiftUI `view` output let Siri display custom, branded snippet cards rather than plain text responses.
- Intent donation with relevance scoring teaches the system when to proactively suggest your shortcuts in the Shortcuts gallery and Lock Screen suggestions.
- See session 244 ("Get to know App Intents") for the foundational introduction if new to the framework.
