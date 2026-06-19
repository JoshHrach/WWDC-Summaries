# Turbocharge Your App for CarPlay
**WWDC25 · Session 216** · [Watch](https://developer.apple.com/videos/play/wwdc2025/216/)

_Platforms:_ iOS 26 (CarPlay / CarPlay Ultra)

## Overview
This session covers a broad set of new CarPlay APIs across multiple app categories, plus the new CarPlay Ultra capability for vehicles with ultra-wide displays. Topics include widgets and Live Activities in CarPlay, new list item row element types, a sports mode for Now Playing, enhanced navigation metadata and junction rendering, and multitouch gesture support. Together these APIs enable richer, more contextual in-car experiences.

## Key Topics

### CarPlay Ultra
CarPlay Ultra is Apple's extended CarPlay experience for vehicles with high-resolution ultra-wide dashboard displays. It fills the full display area and supports additional UI surfaces. The session provides guidance on designing for ultra-wide displays while maintaining backward compatibility with standard CarPlay.

### Widgets in CarPlay
CarPlay supports WidgetKit widgets for the first time. Apps can provide widgets that appear in the CarPlay dashboard. Widget families and configuration follow the same WidgetKit APIs used on other platforms, with CarPlay-specific size guidelines.

### Live Activities in CarPlay
ActivityKit Live Activities can now surface in CarPlay via the Dynamic Island equivalent in the car (instrument cluster / dashboard). Workout apps, navigation apps, and any app publishing a Live Activity can have their content displayed while driving.

### CPListImageRowItem Enhancements
`CPListImageRowItem` gains a rich set of new row element types for displaying diverse list content:
- `RowElement` — base element type
- `CardElement` — card-style UI element within a row
- `CondensedElement` — compact representation
- `GridElement` — grid layout within a row item
- `ImageGridElement` — image-focused grid layout

These types allow navigation, media, and EV apps to show rich content in list-based templates without leaving the standard CarPlay template hierarchy.

### CPNowPlayingModeSports
A new now playing mode (`CPNowPlayingModeSports`) designed for live sports audio content. It adds sport-specific metadata (team names, scores, quarter/period) alongside the standard now playing controls. Media apps streaming live sports commentary can adopt this mode to give drivers richer context.

### Navigation Metadata and CPManeuver Updates
- `CPManeuver` gains new properties for richer turn-by-turn instructions, including lane guidance and signage metadata
- New `CPJunctionElement` renders junction/intersection diagrams directly in the navigation template
- Additional navigation metadata properties allow apps to communicate road names, exit numbers, and speed limits to the CarPlay system

### Multitouch Gesture Support
CarPlay templates can now respond to multitouch gestures on compatible vehicle touchscreens. Apps register gesture recognizers against CarPlay's template system, enabling swipe and pinch interactions in appropriate contexts (e.g., map zoom in navigation apps).

## APIs & Frameworks

**CarPlay framework**
- CarPlay Ultra support **[NEW]** — ultra-wide display layout guidance; same APIs, new display surface
- `CPListImageRowItem` row element types **[NEW]**:
  - `RowElement` **[NEW]**
  - `CardElement` **[NEW]**
  - `CondensedElement` **[NEW]**
  - `GridElement` **[NEW]**
  - `ImageGridElement` **[NEW]**
- `CPNowPlayingModeSports` **[NEW]** — sports-specific now playing mode with live score/team metadata
- `CPManeuver` navigation metadata enhancements **[NEW]** — lane guidance, signage, exit numbers, speed limits
- `CPJunctionElement` **[NEW]** — renders junction/intersection diagrams in navigation templates
- Multitouch gesture support **[NEW]** — gesture recognizer registration in CarPlay template context

**WidgetKit**
- CarPlay widget support **[NEW]** — WidgetKit widgets surface in CarPlay dashboard

**ActivityKit**
- Live Activities in CarPlay **[NEW]** — Dynamic Island-equivalent surface in car dashboard/instrument cluster

## Code Highlights

```swift
// Sports Now Playing mode
let nowPlayingTemplate = CPNowPlayingTemplate.shared
nowPlayingTemplate.upNextButtonEnabled = false
// Set mode for sports audio
nowPlayingTemplate.playingMode = .sports(
    CPNowPlayingModeSports(
        homeTeamName: "Sharks",
        awayTeamName: "Jets",
        homeScore: 3,
        awayScore: 2,
        period: "3rd"
    )
)
```

```swift
// Rich list row with CardElement
let cardElement = CPListImageRowItem.CardElement(
    title: "Highway 101 N",
    detailText: "12 min delay",
    image: UIImage(systemName: "car.fill")!
)
let listItem = CPListImageRowItem(text: "Route Options", images: [...])
// Associate cardElement with the item
```

## Takeaways
- Adopt WidgetKit in your CarPlay-enabled app to provide at-a-glance information on the dashboard without requiring a full template interaction.
- Use `CPNowPlayingModeSports` in sports audio streaming apps to display scores and team names alongside playback controls.
- Upgrade list item rows with the new element types (`CardElement`, `GridElement`, `ImageGridElement`) to surface richer content within standard CarPlay templates.
- Implement `CPJunctionElement` and enhanced `CPManeuver` metadata for more informative turn-by-turn navigation instructions.
- Test on CarPlay Ultra simulator configurations to ensure your layout adapts correctly to ultra-wide vehicle displays.

---
_Source: WWDC25 Session 216 page (abstract, chapter summaries, code samples, and resource links)._
