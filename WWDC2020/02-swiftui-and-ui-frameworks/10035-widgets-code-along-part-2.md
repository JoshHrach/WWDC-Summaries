# Widgets Code-along, Part 2: Alternate Timelines
**WWDC20 · Session 10035** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10035/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
Part 2 of the three-part Widgets Code-along builds on the single-family, single-entry widget from Part 1. This session adds support for multiple widget families using the `@Environment(\.widgetFamily)` value, constructs a proper multi-entry timeline with one-minute granularity so the widget never asks WidgetKit to reload unnecessarily, adds `TimelineEntryRelevance` hints to enable intelligent stack rotation, makes the widget user-configurable via a custom `INIntent`, and adds `widgetURL` deep linking from the small and medium widget into the app.

## Key Topics

### Multiple Widget Families
Three families are available: `.systemSmall`, `.systemMedium`, `.systemLarge`. Support multiple by:
1. Adding them to `.supportedFamilies([.systemSmall, .systemMedium])`
2. Reading `@Environment(\.widgetFamily)` inside the entry view and switching on it to return a different view hierarchy per size

The entry view and placeholder view should both switch on `widgetFamily` to provide consistent placeholders at each size.

### Full Multi-Entry Timeline
Instead of returning a single entry and relying on `.atEnd` to trigger an immediate reload, generate all entries up to the known end state:
- Compute the `endDate` (e.g., when the character finishes healing)
- Loop from `currentDate` to `endDate` in one-minute increments, appending an `EmojiRangerEntry` for each step
- Return `Timeline(entries: entries, policy: .atEnd)` — WidgetKit won't request a reload until all entries are consumed

This reduces unnecessary wake-ups and lets the system schedule intelligently across all active widgets.

### Timeline Entry Relevance
`TimelineEntry` has an optional `relevance: TimelineEntryRelevance?` property. Set it with `TimelineEntryRelevance(score: Float, duration: TimeInterval)`:
- `score` — relative priority for this entry vs. other entries; developer-defined scale (e.g., use health fraction directly so fully-healed = 1.0)
- `duration` — how long the relevance applies
- Used by the system to intelligently rotate widget stacks to show the most relevant widget at the right time

### Intent-Based Configuration (INIntent)
Converting a `StaticConfiguration` to an `IntentConfiguration` makes the widget user-configurable directly on the Home Screen (long press → Edit Widget):
1. Create a SiriKit Intent Definition file (File > New > File > SiriKit Intent Definition); add it to both the app and widget extension targets
2. Add a custom intent: category = Information, mark "Intent is eligible for widgets," add parameters (e.g., `hero` as an enum)
3. Change `TimelineProvider` to `IntentTimelineProvider`; `getSnapshot` and `getTimeline` gain a `configuration: CharacterSelectionIntent` argument
4. Change `StaticConfiguration` to `IntentConfiguration(kind:intent:provider:placeholder:content:)` passing the intent type

### Widget Deep Linking
- `systemSmall` widgets are a single tap target — use `.widgetURL(url)` modifier on the root view
- `systemMedium` and `systemLarge` can use SwiftUI's `Link(destination:)` to create independent tappable regions (covered in Part 3)

## APIs & Frameworks

### WidgetKit
- `@Environment(\.widgetFamily) var widgetFamily` **[NEW]** — current rendering family inside entry view
- `TimelineEntryRelevance(score: Float, duration: TimeInterval)` **[NEW]** — relevance hint for stack rotation
- `TimelineEntry.relevance: TimelineEntryRelevance?` **[NEW]** — optional property on any `TimelineEntry`-conforming type
- `IntentConfiguration(kind:intent:provider:placeholder:content:)` **[NEW]** — widget configuration backed by a custom `INIntent`
- `IntentTimelineProvider` protocol **[NEW]**
  - `getSnapshot(for configuration: Intent, in context: Context, completion: (Entry) -> Void)`
  - `getTimeline(for configuration: Intent, in context: Context, completion: (Timeline<Entry>) -> Void)`
- `View.widgetURL(_ url: URL)` **[NEW]** — sets the deep-link URL for a tapped `systemSmall` widget (or the whole widget for larger sizes that don't use `Link`)

### SiriKit (for Widget Configuration)
- Custom `INIntent` in an Intent Definition file — `category: Information`, marked widget-eligible
- Enum parameters provide the configurable options shown in the Edit Widget UI

## Code Highlights

Multi-family entry view using `@Environment(\.widgetFamily)`:
```swift
struct EmojiRangerWidgetEntryView: View {
    var entry: EmojiRangerEntry
    @Environment(\.widgetFamily) var widgetFamily

    var body: some View {
        switch widgetFamily {
        case .systemSmall:
            AvatarView(character: entry.character)
                .widgetURL(entry.character.url)
        default:
            HStack {
                AvatarView(character: entry.character)
                Text(entry.character.bio)
            }
            .widgetURL(entry.character.url)
        }
    }
}
```

Full timeline with one-minute entries and relevance:
```swift
func getTimeline(for configuration: CharacterSelectionIntent,
                 in context: Context,
                 completion: @escaping (Timeline<EmojiRangerEntry>) -> Void) {
    let selectedCharacter = character(for: configuration)
    let endDate = selectedCharacter.fullHealthDate
    var currentDate = Date()
    var entries: [EmojiRangerEntry] = []

    while currentDate < endDate {
        let relevance = TimelineEntryRelevance(score: Float(selectedCharacter.healthLevel(at: currentDate)),
                                               duration: 60)
        entries.append(EmojiRangerEntry(date: currentDate,
                                        character: selectedCharacter,
                                        relevance: relevance))
        currentDate = Calendar.current.date(byAdding: .minute, value: 1, to: currentDate)!
    }
    completion(Timeline(entries: entries, policy: .atEnd))
}
```

Intent-based configuration:
```swift
@main
struct EmojiRangerWidget: Widget {
    var body: some WidgetConfiguration {
        IntentConfiguration(kind: "EmojiRanger",
                            intent: CharacterSelectionIntent.self,
                            provider: IntentProvider()) { entry in
            EmojiRangerWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("Emoji Ranger")
        .description("Track your favorite Ranger.")
        .supportedFamilies([.systemSmall, .systemMedium])
    }
}
```

## Takeaways

- Pre-generate the full timeline up to a known end point with appropriate granularity (e.g., one-minute intervals for a countdown) — this reduces reload wake-ups and lets WidgetKit schedule across all widgets optimally.
- `TimelineEntryRelevance` is the signal to the system's widget-stack intelligence; use your app's own data scale (e.g., health fraction) as the score, not an arbitrary priority number.
- Switching to `IntentConfiguration` / `IntentTimelineProvider` adds Home Screen configurability with just a new Intent Definition file and a one-word type change — no separate UI required.
- `@Environment(\.widgetFamily)` inside the entry view is the correct place to branch layout per size; apply the same branch in the placeholder view for consistency.

---
_Source: WWDC20 Session 10035 page (abstract, transcript, and resource links)._
