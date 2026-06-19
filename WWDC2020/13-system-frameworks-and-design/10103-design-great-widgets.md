# Design Great Widgets
**WWDC20 · Session 10103** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10103/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
Presented by Mac and Christian from Apple's human interface design team, this session covers the complete design process for building excellent widgets in the redesigned iOS 14 widget system. It addresses ideation (what content to surface, how to think about personalization and context), configuration (how editing and multiple widget instances work), and creation (sizes, tap interactions, layout patterns, typography, color, placeholder states, and common mistakes to avoid).

This is the design companion to the WidgetKit engineering sessions: "Meet WidgetKit" (10028), "Build SwiftUI views for widgets" (10033), and the Widget Code-along series.

## Key Topics

**What's New in iOS 14 Widgets**
- Widgets can be placed directly on Home Screen pages (not just the Today view)
- Smart Stacks: multiple widgets stacked in a single home screen slot; the stack rotates automatically based on usage patterns and context
- Drag-and-drop widget editing: users can rearrange and configure widgets directly
- Widget configuration via intents: users can edit widget parameters (e.g., which city for Weather)

**Design Principles: Personal, Informational, Contextual**

The three guiding qualities for great widget content:
- **Personal** — information that creates emotional connection (birthdays in Calendar, featured photos from memories)
- **Informational** — data people repeatedly return to the app for; surfacing it saves app launches
- **Contextual** — adaptive to time, location, or circumstance (Weather shows rain forecast in higher resolution when rain is imminent; Maps shows ETA to next calendar event when driving)

**Ideation Examples from Apple**
- **Calendar**: next event with start time and location is the most important piece of information; collapses when busy; when no events remain today, shows tomorrow's events instead
- **Photos**: surfaces best photos (not just latest); memories from this day in past years; featured photos — not a camera roll browser
- **Weather**: shows current conditions, high/low, hourly forecast; increases hourly resolution during active weather events (rain, thunderstorms)
- **Maps**: shows parking spot when away from home; ETA to next calendar event with route preview; ETA home when not at home

**Configuration and Multiple Instances**
Widgets in iOS 14 support user editing via intents. Example: Weather widget defaults to current location; users can tap to flip the widget and change the location to Tokyo. Users can add multiple instances of the same widget configured differently (e.g., Weather for Tokyo and Weather for San Francisco side by side). This eliminates the need for complex multi-entity layouts — let the user stack their own.

Apps can offer multiple distinct widget types. Stocks offers a watchlist summary widget and a single-stock widget. News offers a topic-based widget and a pinned note widget.

**Sizes and Interactions**

Three required sizes (support any subset):
- **Small** — single tap target; single deep link; max ~4 pieces of information; fills the entire widget with one deep link
- **Medium** — multiple tap targets; multiple deep links; cell-style or content-style tap regions
- **Large** — multiple tap targets; more information; opportunity to include additional context not possible in smaller sizes

Three tap styles:
- **Fill** — covers the entire widget; used for all small widgets and single-content-type mediums/larges
- **Cell** — content lives in a distinct contained shape; tapping a cell deep-links to that item
- **Content** — content is uncontained but tappable; tapping deep-links to that item

Widgets are not mini-apps: no scrolling, no interactive controls (switches, buttons), no video, no live updating UI. They are static snapshots updated on a timeline.

**Personality Through Content and Appearance**
Design widget personality to reflect the app's visual identity:
- Weather: uses familiar weather-condition background gradient and iconography from the app
- News: rich story photography fills the widget
- Calendar: red tint, minimal typography, familiar app aesthetic
- Notes: notepad illustration style from app icon
- Podcasts: purple gradient from app icon
- Tips: yellow gradient from app icon

**Layout Patterns**
Two main approaches observed across Apple's widgets:
1. **Expanding layout**: the same layout scaled to show progressively more detail across sizes (Weather)
2. **Unique layout per size**: each size has a distinct layout optimized for its dimensions (News: small shows rich single image; medium shows multiple stories)

Do not simply scale up the small widget layout to fill the large size. Design each size for its specific information density.

