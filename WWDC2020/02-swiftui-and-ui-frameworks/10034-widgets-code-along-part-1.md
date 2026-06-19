# Widgets Code-along, Part 1: The Adventure Begins
**WWDC20 · Session 10034** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10034/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
Part 1 of a three-part code-along series introducing WidgetKit. Starting from a provided Emoji Rangers sample app, the session walks through creating a widget extension target from scratch in Xcode: adding the target, connecting an existing SwiftUI view as the widget entry view, implementing a minimal `TimelineProvider` with both snapshot and timeline methods, restricting supported widget families, and using the new `isPlaceholder(_:)` modifier to produce a properly redacted placeholder view. By the end, a functional small-size widget is running on the Home Screen.

## Key Topics

### What a Widget Is
A widget is a SwiftUI view that updates over time. The view is the same declarative SwiftUI code used for the rest of the app — no new language or paradigm. WidgetKit controls when and how it updates, which is the subject of the timeline system explored across all three parts.

### Creating a Widget Extension Target
Widgets live in a separate extension target, created via File > New > Target > Widget Extension in Xcode. The extension is activated in the scheme, files from the main app that the widget needs are added to the new target's membership, and the extension is otherwise treated as a separate mini-app module.

### Widget Structure
Every widget has three required components:
1. **`TimelineEntry`** — a struct conforming to `TimelineEntry`; holds the data the view needs for a specific point in time (minimally a `date: Date`); add any app-specific properties (e.g., a `Character` model object)
2. **`TimelineProvider`** — drives the widget; implements two methods:
   - `getSnapshot(in:completion:)` — called when WidgetKit needs a single representative entry (e.g., Widget Gallery preview); return quickly with a plausible entry
   - `getTimeline(in:completion:)` — called once a widget is placed; return a `Timeline` containing one or more entries and a `TimelineReloadPolicy`
3. **Widget entry view** — a SwiftUI `View` that receives a `TimelineEntry` and renders the widget UI

### Supported Families
By default the widget supports all three sizes (`.systemSmall`, `.systemMedium`, `.systemLarge`). Restrict supported sizes using the `.supportedFamilies([...])` modifier on the widget configuration to signal to the gallery which sizes the widget is ready for.

### Placeholder View
`PlaceholderView` is shown while WidgetKit is loading the timeline. Use `.isPlaceholder(true)` modifier on the SwiftUI view hierarchy to automatically replace text with rounded grey rectangles and images with grey fills — no manual placeholder logic required.

### SwiftUI Previews for Widgets
`WidgetPreviewContext(family:)` passed to `.previewContext(_:)` renders a live SwiftUI preview at the exact dimensions of the specified widget family size. Use a `Group` in the preview provider to show both the full widget and the placeholder side-by-side.

## APIs & Frameworks

### WidgetKit
- `TimelineEntry` protocol **[NEW]** — `date: Date` required; add any additional data properties
- `TimelineProvider` protocol **[NEW]**
  - `getSnapshot(in: Context, completion: (Entry) -> Void)`
  - `getTimeline(in: Context, completion: (Timeline<Entry>) -> Void)`
- `Timeline<Entry>` **[NEW]** — wraps an array of entries and a `TimelineReloadPolicy`
- `TimelineReloadPolicy` **[NEW]** — `.atEnd`, `.after(Date)`, `.never`
- `StaticConfiguration(kind:provider:placeholder:content:)` **[NEW]** — widget configuration for widgets without user customization
- `Widget` protocol **[NEW]** — `var body: some WidgetConfiguration`
- `WidgetConfiguration.supportedFamilies(_:)` **[NEW]** — restricts to `.systemSmall`, `.systemMedium`, `.systemLarge`
- `WidgetFamily` enum **[NEW]** — `.systemSmall`, `.systemMedium`, `.systemLarge`

### SwiftUI (Widget-specific)
- `View.isPlaceholder(_ value: Bool)` **[NEW]** — renders text as grey rounded rectangles and images as grey fills for placeholder state
- `WidgetPreviewContext(family: WidgetFamily)` **[NEW]** — used with `.previewContext(_:)` to preview widget at correct dimensions

## Code Highlights

Minimal widget conformance (end state of Part 1):
```swift
import WidgetKit
import SwiftUI

struct EmojiRangerEntry: TimelineEntry {
    var date: Date
    var character: CharacterDetail
}

struct Provider: TimelineProvider {
    func getSnapshot(in context: Context,
                     completion: @escaping (EmojiRangerEntry) -> Void) {
        completion(EmojiRangerEntry(date: Date(), character: .panda))
    }

    func getTimeline(in context: Context,
                     completion: @escaping (Timeline<EmojiRangerEntry>) -> Void) {
        let entry = EmojiRangerEntry(date: Date(), character: .panda)
        completion(Timeline(entries: [entry], policy: .atEnd))
    }
}

struct EmojiRangerWidgetEntryView: View {
    var entry: EmojiRangerEntry
    var body: some View {
        AvatarView(character: entry.character)
    }
}

@main
struct EmojiRangerWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: "EmojiRanger", provider: Provider()) { entry in
            EmojiRangerWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("Emoji Ranger")
        .description("Keep track of your favorite Emoji Ranger.")
        .supportedFamilies([.systemSmall])
    }
}
```

Placeholder view with automatic redaction:
```swift
struct PlaceholderView: View {
    var body: some View {
        AvatarView(character: .panda)
            .isPlaceholder(true)  // text → grey rectangles; images → grey fills
    }
}
```

SwiftUI preview at widget dimensions:
```swift
struct Widget_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            EmojiRangerWidgetEntryView(entry: .init(date: Date(), character: .panda))
                .previewContext(WidgetPreviewContext(family: .systemSmall))
            PlaceholderView()
                .previewContext(WidgetPreviewContext(family: .systemSmall))
        }
    }
}
```

## Takeaways

- A widget is a SwiftUI view attached to a `TimelineProvider` — the provider supplies entries, each containing a `date` and any app data, and the system renders the view at the right time.
- Creating a widget requires adding a Widget Extension target in Xcode, then implementing `TimelineEntry`, `TimelineProvider`, an entry view, and a `Widget` conformance — all straightforward boilerplate that Xcode's template generates.
- `.isPlaceholder(true)` eliminates all manual placeholder work: SwiftUI automatically redacts text and images, producing the standard system placeholder appearance.
- Use `WidgetPreviewContext` in SwiftUI Previews to see the widget at exact Home Screen dimensions during development without building to device.

---
_Source: WWDC20 Session 10034 page (abstract, transcript, and resource links)._
