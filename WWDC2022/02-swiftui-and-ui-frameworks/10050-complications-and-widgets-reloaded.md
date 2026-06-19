# Complications and Widgets: Reloaded
**WWDC22 · Session 10050** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10050/)

_Platforms:_ iOS 16, watchOS 9

## Overview
WidgetKit gains four new widget families in 2022, extending its reach to the iOS Lock Screen and replacing ClockKit complications on watchOS 9. The new "accessory" families — `.accessoryCircular`, `.accessoryRectangular`, `.accessoryInline`, and `.accessoryCorner` (watchOS only) — let developers write one widget extension that serves both Lock Screen widgets on iPhone and watch face complications on Apple Watch. This eliminates the need to maintain separate ClockKit and WidgetKit implementations.

The session covers the three rendering modes that accessory widgets can appear in (full color, accented, and vibrant), how to adapt widget content for each mode, new SwiftUI additions like `AccessoryWidgetBackground`, `ViewThatFits` for inline content, and auto-updating `ProgressView` for time-based data. It also covers privacy states — Lock Screen redaction on iOS and always-on display support on watchOS — using `.privacySensitive()` and the `\.isLuminanceReduced` environment value.

## Key Topics

### New Accessory Widget Families
- `.accessoryCircular` — brief info, gauges, progress views; replaces ClockKit `graphicCircular` **[NEW]**
- `.accessoryRectangular` — multiple lines of text or small charts; replaces ClockKit `graphicRectangular` **[NEW]**
- `.accessoryInline` — text-only (with optional image) slot; rendered above the time on iOS Lock Screen and on many watch faces **[NEW]**
- `.accessoryCorner` — watchOS-only; mixes a small widget circle with gauges and text **[NEW]**
- Shared widget extension target for iOS and watchOS — duplicate iOS widget extension target, change to watchOS, embed in watch app
- `#if os(iOS)` / `#if os(watchOS)` platform macros needed to differentiate system families from accessory families in `supportedFamilies`

### Rendering Modes
- `WidgetRenderingMode` — new enum with three cases **[NEW]**
  - `.fullColor` — content displayed as specified; default for colorful watch faces and iPhone when unlocked
  - `.accented` — views split into two groups; each group is flatly colored; mark groups with `.widgetAccentable()` modifier **[NEW]**
  - `.vibrant` — iOS Lock Screen mode; content desaturated then mapped to a material appearance; adapts to Lock Screen background and tint color
- Access current mode via `@Environment(\.widgetRenderingMode) var renderingMode`
- `.widgetAccentable(_ accentable: Bool = true)` — marks a view to receive the accent color in accented rendering mode **[NEW]**
- In vibrant mode: use darker colors/black for less prominent content; avoid transparent colors; light source → opaque/bright; dark source → blurred background

### AccessoryWidgetBackground
- `AccessoryWidgetBackground()` — new SwiftUI view providing a consistent, system-tuned backdrop **[NEW]**
- Renders differently per mode: soft transparent in full color and accented; black (full blur) in vibrant
- Use for circular calendar-style widgets that need a background; most widgets do not need it

### Auto-Updating Time Views
- `ProgressView(interval:countdown:label:currentValueLabel:).progressViewStyle(.circular)` — circular progress view that auto-updates for a date interval without requiring dense timeline entries **[NEW/enhanced]**
- `Text(date, style: .timer)` and `Text(date, style: .relative)` — auto-updating countdowns/relative times that the system updates without timeline reload

### ViewThatFits for Inline Widgets
- `ViewThatFits { ... }` — provide multiple views from longest to shortest; system picks first that fits without truncation **[NEW in SwiftUI]**
- Critical for `.accessoryInline` because the inline slot varies in length across different watch faces
- Present three alternatives: full text → shorter text → icon + time

### Intent Recommendations for watchOS
- `IntentTimelineProvider.recommendations() -> [IntentRecommendation<Intent>]` — required on watchOS; provides preconfigured complication options since watchOS has no widget editing UI **[NEW]**
- Returns `[IntentRecommendation(intent:description:)]`

### Privacy States
- Four states: content shown + normal luminance, content shown + low luminance (always-on), content redacted + normal, content redacted + low luminance
- `@Environment(\.isLuminanceReduced) var isLuminanceReduced` — adapt content for always-on watchOS display **[NEW]**
- Time-relative text and progress views automatically switch to reduced-fidelity mode in low luminance
- `.privacySensitive(_ sensitive: Bool = true)` — mark specific views for redaction in privacy mode; rest of widget remains visible **[NEW]**

