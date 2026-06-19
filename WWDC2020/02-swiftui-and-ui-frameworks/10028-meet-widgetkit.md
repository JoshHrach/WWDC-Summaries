# Meet WidgetKit
**WWDC20 · Session 10028** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10028/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
WidgetKit is a brand-new framework in iOS 14 / iPadOS 14 / macOS Big Sur that enables apps to place glanceable, always-up-to-date widgets on the Home Screen, Today View, and macOS Notification Center. Widgets are built entirely with SwiftUI and run as background extensions that return a series of SwiftUI view hierarchies ("a timeline") — the system renders them just-in-time at the scheduled timestamps, avoiding the need to launch a mini-app each time.

Three design goals drive WidgetKit: widgets must be **glanceable** (no loading spinners, instant content, no interactive controls), **relevant** (appear in Smart Stacks at the right moment via on-device intelligence and developer-provided relevance hints), and **personalizable** (configurable through the Intents framework, multiple sizes, deep links).

## Key Topics

### Widget Definition
A widget is declared by conforming to the `Widget` protocol and returning a `WidgetConfiguration` from `body`. Key sub-components:
- **kind** — a `String` identifying this widget type; a single extension can expose multiple kinds
- **configuration** — either `StaticConfiguration` (no user config) or `IntentConfiguration` (user-configurable via an `INIntent` subclass)
- **supportedFamilies** — `.systemSmall`, `.systemMedium`, `.systemLarge`; default is all three
- **placeholder view** — required; device-agnostic representation shown when the system first installs the widget or when environment changes (Dynamic Type, Dark Mode); must contain no user data

### Timeline Architecture
- The widget extension returns a `Timeline<Entry>` containing an array of `(view, date)` pairs
- System serializes view hierarchies to disk and renders each entry just-in-time at the specified date — avoids launching the extension per-render
- **Snapshot** — a single, quickly returned entry used to preview the widget in the Widget Gallery (in most cases the same as the first timeline entry)
- **Timeline reload policies**: `.atEnd` (reload when timeline is exhausted), `.after(Date)` (reload at a specific date), `.never` (no automatic reload)
- Frequently viewed widgets receive more reload budget; infrequent widgets receive fewer

### Timeline Reloads
- **From the main app**: call `WidgetCenter.shared.reloadTimelines(ofKind:)` or `reloadAllTimelines()` when the app's state changes relevantly; be judicious — only reload when the widget content should actually change
- **From background notifications**: call reload API from the notification handler
- **From the extension itself**: request background `URLSession` tasks; handle delivery via `onBackgroundURLSessionEvents(_:_:)` modifier; the payload wakes the extension

### Glanceable UI Rules
- Widgets are **not mini-apps**: no scroll views, no switches or interactive controls, no video or animated images
- `systemSmall` has a single tap target (the whole widget is one link); use `widgetURL(_:)` to associate a deep link URL
- `systemMedium` and `systemLarge` support per-item deep links via SwiftUI's `Link` view

### Personalization with Intents
- Widget configuration is driven by `INIntent` subclasses (same system used for Siri/Shortcuts)
- WidgetKit automatically generates configuration UI from the Intent parameters — no extra UI code
- Dynamic options: the Intents extension (or in-app Intent handling, new in iOS 14) can return server-fetched options in real time (e.g., search for a stock symbol)
- `IntentConfiguration` replaces `StaticConfiguration`; the provider conforms to `IntentTimelineProvider` and receives the `Intent` object in its `timeline(for:with:completion:)` method

### Intelligence and Smart Stacks
- The system rotates Smart Stack widgets using on-device intelligence
- Developer influence: **donate `INIntent` Shortcuts** from the app (if the widget is backed by the same intent, the stack may rotate to it at predicted times)
- **`TimelineEntryRelevance`** — annotate individual timeline entries with a `score: Float` (relative to all entries you have ever provided) and `duration: TimeInterval` to hint at when that entry should surface at the top of the stack

## APIs & Frameworks

