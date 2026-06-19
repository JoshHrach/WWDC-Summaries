# Go further with Complications in WidgetKit
**WWDC22 · Session 10051** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10051/)

_Platforms:_ watchOS 9, iOS 16

## Overview
watchOS 9 brings WidgetKit to watch face complications, replacing ClockKit's twelve complication families with four unified families. This session covers the watchOS-specific WidgetKit features — the `accessoryCorner` family, the `widgetLabel` modifier, the `showsWidgetLabel` environment value, and the Extra Large face behavior — and walks through how to migrate existing ClockKit complications (and users' existing watch face configurations) to WidgetKit.

Adopting WidgetKit for complications unlocks a single SwiftUI codebase shared between iOS lock screen widgets and watchOS complications, while also automatically migrating users' existing watch face setups when the app updates.

## Key Topics

**Four new widget families for watch faces** — WidgetKit on watchOS adds four families, three of which are shared with the iOS lock screen (`.accessoryRectangular`, `.accessoryCircular`, `.accessoryInline`) and one unique to watchOS: `.accessoryCorner`. The corner family supports two modes: large circular content filling the corner, or small circular content with a curved gauge or label in the corner bezel.

**`widgetLabel` modifier** — A new SwiftUI view modifier that passes auxiliary content to the watch face for rendering in the dial or corner bezel. For `.accessoryCorner`, the label is rendered as the curved content in the corner. For `.accessoryCircular` on the Infograph face, it is rendered as text around the outer bezel. Content types supported in `widgetLabel`: `Text`, `Gauge`, `ProgressView` (for `accessoryCorner`); images and text are extracted for `accessoryInline`.

**`showsWidgetLabel` environment value** — A `Bool` environment value that is `true` when the complication slot displays the `widgetLabel` content (e.g., center-top on Infograph). Use it to conditionally render data-rich content when the bezel is available and a simpler icon when it is not, rather than always showing the same view.

**Extra Large face** — Uses the `.accessoryCircular` family and automatically scales the content up to fill the large face. The content should be identical to the normal circular complication — do not pack in more information just because the canvas is larger.

**`accessoryInline` behavior** — Already acts as a `widgetLabel` itself: the watch face extracts `Image` and `Text` views from the inline content and renders them in the face's own style (flat or curved depending on the face).

**Migrating from ClockKit** — ClockKit's 12 families collapse to 4: Rectangular → `.accessoryRectangular`; Corner → `.accessoryCorner`; Graphic Circular / Modular Small / Extra Large → `.accessoryCircular`; Utilitarian Small Flat / Utilitarian Large → `.accessoryInline`; Utilitarian Small → `.accessoryCorner`. The `CLKComplicationWidgetMigrator` protocol (new) provides migration configurations that let the system automatically upgrade a user's existing watch face complications to WidgetKit when the app updates — with no user interaction required.

## APIs & Frameworks

### WidgetKit

**New families (watchOS 9)**
- `.accessoryCorner` **[NEW]** — watchOS-exclusive; large circle or small circle with curved label/gauge
- `.accessoryCircular` — shared with iOS lock screen; circular complication
- `.accessoryRectangular` — shared with iOS lock screen; rectangular complication
- `.accessoryInline` — shared with iOS lock screen; single line of text + optional icon

**New SwiftUI modifiers / environment**
- `.widgetLabel { content }` **[NEW]** — passes content to watch face for rendering in dial/bezel (supported on `.accessoryCorner` and `.accessoryCircular`)
- `@Environment(\.showsWidgetLabel) var showsWidgetLabel: Bool` **[NEW]** — `true` when the current complication position renders the `widgetLabel` content
- `.widgetAccentable()` — marks a view as accentable (tintable) by the watch face

**Existing (reference)**
- `AccessoryWidgetBackground()` — standard circular background for accessory widgets
- `WidgetConfiguration` — base protocol for widget/complication configurations
- `StaticConfiguration`, `IntentConfiguration` — widget configuration types

### ClockKit (migration APIs — new in watchOS 9)

- `CLKComplicationWidgetMigrator` **[NEW protocol]** — conform your `CLKComplicationDataSource` (or another type) to provide migration mappings
- `CLKComplicationWidgetMigrator.widgetConfiguration(from:) async -> CLKComplicationWidgetMigrationConfiguration?` **[NEW]** — return a static or intent migration config for each `CLKComplicationDescriptor`
- `CLKComplicationStaticWidgetMigrationConfiguration(kind:extensionBundleIdentifier:)` **[NEW]** — migrate to a static WidgetKit configuration
- `CLKComplicationIntentWidgetMigrationConfiguration(kind:extensionBundleIdentifier:intent:localizedDisplayName:)` **[NEW]** — migrate to an intent-based WidgetKit configuration
- `CLKComplicationDataSource.widgetMigrator` **[NEW property]** — return a `CLKComplicationWidgetMigrator` conforming object (can be `self`)

## Code Highlights

`accessoryCorner` with auxiliary gauge in the curved label:
```swift
struct CornerView: View {
    let value: Double
    var body: some View {
        ZStack {
            AccessoryWidgetBackground()
            Image(systemName: "cup.and.saucer.fill")
                .font(.title.bold())
                .widgetAccentable()
        }
        .widgetLabel {
            Gauge(value: value, in: 0...500) {
                Text("MG")
            } currentValueLabel: {
                Text("\(Int(value))")
            } minimumValueLabel: { Text("0") } maximumValueLabel: { Text("500") }
        }
    }
}
```

Adapting `accessoryCircular` using `showsWidgetLabel`:
```swift
@Environment(\.showsWidgetLabel) var showsWidgetLabel

var body: some View {
    if showsWidgetLabel {
        // Bezel available — show icon + text label
        ZStack {
            AccessoryWidgetBackground()
            Image(systemName: "cup.and.saucer.fill").widgetAccentable()
        }
        .widgetLabel { Text("\(value.inMG(), formatter: mgFormatter) Caffeine") }
    } else {
        // No bezel — show gauge with numeric value
        Gauge(value: value, in: 0...500) {
            Text("MG")
        } currentValueLabel: { Text("\(Int(value))") }
        .gaugeStyle(.circular)
    }
}
```

Providing a ClockKit-to-WidgetKit migration mapping:
```swift
// In CLKComplicationDataSource
var widgetMigrator: CLKComplicationWidgetMigrator { self }

func widgetConfiguration(
    from complicationDescriptor: CLKComplicationDescriptor
) async -> CLKComplicationWidgetMigrationConfiguration? {
    CLKComplicationStaticWidgetMigrationConfiguration(
        kind: "CoffeeTracker",
        extensionBundleIdentifier: widgetBundle)
}
```

## Takeaways
- watchOS 9 unifies 12 ClockKit families into 4 WidgetKit families; the `accessoryCorner` family is watchOS-exclusive and supports both large-circle and small-circle-with-label modes via the `widgetLabel` modifier.
- Use `@Environment(\.showsWidgetLabel)` to adapt the complication's content to faces that do and don't show the bezel label, rather than always rendering the same view.
- Implement `CLKComplicationWidgetMigrator` (one new protocol, one async function) to automatically migrate users' existing watch face complications to WidgetKit when the app updates — no user action required.
- Intent-based migration configurations require the SiriKit intent definitions to be included in both the watch app and the widget extension.

---
_Source: WWDC22 Session 10051 page (abstract, transcript, code samples, and resource links)._