**Layout Specs**
- Default margin: 16 pt on all sides for all sizes
- Graphical inset shapes (circles, platters): 11 pt margins
- Shape corners near widget edges: use `ContainerRelativeShape` in SwiftUI — it automatically matches the widget's corner radius on each device, keeping inner shapes concentric with the widget border

**Typography**
- Use SF Pro, SF Mono, or SF Pro Rounded
- Custom fonts acceptable if they serve brand identity, but widget must still feel native alongside system widgets

**Light and Dark Mode**
Every widget must look great in both light and dark appearance. Strategies:
- Full widget background changes (Calendar)
- Gradient background works in both without change (Podcasts)
- Mixed approach with content adaptation (Notes: white content on dark background in dark mode)

**Placeholder State**
Every widget must provide a placeholder displayed when the system has no data to show (e.g., immediately after install). Show base graphical structure with text areas blocked in using placeholder shapes. Color and layout should match the filled state so the transition from placeholder to live data is smooth without visual shifts.

**Things to Avoid**
- Do not put app icon in a widget
- Do not put app name in a widget (it already appears in the label beneath on the home screen)
- Only use a logo if the app aggregates content from multiple sources; if used, always position in the top-right corner; never use a wordmark
- Do not use instructional text ("tap here to...", "check the app for...")
- Do not use "last updated" or "last checked" language — communicate information, not meta-information about the widget

## APIs & Frameworks

### WidgetKit (new in iOS 14)
- `Widget` — protocol declaring a widget
- `WidgetConfiguration` — base configuration type
- `StaticConfiguration` — fixed widget with no user customization
- `IntentConfiguration` — user-configurable widget backed by a custom `INIntent`
- `WidgetFamily` — `.systemSmall`, `.systemMedium`, `.systemLarge`
- `TimelineProvider` — protocol; provides `TimelineEntry` snapshots and refresh schedule
- `TimelineEntry` — a date-tagged data snapshot for a point in the widget's timeline
- `Timeline<Entry>` — ordered sequence of entries; controls when widget refreshes
- `WidgetBundle` — declares multiple widget types from one extension

### SwiftUI (widget-specific)
- `@main` struct conforming to `Widget` — entry point for a widget
- `Link(destination:)` — deep link tap target within a widget
- `ContainerRelativeShape` **[NEW]** — shape that matches the widget's system-provided corner radius; use for inset shapes near edges
- `widgetURL(_:)` — assigns a deep link URL to the entire widget (for small/single-tap widgets)

### Intents Framework
- Custom `INIntent` with typed parameters — backs `IntentConfiguration` for user-configurable widgets
- `INInteraction.donate(completion:)` — donations power Smart Stack rotation for your widget

## Code Highlights

No code samples were shown in this design session. For implementation see "Meet WidgetKit" (10028) and "Build SwiftUI views for widgets" (10033).

Conceptual widget structure:
```swift
@main
struct WeatherWidget: Widget {
    var body: some WidgetConfiguration {
        IntentConfiguration(
            kind: "WeatherWidget",
            intent: ShowWeatherIntent.self,
            provider: WeatherTimelineProvider()
        ) { entry in
            WeatherWidgetView(entry: entry)
        }
        .configurationDisplayName("Weather")
        .description("Current conditions for a location.")
        .supportedFamilies([.systemSmall, .systemMedium, .systemLarge])
    }
}
```

## Takeaways
- Design for glanceability first: a widget should communicate its most important information in under two seconds without interaction — if it requires study, it's too complex.
- Do not scale up the small widget to fill the large size; design each size independently for its specific information density and user context.
- Use `ContainerRelativeShape` in SwiftUI for any inset shape near widget edges — this automatically keeps corners concentric with the system widget border across all device sizes without hardcoding corner radius values.
- Always provide a placeholder that matches the visual structure of the filled widget (same layout, same colors, blocked text areas) — the transition from placeholder to live data should be invisible.
- Never put the app name, app icon, or "last updated" text in a widget — these are redundant, consume precious space, and violate system design conventions.

---
_Source: WWDC20 Session 10103 page (abstract and full transcript)._
