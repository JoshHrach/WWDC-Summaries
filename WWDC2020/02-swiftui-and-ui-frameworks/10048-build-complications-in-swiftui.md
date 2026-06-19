# Build Complications in SwiftUI
**WWDC20 · Session 10048** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10048/)

_Platforms:_ watchOS 7

## Overview
watchOS 7 introduces the ability to build Apple Watch complications using SwiftUI views, allowing developers to bring the full power of SwiftUI's drawing and layout system directly to the watch face. New ClockKit templates accept SwiftUI views, including a brand-new `CLKComplicationTemplateGraphicRectangularFullView` that provides a larger canvas than previous templates.

Two new SwiftUI controls — `ProgressView` and `Gauge` — are introduced specifically to complement complication development, though they are usable anywhere in a SwiftUI app. The session also covers the two watch face tinting modes (desaturated and color-opacity), how to control tinting behavior using `complicationForeground` and the `ComplicationRenderingMode` environment value, and how to preview complications in Xcode 12 across multiple faces and tint colors simultaneously.

Best practices are discussed including performance constraints, safe areas for the Rectangular Full View, and the fact that interactive elements (buttons, gestures, animations) are not supported in complications.

## Key Topics

**New SwiftUI Complication Templates in ClockKit**
New templates accept a SwiftUI view for the Graphic Corner, Circular, Rectangular, and Extra Large families. The `CLKComplicationTemplateGraphicRectangularFullView` **[NEW]** provides a full-canvas drawing area for the Rectangular family.

**Text Enhancements for Complications**
`Text` is now complication-family-aware: font size adapts per family, and the default font is SF Rounded. New text date styles — `.relative`, `.offset`, and `.timer` — are automatically kept up to date by the watch face without requiring a new timeline entry.

**ProgressView**
New SwiftUI control for linear progress. Supports `CircularProgressViewStyle` and `LinearProgressViewStyle`, both accepting a `tint` color. Accepts a label view.

**Gauge**
New SwiftUI control for values that vary over a range. Supports `CircularGaugeStyle` and `LinearGaugeStyle`, with optional `currentValueLabel`, `minimumValueLabel`, and `maximumValueLabel` closures. Tintable with a solid `Color` or a `Gradient`.

**Watch Face Tinting**
- Desaturated tinting (default): view is converted to grayscale; some faces apply a single color wash.
- Color-opacity tinting: views are split into layers by opacity; the watch face colors each layer independently. Opt in by applying `.complicationForeground()` to views that should be promoted to the foreground layer.
- `ComplicationRenderingMode` environment value (`.fullColor` / `.tinted`) allows conditional rendering for fine-grained control.

**Xcode 12 Previews**
`CLKComplicationTemplate.previewContext()` wraps a template in a SwiftUI view suitable for Xcode Previews, showing it on the appropriate watch face. `CLKComplicationTemplate.PreviewFaceColor` can be enumerated to preview all tint colors simultaneously.

## APIs & Frameworks

### ClockKit
- `CLKComplicationTemplateGraphicRectangularFullView` **[NEW]** — full-canvas SwiftUI view template for Graphic Rectangular family
- `CLKComplicationTemplateGraphicCircularView` **[NEW]** — SwiftUI view template for Graphic Circular
- `CLKComplicationTemplateGraphicCornerCircularView` **[NEW]** — SwiftUI view for Graphic Corner
- `CLKComplicationTemplateGraphicExtraLargeCircularView` **[NEW]** — SwiftUI view for Extra Large
- `CLKComplicationTemplate.previewContext()` **[NEW]** — wraps template in a Previews-compatible SwiftUI view
- `CLKComplicationTemplate.PreviewFaceColor` **[NEW]** — enumerable face color options for preview

### SwiftUI (watchOS 7 additions)
- `ProgressView(value:)` **[NEW]** — progress indicator control
- `CircularProgressViewStyle(tint:)` **[NEW]** — circular ring progress style
- `LinearProgressViewStyle(tint:)` **[NEW]** — linear bar progress style
- `Gauge(value:in:label:currentValueLabel:minimumValueLabel:maximumValueLabel:)` **[NEW]** — range gauge control
- `CircularGaugeStyle(tint:)` **[NEW]** — circular gauge style; accepts `Color` or `Gradient`
- `LinearGaugeStyle(tint:)` **[NEW]** — linear gauge style; accepts `Color` or `Gradient`
- `Text(_:style:)` with date styles: `.relative`, `.offset`, `.timer` **[NEW]** — auto-updating watch face text
- `.complicationForeground()` modifier **[NEW]** — promotes view to the foreground tinting layer
- `ComplicationRenderingMode` **[NEW]** — environment enum: `.fullColor`, `.tinted`
- `@Environment(\.complicationRenderingMode)` **[NEW]** — reads the current rendering mode
- `.edgesIgnoringSafeArea(_:)` — use with Rectangular Full View to access full canvas
- `Gradient(colors:)` — color gradient for gauge/progress tinting
- `Label(_:systemImage:)`, `Image(systemName:)` — SF Symbol usage in complications

## Code Highlights

Gauge with gradient tint and all labels:
```swift
Gauge(value: acidity, in: 3...10) {
    Image(systemName: "drop.fill").foregroundColor(.green)
} currentValueLabel: {
    Text("\(acidity, specifier: "%.1f")")
} minimumValueLabel: { Text("3") } maximumValueLabel: { Text("10") }
.gaugeStyle(CircularGaugeStyle(
    tint: Gradient(colors: [.orange, .yellow, .green, .blue, .purple])
))
```

Auto-updating timer text in a complication:
```swift
Text("Time remaining: \(Date() + 100, style: .timer)")
```

Previewing a complication template in Xcode:
```swift
CLKComplicationTemplateGraphicRectangularFullView(MyView())
    .previewContext()
```

Color-opacity tinting with `complicationForeground`:
```swift
ZStack {
    Circle().fill(Color.blue)           // background layer
    Image("apple").complicationForeground()  // foreground layer
}
```

Conditional rendering based on tinting mode:
```swift
@Environment(\.complicationRenderingMode) var renderingMode
// ...
switch renderingMode {
case .fullColor: Circle().fill(Color.blue)
case .tinted:    Circle().fill(LinearGradient(...))
}
```

## Takeaways
- SwiftUI complications in watchOS 7 let you use the full SwiftUI drawing library on the watch face; `CLKComplicationTemplateGraphicRectangularFullView` provides the largest canvas yet.
- `ProgressView` and `Gauge` are new SwiftUI controls introduced for complications but usable anywhere; `Gauge` supports gradient tinting and four configurable label slots.
- By default, complications are desaturated on tinted faces; apply `.complicationForeground()` to opt into color-opacity tinting and create distinct color layers.
- Complications must be static SwiftUI views — no animations, buttons, or gestures — and poorly performing views (blurs, oversized images) incur runtime penalties that can restrict your complication's update budget.

---
_Source: WWDC20 Session 10048 page (abstract, chapter summaries, code samples, and resource links)._