## APIs & Frameworks

**WidgetKit** **[NEW]**
- `WidgetFamily.accessoryCircular` **[NEW]**
- `WidgetFamily.accessoryRectangular` **[NEW]**
- `WidgetFamily.accessoryInline` **[NEW]**
- `WidgetFamily.accessoryCorner` (watchOS only) **[NEW]**
- `WidgetRenderingMode` enum — `.fullColor`, `.accented`, `.vibrant` **[NEW]**
- `IntentTimelineProvider.recommendations() -> [IntentRecommendation<Intent>]` **[NEW, required on watchOS]**
- `IntentRecommendation(intent:description:)` **[NEW]**

**SwiftUI — Widget-Specific** **[NEW]**
- `AccessoryWidgetBackground()` — system-tuned widget background view **[NEW]**
- `.widgetAccentable(_ accentable: Bool = true)` — marks view for accent color in accented rendering mode **[NEW]**
- `@Environment(\.widgetRenderingMode) var renderingMode: WidgetRenderingMode` **[NEW]**
- `@Environment(\.isLuminanceReduced) var isLuminanceReduced: Bool` **[NEW]**
- `.privacySensitive(_ sensitive: Bool = true)` — redact view in privacy mode **[NEW]**

**SwiftUI — General** **[NEW]**
- `ViewThatFits(in:) { ... }` — select first view that fits available space **[NEW]**
- `ProgressView(interval:countdown:label:currentValueLabel:)` — auto-updating progress for date intervals **[NEW/enhanced]**
- `Text(date, style: .timer)` / `Text(date, style: .relative)` — system-updated time displays

**Xcode**
- Widget Extension target now available for watchOS (duplicate iOS target, change platform, embed in watch app) **[NEW]**
- `WidgetPreviewContext(family:)` — preview accessory families in Xcode

## Code Highlights

Supporting new families alongside existing ones:
```swift
.supportedFamilies(
    #if os(iOS)
    [.systemSmall, .systemMedium, .accessoryCircular, .accessoryRectangular, .accessoryInline]
    #else
    [.accessoryCircular, .accessoryRectangular, .accessoryInline, .accessoryCorner]
    #endif
)
```

Auto-updating circular progress view:
```swift
case .accessoryCircular:
    ProgressView(interval: entry.character.injuryDate...entry.character.fullHealthDate,
                 countdown: false,
                 label: { Text(entry.character.name) },
                 currentValueLabel: { Avatar(character: entry.character, includeBackground: false) })
    .progressViewStyle(.circular)
```

Rectangular widget with widgetAccentable:
```swift
case .accessoryRectangular:
    HStack(alignment: .center, spacing: 0) {
        VStack(alignment: .leading) {
            Text(entry.character.name)
                .font(.headline)
                .widgetAccentable()
            Text("Level \(entry.character.level)")
            Text(entry.character.fullHealthDate, style: .timer)
        }.frame(maxWidth: .infinity, alignment: .leading)
        Avatar(character: entry.character, includeBackground: false)
    }
```

ViewThatFits for inline widget:
```swift
case .accessoryInline:
    ViewThatFits {
        Text("\(entry.character.name) is resting, combat-ready in \(entry.character.fullHealthDate, style: .relative)")
        Text("\(entry.character.name) ready in \(entry.character.fullHealthDate, style: .timer)")
        Text("\(entry.character.avatar) \(entry.character.fullHealthDate, style: .timer)")
    }
```

Adapting for always-on and privacy:
```swift
@Environment(\.isLuminanceReduced) var isLuminanceReduced

VStack(spacing: -2) {
    Image(systemName: "heart").font(.caption.bold()).widgetAccentable()
    Text("\(currentHeartRate)").font(.title).privacySensitive()
}
```

## Takeaways
- A single WidgetKit extension can now serve both iOS Lock Screen widgets and watchOS complications — add the new accessory families to `supportedFamilies` and use platform macros to exclude system families on watchOS.
- Use `ProgressView(interval:)` and `Text(date, style: .timer)` for time-based data to avoid creating many timeline entries; the system auto-updates these views without a timeline reload.
- Apply `.widgetAccentable()` to the most important view (e.g., title or icon) so it receives the watch face accent color; use `@Environment(\.widgetRenderingMode)` to conditionally swap content that doesn't look good when flattened.
- Provide three variants in `ViewThatFits` for `.accessoryInline` — from most descriptive to most concise — because the inline slot width varies significantly across different watch faces.

---
_Source: WWDC22 Session 10050 page (abstract, transcript, and code samples)._
