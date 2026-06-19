# Hello Swift Charts
**WWDC22 · Session 10136** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10136/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
Swift Charts is Apple's new declarative framework for creating informative, beautiful, and accessible charts entirely within SwiftUI. It uses the same compositional syntax developers already know from SwiftUI, making it easy to build bar charts, line charts, scatter plots, and more with minimal code.

The framework is built around the concept of "marks" — visual elements that represent data items — and "mark properties" such as position, foreground style, and symbol shape. By composing marks and properties, developers can iterate through many chart designs quickly and adapt charts to their app's unique visual style.

Swift Charts automatically handles Dark Mode, Dynamic Type, different device sizes and orientations, VoiceOver accessibility (exposing data to screen readers), and Audio Graphs — all without extra code from the developer.

## Key Topics
- **Declarative chart construction** — using `Chart`, `BarMark`, `LineMark`, and `PointMark` with `.value()` factory methods for x and y axes
- **Data-driven charts** — driving chart marks from arrays of model structs via `ForEach` or the chart initializer directly
- **Multi-series charts** — using `foregroundStyle(by:)` and `symbol(by:)` to distinguish multiple data series with color and shape
- **Mark composition** — combining multiple mark types (e.g., `LineMark` + `PointMark`) in a single chart; applying the `.symbol` modifier to `LineMark` as a shorthand
- **Iterating designs quickly** — swapping mark types (Bar → Point → Line) with a single line change to explore different visualizations
- **Animations** — Swift Charts works natively with SwiftUI animations (e.g., `easeInOut` when toggling datasets)
- **Accessibility** — built-in VoiceOver support reads axis labels and values; Audio Graphs provide sonification of chart data
- **Cross-platform** — same chart code and customizations work across all Apple platforms

## APIs & Frameworks
**Swift Charts (new framework)** **[NEW]**
- `Chart` **[NEW]** — top-level SwiftUI view that hosts mark content
- `Chart(_:content:)` **[NEW]** — initializer that accepts a data collection (acts like `ForEach`)
- `BarMark` **[NEW]** — mark type for bar charts
- `LineMark` **[NEW]** — mark type for line charts
- `PointMark` **[NEW]** — mark type for scatter/point charts
- `.value(_:_:)` **[NEW]** — factory method on `PlottableValue` used to bind a mark property to a data value with a description label
- `.foregroundStyle(by:)` **[NEW]** — mark modifier to map a data dimension to color, creating a legend automatically
- `.symbol(by:)` **[NEW]** — mark modifier to map a data dimension to point symbol shape
- `.symbol(_:)` applied to `LineMark` **[NEW]** — shorthand to add point symbols along a line
- `ForEach` inside `Chart` — standard SwiftUI `ForEach` works within chart content

**SwiftUI (used alongside Swift Charts)**
- `@State` — for toggling between data sets
- `Picker` — standard SwiftUI picker integrated with chart state
- `withAnimation(.easeInOut)` — animating chart transitions
- Xcode Previews variant feature — viewing charts in Dark Mode, different font sizes, device sizes, orientations simultaneously

**Accessibility**
- VoiceOver — automatically reads chart marks with label and value
- Audio Graphs — sonification support (introduced in 2021, automatically supported by Swift Charts)
- High-Contrast mode support — built-in

## Code Highlights
Minimal bar chart from a data array:
```swift
Chart(data) { element in
    BarMark(
        x: .value("Name", element.name),
        y: .value("Sales", element.sales)
    )
}
```

Multi-series line chart with color and symbol differentiation:
```swift
Chart(seriesData) { series in
    ForEach(series.sales) { element in
        LineMark(
            x: .value("Day", element.day),
            y: .value("Sales", element.sales)
        )
    }
    .foregroundStyle(by: .value("City", series.city))
    .symbol(by: .value("City", series.city))
}
```

Transposing a bar chart (horizontal bars) by swapping x and y:
```swift
BarMark(
    x: .value("Sales", element.sales),
    y: .value("Name", element.name)
)
```

## Takeaways
- Swift Charts brings Apple-designed, fully accessible charts to SwiftUI with a declarative, compositional API that mirrors SwiftUI's own patterns.
- Switching chart types is as simple as changing `BarMark` to `LineMark` or `PointMark` — making rapid design iteration very fast.
- Accessibility (VoiceOver + Audio Graphs), Dark Mode, Dynamic Type, and multi-platform support are all provided automatically at no extra cost.
- The framework is extensible: developers can create custom marks beyond the built-in set.

---
_Source: WWDC22 Session 10136 page (abstract, chapter summaries, code samples, and resource links)._
