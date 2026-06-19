# Explore Pie Charts and Interactivity in Swift Charts
**WWDC23 · Session 10037** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10037/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10, visionOS 1

## Overview
This session introduces three major additions to Swift Charts in iOS 17: pie and donut charts via the new `SectorMark` mark type, value selection via `chartXSelection` and `chartAngleSelection` modifiers (with annotation support for selection popovers), and scrollable charts via `chartScrollableAxes`, `chartXVisibleDomain`, and `chartScrollPosition`. Together these fill the most requested gaps in Swift Charts' initial release.

The session demonstrates all three features through a running food truck sales example, building progressively from a stacked bar chart into a donut chart with centered annotations, a line chart with hover/tap selection and popover annotations with overflow resolution, and a 365-day scrollable bar chart with snapping scroll behavior.

## Key Topics

- **SectorMark and pie charts** — New mark type that represents a slice in a pie chart using polar coordinate space; `angle` parameter specifies the value; values are automatically normalized to 360 degrees; supports `angularInset` (gap between sectors) and `cornerRadius` for rounded sectors.
- **Donut charts** — Created from `SectorMark` by setting `innerRadius` to a ratio (e.g., `.ratio(0.618)`) or a fixed point value; center area available for annotation text using `chartBackground`.
- **chartBackground modifier** — Provides a `ChartProxy` and `GeometryReader` in a background layer; used to position custom content (e.g., centered text) relative to the chart's plot area (`chartProxy.plotAreaFrame`).
- **chartXSelection modifier** — New chart interaction modifier; stores selected X-axis value into a `@Binding`; handles gesture recognition automatically (touch on iOS, hover on macOS); supports custom gestures via `chartGesture`.
- **chartAngleSelection modifier** — Analogous to `chartXSelection` but for polar coordinate marks (pie/donut charts); maps tap/touch to an angular value, enabling sector selection.
- **chartXSelection range** — Variant accepting a range binding; two-finger tap on iOS, drag on macOS; enables range-based selection highlighting.
- **chartGesture modifier** — Provides a `ChartProxy` to a custom gesture; use `proxy.selectXValue(at:)` to drive selection from a custom gesture location.
- **Selection popover pattern** — Combine `chartXSelection` binding → computed `selectedDate` property → conditional `RuleMark` with `.zIndex(-1)` → `.annotation(position:spacing:overflowResolution:)` with a custom SwiftUI view.
- **Annotation overflowResolution** — New parameter on `.annotation()`: `.init(x: .fit(to: .chart), y: .disabled)` clamps annotation horizontally within the chart while allowing vertical overflow.
- **chartScrollableAxes modifier** — Makes one or both chart axes scrollable; `.horizontal` is the most common choice.
- **chartXVisibleDomain modifier** — Sets the length of the visible window as a time interval or count; e.g., 30 days as seconds.
- **chartScrollPosition modifier** — Binds the current scroll position (e.g., a `Date`) for reading or programmatic control.
- **chartScrollTargetBehavior modifier** — Configures snapping behavior using `ScrollTargetBehavior`; `valueAligned(matching:majorAlignment:)` snaps to calendar units and controls swipe-page behavior.

## APIs & Frameworks

**Swift Charts**
- `SectorMark` **[NEW]** — pie/donut chart mark; parameters: `angle`, `innerRadius` (`.ratio(_:)` or `.inset(_:)`), `angularInset`, `cornerRadius`
- `SectorMark.angle` **[NEW]** — `PlottableValue`; specifies the proportion of the full circle
- `SectorMark.innerRadius` **[NEW]** — `.ratio(Double)` or fixed value; creates donut chart when nonzero
- `SectorMark.angularInset` **[NEW]** — gap in points between adjacent sectors
- `.chartBackground(content:)` **[NEW]** — background layer with `ChartProxy` and geometry access; use `chartProxy.plotAreaFrame` for positioning
- `.chartXSelection(value:)` **[NEW]** — binds a selection value on the X axis; gesture-driven
- `.chartXSelection(range:)` **[NEW]** — binds a selected range on the X axis
- `.chartAngleSelection(value:)` **[NEW]** — binds a selection angle for pie/donut charts
- `.chartGesture(_:)` **[NEW]** — provides `ChartProxy` for custom gesture-driven selection; `proxy.selectXValue(at:)` maps a point to a value
- `.annotation(position:spacing:overflowResolution:)` **[NEW parameter]** — `overflowResolution: AnnotationOverflowResolution` controls how annotation stays within chart bounds
- `AnnotationOverflowResolution` **[NEW]** — `.init(x:y:)` with `.fit(to: .chart)` or `.disabled`
- `.chartScrollableAxes(_:)` **[NEW]** — makes chart axes scrollable; takes `Axis.Set` (e.g., `.horizontal`)
- `.chartXVisibleDomain(length:)` **[NEW]** — sets visible window length (time interval as `Double` seconds, or count as `Int`)
- `.chartScrollPosition(x:)` **[NEW]** — binds scroll position to a value (e.g., `Date`)
- `.chartScrollTargetBehavior(_:)` **[NEW]** — configures scroll snapping behavior; `valueAligned(matching:majorAlignment:)` snaps to `DateComponents`
- `ChartProxy` — existing; new method `selectXValue(at:)` **[NEW]** for custom gesture selection
- `RuleMark` — existing; used as selection indicator with `.zIndex(-1)` to stay behind line marks

