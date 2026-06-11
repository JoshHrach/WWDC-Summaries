# Rev up your CarPlay app
**WWDC26 · Session 212** · [Watch](https://developer.apple.com/videos/play/wwdc2026/212/)

_Platforms:_ iOS 27

## Overview
iOS 27 expands CarPlay with several new capabilities across all major app categories. The session covers enhancements to the CarPlay framework for audio and media apps (list improvements, a new MiniPlayer for Now Playing, media thumbnails), a new voice-based conversational app category, video app support for supported vehicles when parked, navigation improvements (greater primary interface control and route sharing with vehicle systems), and an improved CarPlay Simulator for easier development.

The CarPlay framework receives the most API surface area changes. List templates gain new presentation options, and `CPNowPlayingTemplate` gains a `allowsMiniPlayer` toggle and new media information display APIs. Navigation apps benefit from enhanced `CPMapTemplate` primary interface control, and a new route sharing delegate method lets the app coordinate its routing with the vehicle's built-in navigation system.

The addition of video apps and conversational apps as first-class CarPlay categories significantly expands the kinds of experiences possible in the car — video is restricted to parked vehicles in supported cars, and conversational apps provide a hands-free voice-first UI pattern.

## Key Topics

### Apps in CarPlay
Existing categories — audio, navigation, communication, EV charging, fueling, parking, quick food ordering, and driving task apps — are joined by two new categories in iOS 27: **voice-based conversational apps** and **video browsing apps** (parked only, supported vehicles).

### CarPlay Framework Enhancements
- **List improvements**: enhanced list templates with new presentation styles
- **MiniPlayer**: `CPNowPlayingTemplate.shared.allowsMiniPlayer` — new toggle to suppress the compact Now Playing overlay when the full Now Playing template is showing
- **Media information**: richer metadata and thumbnail display on now-playing surfaces
- **Voice control presentation style**: new presentation style for voice-controlled UI flows

### Navigation Apps
- `CPMapTemplate` gains more control over the primary interface area, allowing navigation apps to customize what appears in the map viewport
- **Route sharing**: a new delegate method `mapTemplateShouldProvideRouteSharing(_:)` lets apps opt in to sharing their route data with the vehicle. A trip-level flag `trip.routeSegmentsAvailableForRegion` controls whether segments are offered for a given region.

### CarPlay Simulator
The CarPlay Simulator (a macOS app) makes it easy to connect a Mac running Xcode to simulate a CarPlay environment without needing a physical car head unit.

## APIs & Frameworks

### CarPlay framework
- `CPNowPlayingTemplate.shared` — existing singleton
- `CPNowPlayingTemplate.allowsMiniPlayer` **[NEW]** — Bool; set to `false` to hide the compact MiniPlayer overlay
- `CPListTemplate` — list-based template; enhanced presentation options **[NEW improvements]**
- `CPMapTemplate` — map template for navigation apps; expanded primary interface control **[NEW]**
- `CPMapTemplateDelegate.mapTemplateShouldProvideRouteSharing(_:)` **[NEW]** — return `true` to enable route sharing with the vehicle system
- `CPTrip.routeSegmentsAvailableForRegion` **[NEW]** — Bool property; set to `false` to disable route sharing for a particular trip
- `CPVoiceControlTemplate` / voice control presentation style **[NEW]** — new template presentation mode for conversational apps
- Video app template **[NEW]** — new template type for parked-vehicle video browsing (exact type name not specified in the session page)
- `CPApplicationDelegate` — existing delegate for CarPlay lifecycle

### MediaPlayer / Audio (referenced contextually)
- Thumbnail / artwork display APIs for now playing surfaces (specific types not named in session page)

## Code Highlights

Disable the MiniPlayer in Now Playing:
```swift
CPNowPlayingTemplate.shared.allowsMiniPlayer = false
```

Opt in to route sharing with the vehicle:
```swift
func mapTemplateShouldProvideRouteSharing(_ mapTemplate: CPMapTemplate) -> Bool {
    return true
}
```

Disable route sharing for a specific trip:
```swift
trip.routeSegmentsAvailableForRegion = false
```

## Takeaways
- iOS 27 adds two new CarPlay app categories: voice-based conversational apps and (parked) video apps — requiring entitlement and App Store category declarations.
- `CPNowPlayingTemplate.allowsMiniPlayer = false` lets audio apps suppress the compact overlay when the full Now Playing UI is already showing.
- Navigation apps gain route-sharing coordination with vehicle systems via `mapTemplateShouldProvideRouteSharing` and per-trip `routeSegmentsAvailableForRegion`.
- CarPlay Simulator on Mac removes the need for a physical head unit during development.

---
_Source: WWDC26 Session 212 page (abstract, chapter summaries, code samples, and resource links)._