- **WidgetKit** **[NEW]**
  - `Widget` — protocol; defines `body: some WidgetConfiguration`
  - `StaticConfiguration(kind:provider:placeholder:content:)` **[NEW]** — non-configurable widget
  - `IntentConfiguration(kind:intent:provider:placeholder:content:)` **[NEW]** — configurable widget backed by an `INIntent`
  - `WidgetConfiguration` — result builder protocol for widget body
  - `.configurationDisplayName(_:)` / `.description(_:)` — metadata shown in Widget Gallery
  - `.supportedFamilies(_:)` — declare supported size families
  - `WidgetFamily` — `.systemSmall`, `.systemMedium`, `.systemLarge`
  - `TimelineProvider` **[NEW]** — protocol with `snapshot(with:completion:)` and `timeline(with:completion:)`
  - `IntentTimelineProvider` **[NEW]** — variant of `TimelineProvider` that receives an `INIntent` in `timeline(for:with:completion:)`
  - `TimelineEntry` — protocol; requires `date: Date`
  - `Timeline<Entry>` **[NEW]** — wraps an array of entries and a `TimelineReloadPolicy`
  - `TimelineReloadPolicy` **[NEW]** — `.atEnd`, `.after(Date)`, `.never`
  - `TimelineEntryRelevance` **[NEW]** — `score: Float` + `duration: TimeInterval`; annotate entries for Smart Stack intelligence
  - `WidgetCenter` **[NEW]** — reload coordinator callable from the main app or extension:
    - `WidgetCenter.shared.reloadTimelines(ofKind:)` — reload a specific widget kind
    - `WidgetCenter.shared.reloadAllTimelines()` — reload all widget kinds
    - `WidgetCenter.shared.getCurrentConfigurations(_:)` — enumerate installed widget configurations
  - `widgetURL(_:)` — SwiftUI view modifier; associates a deep-link URL with the whole widget (required for `systemSmall`)
  - `onBackgroundURLSessionEvents(_:_:)` — SwiftUI modifier; handles background URL session events in the extension
- **SwiftUI**
  - `Link(destination:label:)` — creates a tappable deep-link target within `systemMedium`/`systemLarge` widgets

## Code Highlights

Static widget definition:
```swift
@main
public struct SampleWidget: Widget {
    private let kind: String = "SampleWidget"

    public var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind,
                            provider: Provider(),
                            placeholder: PlaceholderView()) { entry in
            SampleWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("This is an example widget.")
    }
}
```

TimelineProvider conformance:
```swift
public struct Provider: TimelineProvider {
    public func snapshot(with context: Context, completion: @escaping (SimpleEntry) -> ()) {
        completion(SimpleEntry(date: Date()))
    }

    public func timeline(with context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        let entry = SimpleEntry(date: Date())
        let timeline = Timeline(entries: [entry], policy: .atEnd)
        completion(timeline)
    }
}
```

Intent-driven widget definition:
```swift
IntentConfiguration(kind: kind, intent: ConfigurationIntent.self,
                    provider: Provider(), placeholder: PlaceholderView()) { entry in
    SampleWidgetEntryView(entry: entry)
}
```

Reloading from the main app:
```swift
WidgetCenter.shared.reloadTimelines(ofKind: "SampleWidget")
```

## Takeaways
- Widgets are not mini-apps; design for instant glanceability — no interactive controls, no video, single tap target on `systemSmall`.
- The timeline architecture means the system renders views from serialized data; return accurate dates and plan multiple entries in advance (ideally a full day's worth).
- Use `WidgetCenter.reloadTimelines(ofKind:)` sparingly from the foreground app — only when content has genuinely changed.
- Intelligence in Smart Stacks is influenced by Shortcut donations and `TimelineEntryRelevance`; make scores relative to your own past entries, not absolute values.
- The same `INIntent` infrastructure used for Siri/Shortcuts drives widget configuration, generating UI automatically from Intent parameters.

---
_Source: WWDC20 Session 10028 page (abstract, transcript, and code samples)._
