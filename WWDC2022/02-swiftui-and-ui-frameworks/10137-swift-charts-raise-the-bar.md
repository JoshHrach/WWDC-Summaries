# Swift Charts: Raise the Bar
**WWDC22 · Session 10137** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10137/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
Swift Charts is a new Apple framework introduced in iOS 16 that lets developers create beautiful, accessible, and highly customizable data visualizations using a declarative SwiftUI-style API. Like SwiftUI views, charts are composed from lightweight mark primitives that describe what the data means rather than how to draw it — making it easy to iterate on design and support multiple chart types from the same data model.

The framework provides six mark types — `BarMark`, `LineMark`, `PointMark`, `AreaMark`, `RuleMark`, and `RectangleMark` — which can be layered, styled, and combined to produce charts ranging from simple bar charts to complex multi-series annotations. Charts automatically adapt to Dynamic Type, color schemes, device size classes, and VoiceOver, dramatically reducing the accessibility burden on developers.

Session 10137 is the "deep-dive" companion to the introductory "Hello Swift Charts" session (10136). It covers advanced customization of axes, legends, plot area, annotations, and interactive charts using `ChartProxy`.

## Key Topics

### Six Mark Types
Each mark maps data fields to visual properties via `.value("Label", dataField)` modifiers. Marks can be freely combined inside a single `Chart` closure to layer multiple representations.

### Customizing Axes
`chartXAxis` and `chartYAxis` modifiers accept an `AxisMarks` builder closure. Inside, `AxisGridLine`, `AxisTick`, and `AxisValueLabel` give per-axis granular control over gridlines, tick marks, and label formatting.

### Legends and Plot Style
`chartLegend` controls legend visibility and placement. `chartPlotStyle` adjusts the plot area background, padding, and aspect ratio.

### Annotations
The `.annotation` modifier on any mark adds custom SwiftUI views anchored to specific data points or bar segments, enabling callouts, labels, and overlays directly inside the chart.

### Interpolation and Symbol Modifiers
`LineMark` and `AreaMark` support `.interpolationMethod(_:)` for curve style (e.g., `.catmullRom`, `.cardinal`, `.stepCenter`). `PointMark` supports `.symbol(_:)` and `.symbolSize(_:)` for custom scatter-plot symbols.

### Interactive Charts with ChartProxy
`chartOverlay` provides a `ChartProxy` which maps between data values and geometry coordinates, enabling hover, drag, and tap gestures that resolve to data-space positions.

### Accessibility
Swift Charts generates a default accessibility experience automatically. Developers can enhance it with `.accessibilityLabel(_:)` and `.accessibilityValue(_:)` on individual marks.

## APIs & Frameworks

**Swift Charts** (all **[NEW]** in iOS 16 / macOS 13)

_Mark types_
- `Chart` — top-level SwiftUI view
- `BarMark` **[NEW]** — bar chart mark; `.x(...)`, `.y(...)`, `.width(...)` parameters
- `LineMark` **[NEW]** — line chart mark
- `PointMark` **[NEW]** — scatter / dot mark
- `AreaMark` **[NEW]** — filled area under a line
- `RuleMark` **[NEW]** — horizontal or vertical reference line
- `RectangleMark` **[NEW]** — 2-D rectangle for heat-map style charts

_Mark modifiers_
- `.value("Label", dataField)` **[NEW]** — maps a data property to a visual channel
- `.foregroundStyle(_:)` **[NEW]** — maps a categorical field to color/pattern
- `.symbol(_:)` **[NEW]** — sets `PointMark` symbol shape
- `.symbolSize(_:)` **[NEW]** — sets `PointMark` symbol size
- `.interpolationMethod(_:)` **[NEW]** — controls line/area curve interpolation
- `.lineStyle(_:)` **[NEW]** — sets stroke style for `LineMark`
- `.annotation(position:alignment:spacing:content:)` **[NEW]** — attaches a SwiftUI view to a mark

_Axis customization_
- `chartXAxis(_:)` **[NEW]** — chart modifier; accepts `AxisMarks` or `.hidden`
- `chartYAxis(_:)` **[NEW]** — chart modifier
- `AxisMarks(values:content:)` **[NEW]** — builder for axis tick content
- `AxisGridLine` **[NEW]** — draws a gridline at each axis mark
- `AxisTick` **[NEW]** — draws a tick mark
- `AxisValueLabel` **[NEW]** — draws a label; accepts custom `format:` style

_Chart-level modifiers_
- `chartLegend(_:)` **[NEW]** — controls legend visibility/position
- `chartPlotStyle(_:)` **[NEW]** — customizes plot area background and padding
- `chartOverlay(_:)` **[NEW]** — overlays a SwiftUI view with access to `ChartProxy`

_Interaction_
- `ChartProxy` **[NEW]** — converts between SwiftUI geometry and data-space coordinates; `position(for:)`, `value(at:as:)` methods

_Accessibility_
- `.accessibilityLabel(_:)` on marks **[NEW]** — custom VoiceOver label per data element
- `.accessibilityValue(_:)` on marks **[NEW]**

## Code Highlights

A basic bar chart:
```swift
Chart(data) { item in
    BarMark(
        x: .value("Category", item.name),
        y: .value("Value", item.count)
    )
    .foregroundStyle(by: .value("Series", item.series))
}
```

Customizing axes:
```swift
Chart { ... }
.chartXAxis {
    AxisMarks(values: .stride(by: .month)) { value in
        AxisGridLine()
        AxisTick()
        AxisValueLabel(format: .dateTime.month(.abbreviated))
    }
}
```

Adding an annotation to a bar:
```swift
BarMark(x: .value("Name", item.name), y: .value("Sales", item.sales))
    .annotation(position: .top) {
        Text("\(item.sales)")
            .font(.caption)
    }
```

Interactive chart using `ChartProxy`:
```swift
Chart { ... }
.chartOverlay { proxy in
    GeometryReader { geo in
        Rectangle().fill(.clear).contentShape(Rectangle())
            .gesture(DragGesture().onChanged { value in
                let x = value.location.x - geo[proxy.plotAreaFrame].origin.x
                if let date: Date = proxy.value(atX: x) {
                    selectedDate = date
                }
            })
    }
}
```

## Takeaways
- Swift Charts composes six mark primitives — `BarMark`, `LineMark`, `PointMark`, `AreaMark`, `RuleMark`, `RectangleMark` — with SwiftUI modifiers, making chart creation as simple as building a `List`.
- Axis, legend, and plot-area customization is achieved through dedicated chart modifiers (`chartXAxis`, `chartYAxis`, `chartLegend`, `chartPlotStyle`) without any manual drawing code.
- `ChartProxy` bridges chart geometry and data space, enabling rich touch/pointer interactions with just a gesture recognizer.
- Accessibility comes for free by default; `.accessibilityLabel` and `.accessibilityValue` on marks allow per-element VoiceOver customization.

---
_Source: WWDC22 Session 10137 page (abstract, chapter summaries, code samples, and resource links)._
