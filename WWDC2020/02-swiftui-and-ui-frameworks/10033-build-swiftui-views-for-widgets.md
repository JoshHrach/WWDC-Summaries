# Build SwiftUI Views for Widgets
**WWDC20 · Session 10033** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10033/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
Widgets in iOS 14 and macOS Big Sur are built entirely in SwiftUI, making them an excellent entry point for SwiftUI even if an app still targets older OS versions. This session walks through constructing a complete widget view from scratch for the "Wired" caffeine-tracking app — covering layout decomposition into subviews, dark mode support via asset catalog color variants, dynamic type adaptation, and placeholder construction. Two new SwiftUI APIs central to widgets are introduced: `ContainerRelativeShape` for concentric corner radii and `Text` date styles for automatically-updating countdowns and timers.

The demo emphasizes practical SwiftUI canvas workflow: using "Embed in VStack/HStack," "Extract Subview," the Attributes Inspector, and live preview to iteratively build a widget without switching to code. The resulting widget supports three families (`.systemSmall`, `.systemMedium`, `.systemLarge`), dark mode, dynamic type, and the placeholder state — all from a single view implementation.

WidgetKit provides the timeline infrastructure (covered in "Meet WidgetKit"), and this session covers only the SwiftUI view layer. All the new APIs introduced here work in regular app views too, not just widgets.

## Key Topics

**Widget Families and WidgetPreviewContext**
Previews use `WidgetPreviewContext(family: .systemSmall)` to render a widget at a specific family size. The `@Environment(\.widgetFamily)` variable lets a view conditionally add content (e.g., a drink photo only on `.systemMedium`). Three families are supported: `.systemSmall`, `.systemMedium`, `.systemLarge`.

**ContainerRelativeShape (New)**
A new shape type that automatically adopts the path and corner radius of the nearest parent container. In a widget, the system defines the container shape; a child `ContainerRelativeShape` produces a rounded rect that is concentric with the widget's own corners — maintaining equal border thickness around curves as padding changes. Eliminates the need to hard-code a corner radius value that may differ across devices or widget families.

**Text Date Styles (New)**
`Text(date, style:)` initializer formats and optionally auto-updates a date relative to now:
- `.date` — "June 3, 2019" (static)
- `.time` — "11:23 PM" (static)
- `.offset` — "+2 hours" / "-3 months" (static signed offset from now)
- `.relative` — "2 hours, 23 minutes" (updates live)
- `.timer` — "36:59:01" (updates live, countdown/up)
- Closed range `date...date` — "9:30 AM – 3:30 PM" (interval)

String interpolation works: `Text("\(date, style: .relative) ago")` produces a localized key automatically.

**Placeholder Support**
`.isPlaceholder(true)` modifier replaces all `Text` content with rounded rectangles and all `Image` content with a fill color, producing a skeleton UI. `.isPlaceholder(false)` on a specific child preserves its real content in the placeholder. The same view is used for both content and placeholder states.

**Layout Construction Pattern**
Recommended SwiftUI widget layout approach:
- Outer `ZStack` for background color
- Inner `VStack` with padding for content
- `Spacer()` between content groups for spacing
- `.leading` alignment on `VStack`s for edge-aligned content
- `HStack` + trailing `Spacer(minLength: 0)` to push content to the leading edge
- Extracted subviews for reusable groups (cheap in SwiftUI)
- Asset catalog colors for automatic Dark Mode adaptation

## APIs & Frameworks

### WidgetKit
- `WidgetPreviewContext(family:)` — sets preview widget family in Xcode canvas
- `WidgetFamily` — `.systemSmall`, `.systemMedium`, `.systemLarge`
- `@Environment(\.widgetFamily) var family: WidgetFamily` — reads current widget family in a view

