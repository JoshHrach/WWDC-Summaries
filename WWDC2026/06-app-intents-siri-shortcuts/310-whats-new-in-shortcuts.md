# What's New in Shortcuts
**WWDC26 · Session 310** · [Watch](https://developer.apple.com/videos/play/wwdc2026/310/)

_Platforms:_ iOS, iPadOS, macOS, watchOS

## Overview
This session covers three major Shortcuts enhancements in iOS 26: new automation types, the Use Model action with LLM transparency tooling, and a new Storage system for persisting and syncing data across devices. Together these features make Shortcuts more powerful for users and open new integration surfaces for app developers.

From a developer perspective, the most actionable topics are the new notification automation (which rewards well-structured notification content), the Use Model transcript inspector (which helps tune how App Entity properties appear to Apple Intelligence), and the Storage system's requirement for stable device-consistent entity identifiers — connecting directly to the new `SyncableEntity` protocol covered in Session 345.

## Key Topics

### Automations
In iOS 26, Automations are more discoverable in the Shortcuts editor. Three new automation types are introduced:
- **Screenshot** — triggers when the user takes a screenshot
- **Keyboard connection** — triggers when a hardware keyboard connects or disconnects
- **Notification** — triggers on notification delivery, with optional keyword filtering on notification content

The notification automation is the most developer-relevant: it can be scoped to specific apps and filtered by keywords in the notification body. Design notifications with clear, machine-readable content and consistent keyword patterns to make them useful automation triggers.

### Use Model
The **Use Model** action has access to the latest Apple Intelligence models with web retrieval, enabling natural language reasoning steps in Shortcuts. New in this release: the **model transcript inspector** in Xcode/Shortcuts lets developers see the exact representation passed to the model from their App Entity properties. This is the tool for tuning which `@Property` fields are included and how they read to an LLM — check the transcript to verify concise, unambiguous entity descriptions.

### Storage
Storage lets Shortcuts persist values between runs — similar to UserDefaults but synced across all devices via iCloud:
- **Get Storage Value** — retrieve a named value
- **Set Storage Value** — store a named value
- **Global storage** — shared across all shortcuts on all devices

A key use case is giving the Use Model action a "memory" that persists across Shortcut invocations. For App Intent entities referenced in storage, the identifier must be stable and device-consistent (server UUID, CloudKit record ID, or `SyncableEntityIdentifier`) — otherwise the stored entity reference won't resolve correctly on another device.

## APIs & Frameworks

### Shortcuts / Automations (system, no direct API)
- **Notification automation** — **[NEW]** triggers on notification delivery; supports keyword filtering on notification body content
- **Screenshot automation** — **[NEW]** triggers on screenshot capture
- **Keyboard connection automation** — **[NEW]** triggers on hardware keyboard connect/disconnect
- **Use Model action** — **[NEW enhanced]** Apple Intelligence LLM with web retrieval in Shortcuts
- **Model transcript inspector** — **[NEW]** developer tool showing exact data sent to the LLM from App Entity properties
- **Get Storage Value** — **[NEW]** retrieve a stored value by name
- **Set Storage Value** — **[NEW]** persist a value by name; syncs via iCloud
- **Global storage values** — **[NEW]** shared across all shortcuts and all devices

### AppIntents
- `AppEntity` protocol — `id`, `displayRepresentation`, `defaultQuery`
- `@Property` — property wrapper; properties appear in Use Model transcripts — tune for LLM clarity
- `@Property(title:)` — custom title for the property in Shortcuts UI and model transcripts
- `TypeDisplayRepresentation` — `name:`, `numericFormat:` — shown in Shortcuts and model UI
- `DisplayRepresentation` — `title:`, `subtitle:` — shown in Shortcuts picker and Use Model results
- `SyncableEntity` protocol — **[NEW]** stable cross-device entity IDs required for Storage integration (see Session 345)
- `SyncableEntityIdentifier` — **[NEW]** pairs local + stable ID for cross-device storage references

### UserNotifications (for notification automation integration)
- `UNMutableNotificationContent` — `title`, `body` — these fields are what the notification automation inspects for keyword matching
- Design `body` with clear, consistent keywords to make notification automations predictable

## Code Highlights

**Well-structured App Entity for Use Model clarity:**
```swift
struct SoupEntity: AppEntity, Identifiable {
    static var typeDisplayRepresentation = TypeDisplayRepresentation(
        name: "Soup",
        numericFormat: "\(placeholder: .int) soups"
    )
    static var defaultQuery = SoupEntityQuery()

    var id: Soup.ID

    @Property var name: String

    @Property(title: "Available Today")
    var isAvailableToday: Bool

    @Property(title: "Ingredients")
    var ingredients: String

    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: "\(name)", subtitle: SoupStore.description(for: id))
    }
}
```

_Use the model transcript inspector to verify that `name`, `isAvailableToday`, and `ingredients` read clearly to the LLM. Rename or consolidate properties if the transcript is ambiguous._

## Takeaways
- The notification automation rewards developers who design notifications with clear, keyword-rich body content — build with it in mind to unlock fine-grained user automations.
- The Use Model transcript inspector is the essential tool for tuning how your App Entity properties appear to Apple Intelligence; check it whenever you add or rename a `@Property`.
- Shortcuts Storage syncs via iCloud and can give the Use Model action persistent memory — but entity IDs stored there must be stable across devices, making `SyncableEntity` a prerequisite for entity-referencing storage use cases.
- Shortcuts remains the recommended first testing surface after AppIntentsTesting: if your intent looks correct in Shortcuts, it will work correctly in Siri.

---
_Source: WWDC26 Session 310 page (abstract, chapter summaries, code samples, and resource links)._