**SwiftUI**
- `ScrollTargetBehavior` — new SwiftUI type (see "Beyond Scroll Views"); used by `chartScrollTargetBehavior`
- `ValueAlignedScrollTargetBehavior.majorAlignment` — controls swipe-page snapping to `DateComponents`

## Code Highlights

Basic pie chart with sector marks:
```swift
Chart(data, id: \.name) { element in
    SectorMark(
        angle: .value("Sales", element.sales),
        innerRadius: .ratio(0.618),
        angularInset: 1.5
    )
    .cornerRadius(5)
    .foregroundStyle(by: .value("Name", element.name))
}
```

Donut chart with centered label:
```swift
Chart(data, id: \.name) { element in
    SectorMark(angle: .value("Sales", element.sales),
               innerRadius: .ratio(0.618), angularInset: 1.5)
        .cornerRadius(5)
        .foregroundStyle(by: .value("Name", element.name))
}
.chartBackground { chartProxy in
    GeometryReader { geometry in
        let frame = geometry[chartProxy.plotAreaFrame]
        VStack {
            Text("Most Sold Style").font(.callout).foregroundStyle(.secondary)
            Text(mostSold).font(.title2.bold())
        }
        .position(x: frame.midX, y: frame.midY)
    }
}
```

Line chart with value selection and popover annotation:
```swift
Chart {
    // ... LineMark data ...
    if let selectedDate {
        RuleMark(x: .value("Selected", selectedDate, unit: .day))
            .foregroundStyle(Color.gray.opacity(0.3))
            .offset(yStart: -10)
            .zIndex(-1)
            .annotation(position: .top, spacing: 0,
                        overflowResolution: .init(x: .fit(to: .chart), y: .disabled)) {
                valueSelectionPopover
            }
    }
}
.chartXSelection(value: $rawSelectedDate)
```

Scrollable chart with snapping:
```swift
Chart {
    ForEach(SalesData.last365Days, id: \.day) {
        BarMark(x: .value("Day", $0.day, unit: .day), y: .value("Sales", $0.sales))
    }
    .foregroundStyle(.blue)
}
.chartScrollableAxes(.horizontal)
.chartXVisibleDomain(length: 3600 * 24 * 30)
.chartScrollPosition(x: $scrollPosition)
.chartScrollTargetBehavior(
    .valueAligned(matching: DateComponents(hour: 0),
                  majorAlignment: .matching(DateComponents(day: 1))))
```

## Takeaways

- `SectorMark` is the new mark type for pie and donut charts; convert any `BarMark` stacked chart by swapping to `SectorMark` and changing the `x` argument to `angle` — Swift Charts handles normalization automatically.
- Use `chartXSelection(value:)` to add axis-based selection with automatic gesture handling; pair it with a `RuleMark` at `zIndex(-1)` and `.annotation(overflowResolution:)` to create selection popovers that stay within chart bounds.
- `chartScrollableAxes(.horizontal)` combined with `chartXVisibleDomain` and `chartScrollPosition` provides a production-quality scrollable chart in three modifiers; add `chartScrollTargetBehavior` for calendar-aligned snapping.
- `chartAngleSelection` brings the same selection model to pie/donut charts; use the bound angle to drive sector `.opacity` for a clear visual selection state.

---
_Source: WWDC23 Session 10037 page (abstract, chapter summaries, code samples, and resource links)._
