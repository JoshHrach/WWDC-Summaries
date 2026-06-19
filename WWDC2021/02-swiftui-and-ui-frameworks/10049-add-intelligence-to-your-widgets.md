# Add Intelligence to Your Widgets
**WWDC21 · Session 10049** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10049/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
This session covers how to make widgets intelligent participants in Smart Stacks by working with the system's on-device intelligence. Two complementary mechanisms are covered: Smart Rotate (which scrolls to a widget already in a stack) and Widget Suggestions (new in iOS 15, which temporarily inserts a widget the user doesn't even have in their stack).

The session walks through three APIs that widgets can adopt — `INRelevantShortcut`, `TimelineEntryRelevance`, and `INInteraction` — and explains the tradeoffs and use cases of each. A running example app called Cards (showing credit card recent purchases) demonstrates all three APIs in practice.

Widgets that surface relevant information proactively become more discoverable and more useful, appearing in the right context — for example, a Calendar widget surfacing just before a meeting even if the user never added it to their stack.

## Key Topics

### Smart Stacks and Widget Intelligence
- Smart Stacks let users scroll through multiple widgets in one space
- Smart Rotate: system scrolls to a more relevant widget in the stack at the right time
- Widget Suggestions **[NEW in iOS 15]**: system temporarily inserts a widget not in the stack, removes it when no longer relevant
- iPad Home screen now supports widgets (iPadOS 15) **[NEW]**

### Three APIs for Widget Intelligence

**1. INRelevantShortcut — Explicit Widget Suggestions**
- Donate when app detects a highly relevant situation (e.g., purchase just made)
- Works for both `StaticConfiguration` and `IntentConfiguration` widgets
- `widgetKind` property specifies which widget to suggest
- `relevanceProviders`: use `INDateRelevanceProvider` for a fixed time window, or empty array to let system infer timing from behavioral patterns
- Stored via `INRelevantShortcutStore.default.setRelevantShortcuts(_:completionHandler:)`
- Invalidate by omitting from next donation array

**2. TimelineEntryRelevance — Smart Rotate Scoring**
- Widget provides a `score` and `duration` alongside each `TimelineEntry`
- Positive score = eligible for Smart Rotation; score of 0 = ineligible
- Scores are relative to all other entries in the widget's timelines — use consistent scaling
- `duration`: how long score stays valid; 0 = valid until next entry with relevance

**3. INInteraction — Behavioral Learning**
- Donate the widget's configuration intent every time user views corresponding content in the app
- System builds a behavioral model and predicts when to show the widget
- Also surfaces intent as Siri Suggestion on Lock Screen, Spotlight, and Siri Shortcut Suggestions widget
- Works even if widget doesn't adopt intents (surfaces shortcut suggestions elsewhere)

### Developer Testing
- Enable "WidgetKit Developer Mode" in Developer Settings to bypass system throttling during development

## APIs & Frameworks

### WidgetKit
- `TimelineEntry` protocol — optional `relevance: TimelineEntryRelevance?` property
- `TimelineEntryRelevance` — `score: Float`, `duration: TimeInterval`
- `StaticConfiguration`, `IntentConfiguration` — widget configuration types
- `TimelineProvider` / `IntentTimelineProvider` — provide timelines with relevance

### Intents Framework
- `INRelevantShortcut` — shortcut with relevance metadata for Widget Suggestions
  - `widgetKind: String` **[NEW]** — identifies which widget to suggest
  - `shortcutRole: INShortcutRole` — e.g., `.information`
  - `relevanceProviders: [INRelevanceProvider]`
- `INRelevantShortcutStore` — singleton store
  - `default` — shared instance
  - `setRelevantShortcuts(_:completionHandler:)` — donate/update suggestions
- `INRelevanceProvider` (abstract base)
- `INDateRelevanceProvider(start:end:)` — time-bounded relevance
- `INShortcut(intent:)` — wrap intent as shortcut
- `INInteraction(intent:response:)` — wrap intent for behavioral donation
  - `donate(completion:)` — submit to on-device intelligence
- `INRelevantShortcut.shortcutRole` — `.information`, etc.

## Code Highlights

Donating an `INRelevantShortcut` when a purchase occurs:
```swift
var relevantShortcuts: [INRelevantShortcut] = []
let intent = ViewRecentPurchasesIntent()
intent.card = Card(identifier: card.identifier)
intent.category = .all
if let shortcut = INShortcut(intent: intent) {
    let relevantShortcut = INRelevantShortcut(shortcut: shortcut)
    relevantShortcut.shortcutRole = .information
    relevantShortcut.widgetKind = "CardRecentPurchasesWidget"
    let dateProvider = INDateRelevanceProvider(start: Date(), end: Date(timeIntervalSinceNow: 1800))
    relevantShortcut.relevanceProviders = [dateProvider]
    relevantShortcuts.append(relevantShortcut)
}
INRelevantShortcutStore.default.setRelevantShortcuts(relevantShortcuts) { error in ... }
```

Adding `TimelineEntryRelevance` to a timeline entry:
```swift
let relevance = TimelineEntryRelevance(score: 16.29, duration: 1800)
let entry = CardRecentPurchasesEntry(date: Date(), relevance: relevance, card: card, category: category)
```

Donating an `INInteraction` when user views purchases:
```swift
.onAppear {
    let intent = ViewRecentPurchasesIntent()
    intent.card = Card(identifier: card.id.uuidString, displayString: card.name)
    intent.category = .all
    let interaction = INInteraction(intent: intent, response: nil)
    interaction.donate { error in ... }
}
```

## Takeaways
- Widget Suggestions (iOS 15) allow your widget to appear in Smart Stacks the user never added it to — a powerful discoverability mechanism.
- Use `INRelevantShortcut` for explicit, time-bounded suggestions (event-driven); use `INInteraction` donations to let the system learn behavioral patterns.
- `TimelineEntryRelevance` scores let you tell the system which timeline entries are most worth rotating to — keep scoring consistent and relative within your own timelines.
- All three APIs work together and complement each other; a donation is not guaranteed to surface the widget, but combining all three gives the best chance.

---
_Source: WWDC21 Session 10049 page (abstract, chapter summaries, code samples, and resource links)._
