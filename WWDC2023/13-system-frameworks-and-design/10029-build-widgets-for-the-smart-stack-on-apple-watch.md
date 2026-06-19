# Build Widgets for the Smart Stack on Apple Watch
**WWDC23 · Session 10029** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10029/)

_Platforms:_ watchOS 10

## Overview
watchOS 10 introduces the Smart Stack—a swipeable stack of widgets accessible from any watch face. This code-along session builds a complete WidgetKit widget for the Smart Stack using the new `AppIntentConfiguration`, covering timeline construction, custom `TimelineEntry` relevance scoring, and the new `RelevantIntentManager` for proactively promoting a widget based on future date ranges.

Using the Backyard Birds sample app as the running example, the session walks through every component of a WidgetKit widget: the widget structure, the `WidgetConfigurationIntent`, building a `TimelineProvider` with `placeholder`, `snapshot`, `timeline`, and `recommendations`, and crafting SwiftUI views with the `.accessoryRectangular` widget family. New Xcode canvas timeline previews make it easy to visualize how widget views change across timeline entries.

The session closes with `RelevantIntentManager`, a new API that lets apps register date ranges when specific widget configurations are most actionable—allowing the Smart Stack to surface the right widget before the user even knows they need it.

## Key Topics

### Widget Structure and AppIntentConfiguration
WidgetKit's `AppIntentConfiguration` (new in watchOS 10) enables two capabilities: per-content widget pre-configurations in the watchOS widget gallery, and relevance date ranges for Smart Stack prioritization. The `WidgetConfigurationIntent` protocol drives both.

### TimelineEntry and Relevance
Custom `TimelineEntry` types carry all data needed for rendering at a given date—including future dates. The `relevance` property returns `TimelineEntryRelevance(score:duration:)` to rank entries within a timeline. Higher score = higher priority; duration indicates how long that relevance window lasts.

### TimelineProvider Functions
- `placeholder(in:)` — must return quickly; use any valid data.
- `snapshot(for:in:)` async — use the configured intent to return the right backyard data.
- `timeline(for:in:)` async — iterate over visitor events to build start and end entries for each visit; call `RelevantIntentManager` before returning.
- `recommendations()` — return one `AppIntentRecommendation` per backyard, surfacing pre-configured widgets in the gallery.

### Widget Views
The `.accessoryRectangular` family is the primary Smart Stack widget format. Views should use `containerBackground(_:for:.widget)` for the Smart Stack background (not shown on watch faces). Use `widgetAccentable()` for tintable elements and `minimumScaleFactor` for long text.

### RelevantIntentManager
Apps can proactively tell the Smart Stack when a specific widget configuration is most relevant using `RelevantIntentManager.shared.updateRelevantIntents(_:)`. Each `RelevantIntent` pairs a `WidgetConfigurationIntent` with a `RelevantContext.date(from:to:)` range. Call this whenever relevant data changes (e.g., after refilling supplies), alongside `WidgetCenter.shared.reloadTimelines(ofKind:)`.

### Xcode Canvas Timeline Preview
New in Xcode: the canvas shows a strip of timeline entries for a widget, enabling visual verification of the full timeline without running the app.

## APIs & Frameworks

**WidgetKit**
- `Widget` protocol — widget definition
- `AppIntentConfiguration(kind:intent:provider:content:)` **[NEW for watchOS]** — intent-driven configuration
- `WidgetConfigurationIntent` protocol **[NEW]** — drives configuration and relevance
- `TimelineProvider` protocol — `placeholder`, `snapshot`, `timeline`, `recommendations`
- `TimelineEntry` protocol — `date`, `relevance` properties
- `TimelineEntryRelevance(score:duration:)` **[NEW]** — relevance hint for Smart Stack
- `Timeline(entries:policy:)` — timeline with refresh policy
- `TimelineUpdatePolicy.atEnd` — refresh after last entry
- `AppIntentRecommendation(intent:description:)` **[NEW]** — pre-configured widget in gallery
- `WidgetFamily.accessoryRectangular` — rectangular Smart Stack widget family
- `WidgetCenter.shared.reloadTimelines(ofKind:)` — force timeline refresh

**App Intents**
- `RelevantIntentManager` **[NEW]** — manages date-based widget prioritization
- `RelevantIntentManager.shared.updateRelevantIntents(_:)` **[NEW]** — update relevant intents array
- `RelevantIntent(_:widgetKind:relevance:)` **[NEW]** — intent + widget kind + context
- `RelevantContext.date(from:to:)` **[NEW]** — date range relevance context

**SwiftUI**
- `View.containerBackground(_:for:)` **[NEW]** — widget background (shown in Smart Stack, not on watch face)
- `ContainerBackgroundPlacement.widget` **[NEW]** — placement for widget containers
- `View.widgetAccentable()` — tintable with watch face accent color
- `Environment(\.widgetFamily)` — read current widget family
- `Text(_:format:)` with `Duration.FormatStyle` — format durations
- `View.minimumScaleFactor(_:)` — scale down text for long content

## Code Highlights

TimelineEntry with relevance:
```swift
struct SimpleEntry: TimelineEntry {
    var date: Date
    var configuration: ConfigurationAppIntent
    var backyard: Backyard

    var relevance: TimelineEntryRelevance? {
        if let visitor = backyard.visitorEventForDate(date: date) {
            return TimelineEntryRelevance(score: 10,
                duration: visitor.endDate.timeIntervalSince(date))
        }
        return TimelineEntryRelevance(score: 0)
    }
}
```

Registering relevant intents by date range:
```swift
let relevantFoodDateContext = RelevantContext.date(
    from: backyard.lowSuppliesDate(for: .food),
    to: backyard.expectedEmptyDate(for: .food))
let relevantFoodIntent = RelevantIntent(configIntent,
    widgetKind: "BackyardVisitorsWidget",
    relevance: relevantFoodDateContext)
try await RelevantIntentManager.shared.updateRelevantIntents(relevantIntents)
```

Container background for Smart Stack:
```swift
.containerBackground(entry.backyard.backgroundColor.gradient, for: .widget)
```

## Takeaways
- `AppIntentConfiguration` enables per-content widget pre-configurations and relevance date ranges, both key for a high-quality Smart Stack experience.
- `TimelineEntryRelevance` scores entries within a timeline; `RelevantIntentManager` signals relevance across timelines for specific configurations.
- The `.accessoryRectangular` family and `containerBackground` are the two new watchOS 10 requirements for Smart Stack widgets.
- The new Xcode canvas timeline preview dramatically accelerates widget development by showing the full timeline visually at design time.

---
_Source: WWDC23 Session 10029 page (abstract, chapter summaries, code samples, and resource links)._
