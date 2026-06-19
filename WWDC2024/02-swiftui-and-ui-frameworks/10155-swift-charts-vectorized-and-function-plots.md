# Swift Charts: Vectorized and Function Plots
**WWDC24 · Session 10155** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10155/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11

## Overview
Swift Charts gains two major new capabilities in 2024: function plots and vectorized plots. Function plots let you render mathematical functions directly—no need to pre-sample data yourself—using new `LinePlot` and `AreaPlot` mark types that accept Swift closures mapping `Double → Double` (or parametric `t → (x, y)`). Vectorized plots introduce parallel-processing-friendly alternatives to the existing mark types (`PointPlot`, `RectanglePlot`, etc.) that accept entire collections and KeyPaths, enabling efficient rendering of large datasets such as heatmaps and scatter plots with thousands of points.

Both additions maintain Swift Charts' existing accessibility guarantees: VoiceOver descriptions and Audio Graphs work out of the box with function plots, with no additional developer code required.

The session uses a solar energy installation dataset from the US Geological Survey as a running example, building up from histograms through normal distribution overlays to a full interactive geospatial scatter plot.

## Key Topics

### Function Plots: LinePlot and AreaPlot
`LinePlot` accepts a closure `(Double) -> Double` and renders a continuous curve. `AreaPlot` fills the area under a curve (one closure returning `Double`) or between two curves (one closure returning `(yStart: Double, yEnd: Double)`). Both support standard chart modifiers (`foregroundStyle`, `opacity`, etc.). The chart's domain is auto-inferred by sampling, but can be overridden with `chartXScale(domain:)` / `chartYScale(domain:)`. The `domain:` parameter on the plot itself restricts the x-range sampled.

### Parametric Functions
`LinePlot(x:y:t:domain:)` accepts a closure `(Double) -> (Double, Double)` to plot parametric curves where both x and y are functions of a third variable `t`.

### Handling Piecewise and Discontinuous Functions
Return `.nan` from a function plot closure to indicate no defined value at that x (e.g., `1/x` at x=0). Swift Charts renders a gap rather than connecting across the discontinuity.

### Vectorized Plots
Vectorized plot APIs (`PointPlot`, `RectanglePlot`, `BarPlot`, `LinePlot` with collection, `AreaPlot` with collection, etc.) accept a collection plus `KeyPath`-based value descriptors instead of per-element closures. Modifiers like `.symbolSize(by:)` and `.foregroundStyle(by:)` accept KeyPaths too. Swift Charts processes all data points with a constant memory offset rather than calling property getters, yielding significant rendering efficiency gains for large, homogeneously styled datasets. Use stored properties (not computed properties) on data model types to maximize performance.

### Best Practices
Use vectorized plots for large, homogeneously styled datasets; use the Mark API (e.g., `ForEach` + `PointMark`) when per-element customization, mixed mark types, or `zIndex` layering is needed. Group data by style to reduce alternation. Pre-specify known style categories and axis domains to avoid extra sampling passes.

## APIs & Frameworks

- `Swift Charts` framework
- `LinePlot(x:y:_:)` **[NEW]** — plots a `(Double) -> Double` function as a continuous line
- `LinePlot(x:y:t:domain:_:)` **[NEW]** — plots a parametric function `(Double) -> (Double, Double)`
- `LinePlot(_:x:y:)` **[NEW]** — vectorized line plot over a collection with KeyPaths
- `AreaPlot(x:y:_:)` **[NEW]** — fills area under a `(Double) -> Double` function
- `AreaPlot(x:yStart:yEnd:_:)` **[NEW]** — fills area between two functions from one closure returning `(yStart:yEnd:)`
- `AreaPlot(x:yStart:yEnd:domain:_:)` **[NEW]** — area plot with restricted sampling domain
- `AreaPlot(_:x:yStart:yEnd:)` **[NEW]** — vectorized area plot over a collection
- `PointPlot(_:x:y:)` **[NEW]** — vectorized point plot over a collection
- `RectanglePlot(_:x:y:)` **[NEW]** — vectorized rectangle plot over a collection
- `BarPlot(_:x:y:)` **[NEW]** — vectorized bar plot over a collection
- `.symbolSize(by:)` — modifier accepting `PlottableValue` with KeyPath for vectorized plots
- `.foregroundStyle(by:)` — modifier accepting `PlottableValue` with KeyPath for vectorized plots
- `.opacity(_:)` — modifier accepting a KeyPath for per-point opacity in vectorized plots
- `chartXScale(domain:)` — constrains chart horizontal axis domain
- `chartYScale(domain:)` — constrains chart vertical axis domain
- `Double.nan` — sentinel value returned from function closures to signal no value
- `BarMark`, `PointMark`, `RectangleMark` — existing mark types (unchanged)
- `PlottableValue.value(_:_:)` — existing value descriptor, now also accepts KeyPaths
- `Chart` — top-level container (unchanged)
- `ForEach` — existing per-element iteration (use when per-element customization is needed)

## Code Highlights

Plot a normal distribution over a histogram:
```swift
Chart {
    ForEach(bins) { bin in
        BarMark(x: .value("Capacity density", bin.range),
                y: .value("Probability", bin.probability))
    }
    LinePlot(x: "Capacity density", y: "Probability") { x in
        normalDistribution(x, mean: mean, standardDeviation: standardDeviation)
    }
    .foregroundStyle(.gray)
}
```

Fill area between two curves with restricted sampling domain:
```swift
AreaPlot(x: "x", yStart: "cos(x)", yEnd: "sin(x)", domain: -135...45) { x in
    (yStart: cos(x / 180 * .pi), yEnd: sin(x / 180 * .pi))
}
.chartXScale(domain: -315...225)
```

Parametric curve:
```swift
LinePlot(x: "x", y: "y", t: "t", domain: -.pi ... .pi) { t in
    let x = sqrt(2) * pow(sin(t), 3)
    let y = cos(t) * (2 - cos(t) - pow(cos(t), 2))
    return (x, y)
}
```

Vectorized point plot with KeyPath modifiers:
```swift
PointPlot(model.data,
          x: .value("Longitude", \.x),
          y: .value("Latitude", \.y))
    .symbolSize(by: .value("Capacity", \.capacity))
    .foregroundStyle(by: .value("Axis type", \.panelAxisType))
```

Return `.nan` for discontinuous functions:
```swift
LinePlot(x: "x", y: "1 / x") { x in
    guard x != 0 else { return .nan }
    return 1 / x
}
```

## Takeaways

- `LinePlot` and `AreaPlot` now render pure mathematical functions (including parametric forms) directly from Swift closures—no data pre-sampling required; VoiceOver and Audio Graph support is automatic.
- Vectorized plot variants (`PointPlot`, `RectanglePlot`, etc.) accept collections and KeyPaths, processing all points with constant memory offset for significantly faster rendering of large, homogeneously styled datasets.
- Use stored properties (not computed properties) on model types for vectorized plots to maximize Swift Charts' memory access efficiency.
- Use `Double.nan` to represent gaps or undefined regions in function plots; use `domain:` on the plot initializer to restrict the x-range sampled without affecting the overall chart scale.

---
_Source: WWDC24 Session 10155 page (abstract, chapter summaries, code samples, and resource links)._
