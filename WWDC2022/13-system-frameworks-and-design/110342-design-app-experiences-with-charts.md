# Design App Experiences with Charts
**WWDC22 · Session 110342** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110342/)

_Platforms:_ iOS, iPadOS, macOS, watchOS (Swift Charts / Design)

## Overview
This session is the first step in the WWDC22 chart design series. Where the companion session "Design an Effective Chart" (110340) focuses on designing a single chart well, this session addresses the broader question: when should you use charts in your app, how should individual charts function at different sizes, and how do multiple charts work together as a coherent design system?

The session uses a food-truck pancake sales app as a running example, walking from an app with only a transaction list to a fully charted app with three distinct charts, each serving a different decision-making goal for the truck operator.

## Key Topics

### When to Use Charts
Charts are justified when data genuinely benefits from visual communication. Three common scenarios:
1. **Change over time** — historical or predicted values where trend, trajectory, and fluctuation matter
2. **Part of a whole** — progress toward a goal, completion percentage, quota fill
3. **Comparison** — evaluating items or categories against each other

Before adding a chart, ask: what are the core goals of the app, and how does this chart support them? Charts should direct attention to the most important information — use them intentionally, not exhaustively. The food truck app identified three priority insights: recent sales trend, item popularity, and best sales location by day of week.

### How to Use Charts: Descriptions
Every chart needs a description. A title alone ("Sales in the Past 30 Days") labels the chart but communicates nothing. Descriptions should:
- **Be self-contained** — informative even when read in isolation
- **Include a key take-away** — e.g., "Total Sales: 1,234 Pancakes" summarizes the most critical insight
- **Interpret the data when useful** — e.g., "Sales for the past 30 days are up 12%, totaling 1,234 pancakes" helps readers understand whether a value is high or low

### How to Use Charts: Data Perspectives
Look beyond the summary. Rich charts provide insights at three scales:
- **Macro** — totals, averages, overall trends across the full data set
- **Medium** — sub-sets by time (weekdays vs. weekends, time of day) or category (item type, location)
- **Micro** — individual records (last transaction, largest single-day sale, outliers)

Tappable annotation rows below a chart can let users toggle between perspectives, updating the chart overlay without leaving the screen.

### How to Use Charts: Size and Interactivity
Chart size and interactivity are coupled. Three tiers:
1. **Small static charts** (sparklines, Watch complications, trend platters) — pattern only; no grid lines, labels, or interactivity; act as entry points to larger charts; touching leads to a detail view
2. **Medium interactive charts** (full-width, not full-height) — include axis labels and grid lines; support touch interaction for precise values; allow toggling time range
3. **Large deep-dive charts** (full width, tall) — maximum detail; support all interaction modes; progressive disclosure of complexity

Key principle: **progressive revelation** — introduce chart complexity gradually. Use a small static chart higher in the navigation hierarchy; a tap reveals an expanded version. When expanding, maintain **continuity**: preserve values, context, and visual shape. Never show something different when the user expects to see more of what they already see.

### Chart Design Systems
When an app has multiple charts, they form a design system. Principles:

**Use familiar forms first** — bar charts and line charts are widely understood; scatter plots or custom forms require explicit onboarding. Novel chart forms should be central to the app (like Activity rings), not supplementary.

**Make differences intentional and visible** — small changes between charts (time scope) may need only a description update; larger changes (different data type) require a visual distinction like different color; major changes (different metric and form) need distinct styling. Users interpret similarity as equivalence; intentional differences signal that something has changed.

**Differentiation toolkit**:
- Description / title change
- Bar color change (different color per topic or category)
- Orientation change (vertical → horizontal bars distinguish item comparison from time series)
- Mark style change (bars → lines to shift from magnitude to trend focus)

## APIs & Frameworks

### Swift Charts (Referenced — see Session 10136 for implementation)
- `Chart` view **[NEW in iOS 16]**
- `BarMark`, `LineMark`, `PointMark` **[NEW]** — mark types used in examples
- `.foregroundStyle(by:)` modifier — drives per-category color differentiation
- `AxisMarks` — configures grid lines and labels for interactive charts
- Audio graph support — built-in per-chart accessibility sonification

### Related Design Frameworks (Informational)
- `SwiftUI` — hosts `Chart` views within the app's navigation hierarchy
- Human Interface Guidelines — chart design guidance consistent with this session

## Code Highlights
This is a design/strategy session; no code samples appear. Companion sessions with code:
- "Hello Swift Charts" (10136) — introduction to the Swift Charts API
- "Swift Charts: Raise the Bar" (10137) — advanced customization

Design pattern illustrated — small static chart linking to large interactive chart:
```swift
// In a summary tab row
NavigationLink {
    SalesDetailView()  // large interactive chart
} label: {
    SalesPlatterView() // small static chart preview
}
```

Design pattern illustrated — tappable annotation rows to toggle chart perspectives:
```swift
// Each row updates @State selectedPerspective
ForEach(perspectives) { perspective in
    Button(perspective.label) { selectedPerspective = perspective }
}
Chart(filteredData(for: selectedPerspective)) { ... }
```

## Takeaways
- Only add a chart when it serves the core goals of the app; charts should direct attention to the most important information, not display everything available.
- Every chart description should be self-contained and include a take-away; interpretive descriptions ("up 12%") are more valuable than labels alone.
- Match chart size to its purpose: small static charts for scanning and navigation entry points; medium interactive charts for exploration; large charts for deep analysis.
- Always maintain visual continuity when transitioning from a preview chart to a detail chart — users expect more of the same, not something different.
- In a multi-chart app, make differences between charts intentional and visible (color, orientation, mark style) so users immediately recognize when they are looking at distinct information.

---
_Source: WWDC22 Session 110342 page (abstract, transcript, and resource links)._