### SwiftUI — New in iOS 14
- `ContainerRelativeShape` **[NEW]** — shape that matches the nearest container's shape and corner radius; use as `.background(ContainerRelativeShape().fill(color))`
- `Text(_ date: Date, style: Text.DateStyle)` **[NEW]** — date-formatted text with optional live updates
  - `Text.DateStyle.date` — locale-formatted date string (static)
  - `Text.DateStyle.time` — locale-formatted time string (static)
  - `Text.DateStyle.offset` — signed offset from now (static)
  - `Text.DateStyle.relative` — human-readable relative duration (live-updating)
  - `Text.DateStyle.timer` — HH:MM:SS countdown/countup (live-updating)
  - `Text(_ interval: ClosedRange<Date>)` — time interval string
- `.isPlaceholder(_ isPlaceholder: Bool)` **[NEW]** — marks a view (or subtree) as placeholder content; replaces Text/Image with skeleton fills; available in a later seed of Xcode

### SwiftUI — Layout
- `ZStack` — depth layering for background + foreground content
- `VStack(alignment:spacing:)` — vertical stack; use `.leading` alignment
- `HStack(spacing:)` — horizontal stack; add `Spacer(minLength: 0)` to push content to leading edge
- `Spacer(minLength:)` — flexible space; set `minLength: 0` to allow shrinking to zero
- `.padding(_:)` — system-default padding (omit value for context-appropriate defaults)
- `.background(_:)` — applies a shape or color as a view's background
- `.foregroundColor(_:)` — text/icon color, typically driven by asset catalog color sets
- `.font(_:)` — `.body`, `.title`, etc. with `.bold()` modifier
- `.minimumScaleFactor(_:)` — allows text to shrink to prevent clipping

### SwiftUI — Previews
- `PreviewProvider` — protocol for Xcode canvas previews
- `.environment(\.colorScheme, .dark)` — preview dark mode
- `.environment(\.sizeCategory, .extraExtraExtraLarge)` — preview accessibility large text
- Multiple preview groups in a single `PreviewProvider` using `Group { }`

## Code Highlights

Concentric corner radius with `ContainerRelativeShape`:
```swift
struct CaffeineAmountView: View {
    var body: some View {
        Text(caffeineAmount)
            .padding(8)
            .background(
                ContainerRelativeShape()
                    .fill(Color("latte"))
            )
    }
}
```

Live-updating relative time in a widget:
```swift
// In widget view body:
Text("\(entry.drinkDate, style: .relative) ago")
// Automatically updates as time passes; generates localized key for "ago"
```

All `Text` date style variants:
```swift
Text(event.startDate, style: .date)      // "June 3, 2019"
Text(event.startDate, style: .time)      // "11:23 PM"
Text(event.startDate, style: .offset)    // "+2 hours"
Text(event.startDate, style: .relative)  // "2 hours, 23 minutes" (live)
Text(event.startDate, style: .timer)     // "36:59:01" (live)
Text(event.startDate...event.endDate)    // "9:30 AM – 3:30 PM"
```

Multi-family layout with conditional image:
```swift
@Environment(\.widgetFamily) var family

var body: some View {
    HStack {
        VStack(alignment: .leading) { /* existing content */ }
        if family == .systemMedium, let photo = entry.drinkPhoto {
            Image(photo)
                .resizable()
                .aspectRatio(contentMode: .fill)
        }
    }
}
```

## Takeaways
- `ContainerRelativeShape` is the correct way to produce concentric rounded rectangles inside a widget (or any container) — it automatically matches the parent container's corner radius so borders stay geometrically consistent as padding changes.
- `Text(date, style: .relative)` and `Text(date, style: .timer)` are the only SwiftUI APIs that produce live-updating text in a widget without a new timeline entry — use them for "time ago" and countdown displays.
- SwiftUI's adaptive layout, Dynamic Type support, and Dark Mode via asset catalog colors require no conditional code — a single view body handles all variants, making widget views naturally accessible and appearance-adaptive.
- Use `.isPlaceholder(true)` to convert a real widget view into its skeleton placeholder state; add `.isPlaceholder(false)` to specific children (like a title label) to preserve real content in the placeholder.

---
_Source: WWDC20 Session 10033 page (transcript, code samples, and resource links)._
