# WidgetKit Foundations
**WWDC26 · Session 277** · [Watch](https://developer.apple.com/videos/play/wwdc2026/277/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, visionOS

## Overview
This foundations-level session walks through everything needed to build a quality widget from scratch. Using a book-club reading-goal widget as the running example, it covers the full development arc: defining a widget extension, building a `TimelineProvider`, selecting a reload policy, choosing supported families, deep-linking into the app, enabling configurable widgets with App Intents, and adapting the widget's appearance across the system's rendering modes (full-color, tinted, clear).

Two new features are highlighted. First, a new `systemExtraLargePortrait` widget family comes to macOS, iOS, and iPadOS 27 — a taller variant of the existing extra-large family oriented for portrait use. Second, interactive widget elements backed by `App Intents` let users perform meaningful actions directly from the widget without launching the app.

This is the go-to reference session for anyone who hasn't built a widget before, and a useful refresh for existing widget developers adopting new 2027 APIs.

## Key Topics

### Fundamentals (1:03)
A widget extension exposes a `Widget` conformance with a `WidgetConfiguration` body. The configuration type is either `StaticConfiguration` (fixed content) or `AppIntentConfiguration` (user-configurable content). Both require a `TimelineProvider` that produces an array of `TimelineEntry` values — each entry pairs a `Date` with the data needed to render the widget at that moment.

The three qualities of memorable widgets: **glanceable** (meaningful at a glance, minimal interaction required), **relevant** (right content at the right time), **personalizable** (configurable by the user).

**Timeline providers**: `StaticConfiguration` uses `TimelineProvider`; `AppIntentConfiguration` uses `AppIntentTimelineProvider`. The provider implements `placeholder(in:)`, `getSnapshot(in:completion:)`, and `getTimeline(in:completion:)`. The `Timeline` is returned with a `TimelineReloadPolicy` (`.atEnd`, `.after(Date)`, `.never`).

**Widget families**: declared with `.supportedFamilies([.systemSmall, .systemMedium, .systemLarge, .systemExtraLarge, .systemExtraLargePortrait, .accessoryCircular, .accessoryRectangular, ...])`. The new `systemExtraLargePortrait` family is available on macOS, iOS, and iPadOS 27.

### Integrate with Your App (13:15)
Three integration points:
1. **Deep links** — `.widgetURL(URL(string: "..."))` modifier routes taps into the app; use `Link` for multi-tappable zones within a single widget
2. **Configurable widgets** — use `AppIntentConfiguration` with an `AppIntent` that has `@Parameter` values the user edits in widget settings
3. **Interactive elements** — `Button(intent:)` and `Toggle(intent:)` inside the widget view perform `AppIntent.perform()` actions in-process without launching the app

### Adapt with the System (17:04)
Widgets render in three system appearance modes:
- **Full color** — default; all colors as specified
- **Tinted** — system applies a monochrome tint; most views adapt automatically
- **Clear** — transparent background; requires `containerBackground` to be meaningful

The `.widgetAccentedRenderingMode(.fullColor)` modifier on a specific view (e.g., `Image`) opts that view out of the tinted mode and keeps it full-color. Test all three modes in Xcode previews and Simulator.

## APIs & Frameworks

**WidgetKit** — `import WidgetKit`
- `Widget` protocol — entry point for a widget extension
- `WidgetConfiguration` — result type of `body`
- `StaticConfiguration(kind:provider:content:)` — fixed widget
- `AppIntentConfiguration(kind:intent:provider:content:)` — configurable widget
- `TimelineProvider` protocol — `placeholder`, `getSnapshot`, `getTimeline`
- `AppIntentTimelineProvider` protocol — configurable variant
- `TimelineEntry` — `date: Date` + custom data
- `Timeline(entries:policy:)` — returned from `getTimeline`
- `TimelineReloadPolicy` — `.atEnd`, `.after(Date)`, `.never`
- `.supportedFamilies([WidgetFamily])` modifier
- `WidgetFamily` — `.systemSmall`, `.systemMedium`, `.systemLarge`, `.systemExtraLarge`
- **[NEW]** `WidgetFamily.systemExtraLargePortrait` — new taller portrait family (macOS/iOS/iPadOS 27)
- `.accessoryCircular`, `.accessoryRectangular`, `.accessoryInline` — Lock Screen / watch faces
- `.containerBackground(for: .widget) { }` modifier — required background declaration
- `.widgetURL(_:)` modifier — deep link on tap
- `Link(destination:)` — tappable zone inside a widget
- `.widgetAccentedRenderingMode(_:)` modifier — `WidgetRenderingMode` adaptation
- `WidgetRenderingMode` — `.fullColor`, `.accented`, `.vibrant`

**App Intents** — `import AppIntents`
- `AppIntentConfiguration` — configurable widget backed by an intent
- `AppIntentTimelineProvider` — provider associated with an intent
- `AppIntent.perform() async throws -> some IntentResult` — action handler
- `Button(intent:)` — interactive widget button
- `Toggle(intent:)` — interactive widget toggle
- `@Parameter` — annotates intent parameters shown in widget configuration UI

**SwiftUI**
- `@Environment(\.widgetFamily)` — read current family in the view
- `@Environment(\.colorScheme)` — used to force `.dark` on the widget
- `ContainerRelativeShape` — shape that matches the widget container corner radius

## Code Highlights

Minimal widget definition with deep link and supported families:
```swift
struct DailyReadingGoalWidget: Widget {
    let kind = "DailyReadingGoalWidget"
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: DailyReadingGoalProvider()) { entry in
            DailyReadingGoalView(book: entry.book, message: entry.message, timeOfDay: entry.timeOfDay)
                .containerBackground(for: .widget) { Background() }
                .widgetURL(URL(string: "bookclub://reading/\(entry.book.bookID)"))
        }
        .supportedFamilies([.systemMedium])
    }
}
```

Keeping a specific image in full color during tinted rendering:
```swift
Image(imageName: bundle: .main)
    .widgetAccentedRenderingMode(.fullColor)
```

## Takeaways
- Every widget needs a `containerBackground` — the system will not display the widget without one in iOS/macOS 27.
- Use `.supportedFamilies` to opt into only the sizes your widget content actually supports; don't declare families you haven't tested at that size.
- Add `Button(intent:)` interactive elements for the single most important action in your widget — this dramatically increases engagement without requiring a full app launch.
- Test all three appearance modes (full color, tinted, clear) in Xcode Previews before shipping; tinted mode in particular can make image-heavy widgets illegible if `widgetAccentedRenderingMode` is not applied.

---
_Source: WWDC26 Session 277 page (abstract, chapter summaries, code samples, and resource links)._
