# Bring Your App's Core Features to Users with App Intents
**WWDC24 · Session 10210** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10210/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia, watchOS 11

## Overview
This session provides a comprehensive guide to building with the App Intents framework, focusing on best practices for modeling intents, entities, and queries so that your app's core features are discoverable and actionable across Siri, Spotlight, Shortcuts, and Apple Intelligence. It's the "how to design well" companion to the Siri integration session (10133).

The session emphasizes that App Intents should represent atomic, meaningful actions—not internal implementation details—and demonstrates how to choose between different entity query types, how to make intents robust by handling missing or ambiguous parameters gracefully, and how to use the new `ConfirmationActionName` and `IntentDialog` APIs for richer user feedback. The session also covers how to expose intents as Spotlight actions and use `AppShortcutsProvider` effectively.

## Key Topics
- **Intent design principles** — intents should map to user-facing actions, not internal methods; prefer narrow, focused intents over broad ones
- **Entity modeling** — `AppEntity`, `EntityProperty`, `DisplayRepresentation`; what data to expose and how
- **Query types** — `EntityQuery`, `EnumerableEntityQuery`, `UniqueAppEntityQuery`, `EntityStringQuery`; when to use each
- **Parameter resolution** — `IntentParameter`, `DynamicOptionsProvider`, `EntityPropertyQuery` for field-level filtering
- **Error handling** — `IntentError`, graceful degradation when entities are not found
- **[NEW] `ConfirmationActionName`** — customize the confirmation button label in Siri/Shortcuts UI
- **[NEW] `IntentDialog`** — richer spoken/displayed prompts for parameter request and confirmation
- **Spotlight integration** — `CSSearchableItem` + App Intents; surface intents as actions in Spotlight results
- **Widgets + App Intents** — `AppIntentTimelineProvider`, configurable widgets backed by `AppIntent` parameters

## APIs & Frameworks
### App Intents
- `AppIntent` — `title`, `description`, `perform()`; `@Parameter` for inputs
- `AppEntity` — `id`, `displayRepresentation`, `typeDisplayRepresentation`; `EntityProperty` for queryable fields
- `EntityQuery` — `entities(for:)` fetch by IDs; required on all entity types
- `EnumerableEntityQuery` — `allEntities()` for small, finite collections
- `UniqueAppEntityQuery` — single-value entity (e.g., "current user")
- `EntityPropertyQuery` — field-level filtering with `EntityQueryProperty` and `EntityQueryComparator`
- `EntityStringQuery` — free-text search over entity names
- **[NEW] `ConfirmationActionName`** — `static var confirmationActionName: ConfirmationActionName` on `AppIntent`; customizes button label (e.g., "Send", "Delete", "Book")
- **[NEW] `IntentDialog`** — `IntentDialog(stringLiteral:)` or `IntentDialog(full:supporting:)` for multi-line prompts; used in `requestConfirmation`, `requestValue`, `needsValue`
- `AppShortcut` — phrase + intent binding; `shortTitle`, `systemImageName`
- `AppShortcutsProvider` — static collection; `updateAppShortcutParameters()` to refresh dynamic phrases
- `PredictableIntent` — marks intents safe for proactive suggestion by the system
- `OpenIntent` — navigate to a specific entity/screen in the app
- `AppIntentTimelineProvider` — for widget configuration intents

### Core Spotlight
- `CSSearchableItem` with `CSSearchableItemAttributeSet` — index entities for Spotlight
- `donateInteraction(_:completionHandler:)` — donate performed intents for Siri suggestions

## Code Highlights
```swift
// Well-designed AppEntity
struct TaskEntity: AppEntity {
    static var typeDisplayRepresentation = TypeDisplayRepresentation(name: "Task")
    static var defaultQuery = TaskQuery()

    var id: UUID
    var title: String
    var isCompleted: Bool

    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: "\(title)")
    }
}

// EntityPropertyQuery for field-level filtering
struct TaskQuery: EntityPropertyQuery {
    static var properties = EntityQueryProperties<TaskEntity, TaskPredicate> {
        Property(\TaskEntity.$isCompleted) {
            EqualToComparator { TaskPredicate.completedEquals($0) }
        }
    }
    func entities(matching comparators: [TaskPredicate],
                  mode: EntityQueryMode, sortedBy: [Sort<TaskEntity>]) async throws -> [TaskEntity] {
        return try await TaskStore.shared.filter(comparators)
    }
}

// Custom confirmation button label
struct DeleteTaskIntent: AppIntent {
    static var title: LocalizedStringResource = "Delete Task"
    static var confirmationActionName = ConfirmationActionName(
        title: "Delete", confirmingTitle: "Deleting…"
    )
    @Parameter(title: "Task") var task: TaskEntity

    func perform() async throws -> some IntentResult {
        try await requestConfirmation(
            result: .result(dialog: IntentDialog("Delete \"\(task.title)\"?")))
        try await TaskStore.shared.delete(task)
        return .result()
    }
}
```

## Takeaways
- Choose the most specific query type for your entity: `UniqueAppEntityQuery` for singletons, `EnumerableEntityQuery` for small finite sets, `EntityPropertyQuery` for filterable collections—Siri and Shortcuts use these to provide smarter UI
- `ConfirmationActionName` lets you replace the generic "Confirm" button with domain-appropriate labels like "Send", "Delete", or "Book"—a small change that meaningfully improves trust in destructive or irreversible actions
- Design intents around user-facing verbs, not implementation methods; one intent per atomic user action is the right granularity
- Donate performed intents via `INInteraction` to fuel Siri Suggestions; this is separate from but complementary to `AppShortcutsProvider`

---
_Source: WWDC24 Session 10210 page (abstract, chapter summaries, code samples, and resource links)._
