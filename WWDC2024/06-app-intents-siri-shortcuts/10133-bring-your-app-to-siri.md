# Bring Your App to Siri
**WWDC24 · Session 10133** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10133/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia (Apple Intelligence and Siri; requires A17 Pro or M-series for Apple Intelligence features)

## Overview
This session explains how to integrate your app's capabilities into Siri and Apple Intelligence using the App Intents framework in iOS 18. Apple Intelligence supercharges Siri with on-device understanding, enabling natural language requests that go far beyond the rigid phrase matching of previous Siri integrations. The session covers what's new with Siri in iOS 18, the role of App Intents as the integration layer, and specifically the new **Assistant Schemas**—a set of predefined, structured intent templates that Siri and Apple Intelligence understand out of the box.

The session demonstrates adopting App Shortcuts with Assistant Schemas across several domains (photos, mail, messaging, notes) and explains how on-device semantic search via `IndexedEntity` allows Siri to find and operate on your app's content without server round-trips.

## Key Topics
- **Apple Intelligence + Siri** — on-device language model enables natural, multi-step requests without explicit phrase registration
- **App Intents as the integration layer** — `AppIntent`, `AppEntity`, `EntityQuery` are unchanged core building blocks; Assistant Schemas build on top
- **[NEW] Assistant Schemas** — predefined intent schemas for common domain actions; annotate your `AppIntent` with `@AssistantIntent(schema: .)` to register with Apple Intelligence
- **Supported schema domains** — `.photos`, `.mail`, `.messaging`, `.notes`, `.journaling`, `.files`, `.web`, `.spreadsheets`, `.presentations`, `.documents`, `.tasks`, `.alarms`, `.reminders`, `.calendar`
- **[NEW] `IndexedEntity`** — conforms your `AppEntity` to on-device semantic search index; Siri can find entities by natural language description without a network call
- **App Shortcuts** — still the entry point for surfaces like Spotlight, Siri suggestions, and the Shortcuts app; now elevated by Apple Intelligence understanding
- **Flexible matching** — Siri uses the semantic meaning of the request, not keyword matching; reduces the need to enumerate every phrase variant

## APIs & Frameworks
### App Intents
- `AppIntent` — core protocol; `func perform() async throws -> some IntentResult`
- `AppEntity` — protocol for structured app data types exposed to Siri
- `EntityQuery` — protocol for fetching entities by ID or by property predicates
- `AppShortcut` — registers a phrase trigger + intent for Siri/Spotlight surfaces
- **[NEW] `@AssistantIntent(schema:)`** — macro to associate an `AppIntent` with a predefined Apple Intelligence schema
  - Example schemas: `.photos.search`, `.mail.sendMessage`, `.messaging.sendMessage`, `.notes.createNote`, `.tasks.createTask`
- **[NEW] `IndexedEntity`** — protocol; conforming `AppEntity` types are indexed on-device for semantic search; `var indexedContent: IndexedEntityContent { get }`
- `StringQuery` — query type for text-based entity search; used with `IndexedEntity`
- `EntityStringQuery` — legacy; still supported
- `IntentParameter` — parameter declaration on intents; `@Parameter` property wrapper
- `ResolverResult` — resolution outcome for parameters
- `AppShortcutsProvider` — static collection of `AppShortcut` instances

### SiriKit (legacy, for reference)
- Previous `INIntent` / `INExtension` model is superseded by App Intents for new development; existing SiriKit extensions continue to work

## Code Highlights
```swift
import AppIntents

// Adopt an Assistant Schema for sending a message
@AssistantIntent(schema: .messaging.sendMessage)
struct SendMessageIntent: AppIntent {
    static var title: LocalizedStringResource = "Send Message"

    @Parameter(title: "Recipients") var recipients: [ContactEntity]
    @Parameter(title: "Content") var content: String

    func perform() async throws -> some IntentResult {
        try await MessageService.send(content, to: recipients)
        return .result()
    }
}

// IndexedEntity — make your content findable by Siri semantically
struct NoteEntity: AppEntity, IndexedEntity {
    static var typeDisplayRepresentation = TypeDisplayRepresentation(name: "Note")
    var id: UUID
    var title: String
    var body: String

    var indexedContent: IndexedEntityContent {
        .init(title: title, body: body)
    }
}

// App Shortcut registration
struct MyAppShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: SendMessageIntent(),
            phrases: ["Send a message with \(.applicationName)"],
            shortTitle: "Send Message",
            systemImageName: "message"
        )
    }
}
```

## Takeaways
- Assistant Schemas are the primary new integration point for WWDC24: annotating your intents with `@AssistantIntent(schema:)` maps them to Apple Intelligence's understanding without writing custom NLU
- `IndexedEntity` enables Siri to find your app's content semantically on-device—adopt it for any entity type users might reference by description in a voice or text request
- You do not need to pre-register every phrase variant; Apple Intelligence infers intent from natural language, so focus on correct schema adoption and well-typed parameters
- App Intents is additive: existing `AppIntent`/`AppShortcut` code continues to work; Assistant Schemas are an annotation layer on top

---
_Source: WWDC24 Session 10133 page (abstract, chapter summaries, code samples, and resource links)._
