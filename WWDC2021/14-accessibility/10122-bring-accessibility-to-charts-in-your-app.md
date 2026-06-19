# Bring Accessibility to Charts in Your App
**WWDC21 · Session 10122** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10122/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session covers how to make charts fully accessible to blind and low-vision users, covering both visual design best practices and a new API introduced in iOS 15: Audio Graphs. Audio Graphs are a VoiceOver feature that renders chart data as a continuous tone that rises and falls with the data, allowing users to perceive chart shape, trends, and outliers without seeing the chart at all.

The session also shows how to make chart data perceivable and navigable in VoiceOver by constructing `UIAccessibilityElement` objects for each data point, and how to use the new `AXChart` protocol and `AXChartDescriptor` to drive the full audio graph experience.

## Key Topics

### Visual Design Accessibility
- **High contrast colors**: aim for at least 4.5:1 contrast ratio between foreground and background; use Xcode's Accessibility Inspector Color Contrast Calculator
- **Avoid red-green and blue-yellow color pairings**: red-green colorblindness is most common; blue-yellow is second most common
- **Use symbols in addition to color**: allow data series differentiation without relying on color at all
- Respond to system accessibility settings:
  - `differentiateWithoutColor` — add symbols even if not in default design
  - `isReduceTransparencyEnabled` / `UIAccessibility.isReduceTransparencyEnabled` — disable transparency effects
  - `isIncreaseContrastEnabled` — switch to higher-contrast color palette
  - macOS counterparts available for each setting

### VoiceOver Navigation for Charts
- Set `accessibilityContainerType = .semanticGroup` on the chart view — helps VoiceOver group elements correctly
- Set `accessibilityLabel` to the chart title
- Override `accessibilityElements` to return an array of `UIAccessibilityElement` objects — one per data point (or per interval for large datasets)
- Each element needs: `accessibilityValue` (spoken string describing the data point), `accessibilityFrameInContainerSpace` (position on screen)
- For charts with hundreds/thousands of points: group into intervals instead of one element per point

### Audio Graphs (New in iOS 15)
- New VoiceOver feature accessible via the Audio Graph option in the VoiceOver rotor **[NEW]**
- Plays chart data as a continuous tone; pitch corresponds to Y-axis value (higher pitch = higher value)
- Interactive mode: user double-taps and drags; pausing reads the data value at that position
- Explorer view shows: play button, chart summary text, statistical analysis (shape, trend, outliers)
- Grid line positions are rendered as haptic feedback during playback

### Implementing Audio Graphs: `AXChart` Protocol
- Import `Accessibility` framework
- Conform chart view to `AXChart` protocol **[NEW]**
- Implement `accessibilityChartDescriptor: AXChartDescriptor?` **[NEW]**
- Build descriptor objects:
  - `AXNumericDataAxisDescriptor` (numeric axis) or `AXCategoricalDataAxisDescriptor` (categorical axis) for each axis
  - `AXDataSeriesDescriptor` for each data series
  - `AXDataPoint` for each data point
  - Assemble into `AXChartDescriptor`
- `summary` property on `AXChartDescriptor` — plain-language insight summary (like alt text for charts); shown in explorer view

## APIs & Frameworks

### Accessibility Framework **[NEW]**
- `AXChart` protocol **[NEW]** — protocol for views that contain chart data
  - `accessibilityChartDescriptor: AXChartDescriptor?` **[NEW]** — single required property
- `AXChartDescriptor` **[NEW]** — describes the full chart
  - `title: String`
  - `summary: String?` — plain-language insight summary
  - `xAxis: AXDataAxisDescriptor`
  - `yAxis: AXNumericDataAxisDescriptor`
  - `additionalAxes: [AXDataAxisDescriptor]`
  - `series: [AXDataSeriesDescriptor]`
- `AXNumericDataAxisDescriptor` **[NEW]** — numeric axis
  - `title: String`, `range: ClosedRange<Double>`, `gridlinePositions: [Double]`
  - `valueDescriptionProvider: (Double) -> String` — localized spoken value formatter
- `AXCategoricalDataAxisDescriptor` **[NEW]** — categorical axis
- `AXDataSeriesDescriptor` **[NEW]** — one data series
  - `name: String`, `isContinuous: Bool`, `dataPoints: [AXDataPoint]`
  - `isContinuous: true` for line charts, `false` for bar/scatter
- `AXDataPoint` **[NEW]** — individual data point
  - `xValue: AXDataPoint.Value`, `yValue: AXDataPoint.Value?`

### UIKit Accessibility
- `UIAccessibilityContainerType.semanticGroup` — chart container type
- `UIAccessibilityElement(accessibilityContainer:)` — create a per-data-point element
  - `accessibilityValue: String?` — spoken value description
  - `accessibilityFrameInContainerSpace: CGRect` — position for VoiceOver tap navigation
- `UIAccessibility.isReduceTransparencyEnabled`, `.isIncreaseContrastEnabled`, `.isDifferentiateWithoutColorEnabled`

### SwiftUI Accessibility
- `accessibilityReduceMotion` — environment value for reduce motion
- `differentiateWithoutColor` — environment value

## Code Highlights

Making chart elements navigable in VoiceOver:
```swift
extension ChartView {
    override var accessibilityContainerType: UIAccessibilityContainerType { .semanticGroup }
    override var accessibilityLabel: String? { model.title }
    override var accessibilityElements: [Any]? {
        get {
            return model.dataPoints.map { point in
                let element = UIAccessibilityElement(accessibilityContainer: self)
                element.accessibilityValue = "\(point.x) cups, \(point.y) lines of code"
                element.accessibilityFrameInContainerSpace = frameRect(for: point)
                return element
            }
        }
        set {}
    }
}
```

Implementing Audio Graph support:
```swift
import Accessibility

extension ChartView: AXChart {
    var accessibilityChartDescriptor: AXChartDescriptor? {
        get {
            let xAxis = AXNumericDataAxisDescriptor(
                title: model.xAxis.title,
                range: model.xAxis.range,
                gridlinePositions: [],
                valueDescriptionProvider: { value in "\(value) cups" }
            )
            let yAxis = AXNumericDataAxisDescriptor(
                title: model.yAxis.title,
                range: model.yAxis.range,
                gridlinePositions: [],
                valueDescriptionProvider: { value in "\(value) lines of code" }
            )
            let dataPoints = model.data.map { AXDataPoint(x: .number($0.x), y: .number($0.y)) }
            let series = AXDataSeriesDescriptor(name: "Data", isContinuous: true, dataPoints: dataPoints)
            return AXChartDescriptor(title: model.title, summary: model.summary,
                                     xAxis: xAxis, yAxis: yAxis,
                                     additionalAxes: [], series: [series])
        }
        set {}
    }
}
```

## Takeaways
- Audio Graphs (iOS 15) give blind and low-vision users the same quick, high-level understanding of chart shape, trends, and outliers that sighted users get at a glance — implement `AXChart` and `AXChartDescriptor` for any chart view.
- The `summary` string on `AXChartDescriptor` is the chart equivalent of image alt text — write two sentences capturing the most important insight from the data.
- Always make data navigable element-by-element via `UIAccessibilityElement`; for large datasets, group into intervals rather than one element per point.
- Visual accessibility improvements (contrast, symbols alongside color, accessibility-setting responses) benefit all users, not just those with vision impairments.

---
_Source: WWDC21 Session 10122 page (abstract, chapter summaries, code samples, and resource links)._
