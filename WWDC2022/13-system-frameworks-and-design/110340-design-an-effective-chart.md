# Design an Effective Chart
**WWDC22 · Session 110340** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110340/)

_Platforms:_ iOS, iPadOS, macOS, watchOS (Swift Charts / Design)

## Overview
This session is the companion design guide to the Swift Charts framework sessions. It walks through a complete chart design process using a food-truck pancake sales app as the running example, covering five design dimensions: marks, axes, descriptions, interaction, and color. At every step, the session pairs visual design decisions with accessibility requirements — particularly VoiceOver navigation, audio graphs, and accessibility labels.

The session's core principle is that an effective chart must be **focused** (communicates a few key insights clearly), **approachable** (quick to understand with context and labels), and **accessible** (usable by everyone, including people with visual impairments or motor differences).

## Key Topics

### 1. Marks — Visual Building Blocks
A **mark** is the fundamental visual element: a bar in a bar chart, a line in a line chart, a point in a scatterplot. Choosing the right mark for your data and goals is the foundation of good chart design.

- **Points**: good for identifying outliers and clusters; poor for showing pattern in noisy data
- **Lines**: good for representing rates of change and trajectories; misleading when large gaps exist between data points
- **Bars**: flexible, intuitive, shows cumulative weight visually; zeros are visible without distraction; best for count/quantity data with potential zero values

Design rule: test mark choices with realistic (messy) data, not idealized smooth data. Design for edge cases (gaps, outliers, zeros).

Accessibility requirement: every visual mark representation must also have a non-visual equivalent. Swift Charts automatically provides customizable VoiceOver accessibility labels per mark and an audio graph implementation.

### 2. Axes — Range and Grid Density
Axes provide references for interpreting mark values.

**Range**:
- **Fixed range**: use when there is a known physical or conceptual bound (e.g., battery 0–100%)
- **Dynamic range**: use when data bounds are open-ended (e.g., step count, sales); always fix the lower bound to 0 for bar charts so bar heights remain proportionally meaningful

**Grid line and label density**:
- Zero grid lines: acceptable for sparkline/preview charts where only pattern matters
- Too few grid lines: range and middle values are unreadable
- Too many grid lines: visually distracting
- Rule of thumb: 3–5 horizontal grid lines; use intuitive interval values (multiples of 20, multiples of 7 days for 30-day charts)

### 3. Descriptions — Context and Take-Aways
Text surrounding a chart is critical for approachability:
- Use a **title** that contextualizes what the chart measures (e.g., "Pancakes Sold" rather than an abstract axis label)
- Add a **summary take-away** in the title or subtitle (e.g., "Total Sales: 1,234 Pancakes") to anchor the reader and highlight the most important insight
- Keep descriptions brief; leverage existing surrounding UI (screen title, segmented controls) to avoid redundancy

Accessibility: audio graphs (via Swift Charts or custom implementation) must describe the x-axis, y-axis, and provide data summaries. These descriptions are especially critical for people for whom examining visual chart details is time-consuming or challenging.

### 4. Interaction — Details on Demand
Interaction deepens exploration and is a critical accessibility surface:
- **Interactive tooltips**: let users touch/tap to highlight a specific data point and see exact values
- **Touch targets**: extend touch targets to the full height/width of the chart area, not just the visual mark; pad generously to make interaction easy even for small marks

Every touch/click interaction must have equivalents for: keyboard navigation, Voice Control, Switch Control, and VoiceOver. Screen changes during interaction (focus indicators, selected-state highlights) must be visually apparent.

**Accessibility labels for marks** — design guidelines:
- Be succinct; do not repeat axis names already described by audio graphs
- Spell out full words, not abbreviations (VoiceOver reads "June" not "Jun")
- Order: context value (date, category) first, data value second — e.g., "May 30th, 41 pancakes"
- For dense charts with too many marks to navigate individually, label sections or ranges instead of individual marks

### 5. Color — Personality and Clarity
Color should **enhance** but not be the **only** means to convey critical information.

Uses of color:
- **Distinguish categories** (activity rings)
- **Communicate intensity** (weather heat maps)
- **Draw attention** by removing color from non-highlighted elements

Multi-series charts: supplement color with additional visual encoding (symbols, shapes) so people with color blindness can distinguish series without relying on color alone. Support the "Differentiate Without Color" system setting.

**Choosing categorical colors**:
- Balance visual weight — unequal saturation/luminosity implies hierarchy
- Pick visually distinct colors (distinguishable by name; high contrast from each other and background)
- Test with color blindness filters
- Colors must adapt to Dark Mode, Light Mode, and Increase Contrast

## APIs & Frameworks

### Swift Charts (Referenced — see Session 10136 for implementation)
- `Chart` view — renders charts from mark declarations **[NEW in iOS 16]**
- `BarMark`, `LineMark`, `PointMark`, `AreaMark`, `RectangleMark`, `RuleMark` **[NEW]** — mark types
- `.accessibilityLabel(_:)` modifier on marks — customizes VoiceOver accessibility label per data point
- Audio graph support — built-in; triggered via VoiceOver's "Audio Graph" action
- `AxisMarks` — controls axis grid lines, labels, and strokes

### VoiceOver / Accessibility APIs
- `UIAccessibility.post(notification:argument:)` — post accessibility notifications for dynamic chart updates
- Audio graph: `AXChart` / `AXChartDescriptor` / `AXDataSeriesDescriptor` — programmatic audio graph support for non-Swift-Charts implementations
- System settings to support: VoiceOver, Switch Control, Voice Control, Increase Contrast, Differentiate Without Color, Dark Mode / Light Mode

## Code Highlights
This is a design session; code samples appear in the companion sessions "Hello Swift Charts" (10136) and "Swift Charts: Raise the Bar" (10137). Key design patterns:

Accessibility label pattern for a bar mark in Swift Charts:
```swift
BarMark(
    x: .value("Date", day.date, unit: .day),
    y: .value("Sales", day.sales)
)
.accessibilityLabel("\(day.date.formatted(.dateTime.month().day())), \(day.sales) pancakes")
```

Multi-series chart with symbol differentiation (color + shape):
```swift
LineMark(x: .value("Date", point.date), y: .value("Sales", point.sales))
    .symbol(by: .value("City", point.city))   // shape distinguishes series
    .foregroundStyle(by: .value("City", point.city))  // color enhances
```

## Takeaways
- Start with a focused goal — identify 2–3 key insights before choosing a mark type; design for realistic messy data, not ideal smooth data.
- Every visual encoding (mark, color, shape) needs a non-visual equivalent: accessibility labels on marks and audio graph descriptions at minimum.
- Axes: use dynamic range for open-ended data; fix the lower bound at 0 for bar charts; choose 3–5 grid lines with intuitive interval values.
- Descriptions: a title that names the metric and a subtitle with the key take-away make a chart immediately approachable for all users.
- Color should add to existing visual differentiation (symbols, shapes), not replace it; test with color blindness filters and ensure adaptation to Dark Mode and Increase Contrast.

---
_Source: WWDC22 Session 110340 page (abstract, transcript, and resource links)._
