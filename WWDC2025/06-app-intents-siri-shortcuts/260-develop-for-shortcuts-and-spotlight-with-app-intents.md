# Develop for Shortcuts and Spotlight with App Intents
**WWDC25 · Session 260** · [Watch](https://developer.apple.com/videos/play/wwdc2025/260/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26

## Overview
This session covers three major new capabilities for the App Intents framework: the new "Use Model" action in Shortcuts that lets users invoke Apple Intelligence language models within their automations; running Shortcuts actions directly from Spotlight on Mac; and personal automations on Mac (folder-change, Bluetooth, time-of-day triggers, etc.).

The session is especially valuable for developers whose apps expose App Entities, as it explains how to structure those entities for optimal compatibility with the Use Model action — covering attributed string support, entity JSON representation, and Find actions.

## Key Topics

### Use Model Action
- A new Shortcuts action that invokes Apple Intelligence models (on-device, Private Cloud Compute, or ChatGPT) with a natural-language prompt. **[NEW]**
- **Output types**: Text (can be rich/attributed), Dictionary (structured key-value), or **Content from your app** (App Entities returned directly).
- When the output connects to an action expecting a Boolean, the runtime automatically coerces the model's yes/no response.
- Apps should expose **Attributed String** parameters in App Intents where rich text is possible, so the model's formatted output passes through losslessly.
- For entity output: the model receives a **JSON representation** of the entity built from `@Property` values exposed to Shortcuts, the type display representation name, and the entity's title/subtitle from `DisplayRepresentation`.
- **Follow Up toggle**: users can iterate on the model's output before passing it to the next action (e.g., "double the recipe").

### Spotlight on Mac
- App Intents automatically surface as **runnable actions in Mac Spotlight**. **[NEW]**
- Rules for Spotlight eligibility: all required parameters without default values must appear in the `parameterSummary`; intent must not have `isDiscoverable = false` or `assistantOnly = true`; intents used only for widget configuration (no `perform`) are excluded.
- **Suggestions**: implement `SuggestedEntities` (subset of large collections) or `AllEntities` (small bounded sets) on queries; also tag on-screen content via `NSUserActivity.appEntityIdentifier`.
- **Search**: implement `EntityStringQuery` or `IndexedEntity` for deeper search beyond suggestions.
- **Background vs. foreground intents**: pair a background intent (e.g., Create Event) with an `OpensIntent` (e.g., Open Event) so users can choose their preferred experience.

### Automations on Mac
- **[NEW]** Personal Automations come to Mac with triggers including Folder changes, external drive connections, Time of Day, Bluetooth, and more.
- Any intent available on macOS (including iOS apps installable on macOS) can be used in Mac Automations.

## APIs & Frameworks

### App Intents
- `AppIntent` protocol — base for all intents.
- `AppEntity` / `@Property` — **[NEW JSON representation for Use Model]** entity properties exposed to Shortcuts are serialized to JSON for the model.
- `IndexedEntity` protocol — indexes entities in Core Spotlight; new `indexingKey` parameter on `@Property` to map to Spotlight attribute keys. **[NEW]**
- `EnumerableEntityQuery` — provides `AllEntities` for bounded sets.
- `EntityQuery` + `SuggestedEntities` — provides suggested entities for large/unbounded collections.
- `EntityStringQuery` — enables text search within entity queries.
- `PredictableIntent` protocol — allows Spotlight to surface intent suggestions based on usage history.
- `parameterSummary` — natural-language intent summary shown in Spotlight UI and Shortcuts editor.
- `OpensIntent` — returned by a background intent to pair it with a foreground navigation intent. **[NEW in Spotlight context]**
- `NSUserActivity.appEntityIdentifier` — **[NEW]** tags on-screen content for context-based Spotlight suggestions.

### Foundation / SwiftUI
- `AttributedString` — key type for passing rich text from the Use Model action into App Intent parameters without loss. **[NEW recommendation]**

### Resources
- App Intents Travel Tracking App sample — available on developer.apple.com.
- App Intents Sample Code app — includes `EntityStringQuery` example.

## Code Highlights
No code shown in this session; referenced implementations are in the linked sample apps and the "Exploring New Advances in App Intents" session.

## Takeaways
- Expose your entities' important properties via `@Property` (with `indexingKey` where possible) so the Use Model action can reason over them meaningfully.
- Support `AttributedString` input parameters in App Intents that accept text so rich-text output from the model passes through intact.
- Audit your intents for Spotlight eligibility: every required parameter with no default must appear in `parameterSummary`.
- Pair background intents with `OpensIntent` foreground intents to give Mac Spotlight users a seamless "create then view" experience.

---
_Source: WWDC25 Session 260 page (abstract, chapter summaries, and full transcript)._
