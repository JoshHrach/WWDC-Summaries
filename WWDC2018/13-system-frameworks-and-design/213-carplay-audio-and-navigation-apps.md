# CarPlay Audio and Navigation Apps
**WWDC18 · Session 213** · [Watch](https://developer.apple.com/videos/play/wwdc2018/213/)

_Platforms:_ iOS 12

## Overview
This session covers two major areas of CarPlay development. The first is audio app support via the existing `MPPlayableContent` APIs, with performance improvements in iOS 12 and best practices for real-world connectivity and device-lock scenarios. The second — and the headline announcement — is the brand-new CarPlay framework (`CarPlay`) introduced in iOS 12, giving navigation apps a template-based system to display maps, provide destination search, show route previews, and deliver turn-by-turn guidance on the car screen.

CarPlay presents an app on the vehicle's built-in display as if it were a second external screen, abstracting away differences in touchscreen, rotary knob, and touchpad input styles, screen sizes, aspect ratios, and right-hand vs. left-hand drive orientations. Navigation apps draw their map tiles into a provided window; the CarPlay framework overlays template-based UI on top.

## Key Topics

### CarPlay Audio Apps (MPPlayableContent)
- Apps use `MPPlayableContent`, `MPNowPlayingInfoCenter`, and `MPRemoteCommandCenter` — same as iOS 12 Lock Screen / Control Center integration.
- iOS 12 "remastered" `MPPlayableContent`: faster startup sequence, smoother animations, better prediction of content the user wants.
- Avoid calling `reloadData()` on every change; use `beginUpdates()` / `endUpdates()` for targeted content updates.
- Cache content internally so data source callbacks return quickly (data source calls are asynchronous).
- `beginLoadingChildItems(at:in:)` — called when an index path becomes visible on the car screen; use to prefetch content before the user taps.
- Real-world considerations: users often drive with phone locked (data protection) and in areas with poor connectivity; audit data protection file policies to allow background access.
- Provide a meaningful UI even when logged out — CarPlay will time out if no content is provided.

### CarPlay Framework — Navigation Apps (New in iOS 12)
- New `CarPlay` framework; navigation apps must apply for a navigation entitlement.
- App provides a window for map tile rendering; the CarPlay framework overlays template UI on top.
- `CPApplicationDelegate` protocol — app delegate must conform; key method: `application(_:didConnectCarInterfaceController:to:)`.
- `CPInterfaceController` — manages template stack (push/pop/set root); analogous to `UINavigationController`.
- App receives a `CPWindow` to render map tiles into.
- `UIView.safeAreaInsets` / `safeAreaInsetsDidChange()` — CarPlay updates safe area insets as template content changes; app must draw within them.

### CarPlay Templates
- `CPMapTemplate` — transparent overlay showing map window behind; hosts navigation bar buttons, map buttons, pan mode, navigation alerts, route preview, and turn-by-turn guidance card.
- `CPBarButton` — text or image button for map template navigation bar (up to 4: 2 leading, 2 trailing); system-styled.
- `CPMapButton` — image button above map window (up to 4); app-styled.
- `CPGridTemplate` — grid of up to 8 `CPGridButton` items; supports navigation bar with title and buttons.
- `CPListTemplate` — list of `CPListItem` objects in one or more sections; supports navigation bar with title and buttons; shows system scroll bar.
- `CPSearchTemplate` — text search with touchscreen keyboard or linear keyboard (for rotary-knob vehicles); touchpad character-recognition input translated automatically.
- `CPAlertTemplate` — `.actionSheet` or `.fullScreen` styles for important alerts.
- Voice control template (for voice-guided input).
- `CPNavigationAlert` — banner shown on the map template; configurable with title, subtitle, image, primary/secondary actions, and optional auto-dismiss interval.

### Route Preview and Turn-by-Turn Guidance
- `CPTrip` — wraps origin, destination, and an array of `CPRouteChoice` objects.
- `CPRouteChoice` — a single route option with additional details and advisory notices.
- `CPTravelEstimates` — ETA and distance, associated with a trip or individual maneuver.
- `CPMapTemplate.showTripPreviews(_:textConfiguration:)` — displays route preview UI including "More Routes" button for multiple choices.
- `CPMapTemplateDelegate.mapTemplate(_:selectedPreviewFor:using:)` — called when user toggles between routes; app updates map tile content.
- `CPMapTemplateDelegate.mapTemplate(_:startedTrip:using:)` — user tapped Go; begin navigation.
- `CPMapTemplate.startNavigationSession(for:)` → `CPNavigationSession` — represents the active guidance session.
- `CPNavigationSession.upcomingManeuvers` — set an array of `CPManeuver` objects (primary + optional secondary shown in guidance card).
- `CPManeuver` — turn instruction image, initial travel estimates.
- `CPNavigationSession.updateEstimates(_:for:)` — update ETA and distance for a maneuver in real time.
- `CPNavigationSession.pauseNavigation(for:)` — set pause reason (e.g., rerouting).

### Background Notifications
- When app is backgrounded, banner notifications keep users informed of maneuvers and alerts.
- `CPMapTemplateDelegate.mapTemplate(_:shouldShowNotificationFor:)` — return `true` to promote a new maneuver to a banner.
- `CPMapTemplateDelegate.mapTemplate(_:shouldUpdateNotificationFor:with:)` — update existing banner with new estimates instead of posting a new one.
- `CPMapTemplateDelegate.mapTemplate(_:shouldShowNotificationFor:)` (navigationAlert variant) — show navigation alerts as banners.

### Audio Session for Navigation Prompts
- Configure `AVAudioSession` with `mode: .voicePrompt` **[NEW]** and category options `.duckOthers` and `.interruptSpokenAudioAndMixWithOthers` to properly duck both iOS audio sources and car audio (FM radio, etc.).

## APIs & Frameworks

**CarPlay (new framework, iOS 12)**
- `CPApplicationDelegate` protocol **[NEW]** — `application(_:didConnectCarInterfaceController:to:)`, `application(_:didDisconnectCarInterfaceController:from:)`
- `CPInterfaceController` **[NEW]** — `setRootTemplate(_:animated:)`, `pushTemplate(_:animated:)`, `popTemplate(animated:)`, `popToRootTemplate(animated:)`
- `CPWindow` **[NEW]** — `rootViewController`
- `CPMapTemplate` **[NEW]** — `leadingNavigationBarButtons`, `trailingNavigationBarButtons`, `mapButtons`, `showTripPreviews(_:textConfiguration:)`, `startNavigationSession(for:)`, `presentNavigationAlert(_:animated:)`, `dismissNavigationAlert(animated:completion:)`
- `CPMapTemplateDelegate` **[NEW]**
- `CPBarButton` **[NEW]** — `.text`, `.image` styles
- `CPMapButton` **[NEW]**
- `CPGridTemplate` **[NEW]**
- `CPGridButton` **[NEW]**
- `CPListTemplate` **[NEW]**
- `CPListItem` **[NEW]**
- `CPListTemplateDelegate` **[NEW]** — `listTemplate(_:didSelect:completionHandler:)`
- `CPSearchTemplate` **[NEW]**
- `CPAlertTemplate` **[NEW]** — `.actionSheet`, `.fullScreen`
- `CPNavigationAlert` **[NEW]**
- `CPTrip` **[NEW]**
- `CPRouteChoice` **[NEW]**
- `CPTravelEstimates` **[NEW]**
- `CPNavigationSession` **[NEW]** — `upcomingManeuvers`, `updateEstimates(_:for:)`, `pauseNavigation(for:)`, `finishNavigation()`
- `CPManeuver` **[NEW]**

**MediaPlayer**
- `MPPlayableContentManager` — `dataSource`, `delegate`, `reloadData()`, `beginUpdates()`, `endUpdates()`
- `MPPlayableContentDataSource` — `numberOfChildItems(at:in:)`, `childItemsDisplayPlaybackProgress(at:in:)`, `contentItem(at:in:completionHandler:)`, `beginLoadingChildItems(at:in:)`
- `MPPlayableContentDelegate` — `playableContentManager(_:initiatePlaybackOfContentItemAt:completionHandler:)`
- `MPNowPlayingInfoCenter` — `nowPlayingInfo` dictionary
- `MPRemoteCommandCenter` — `playCommand`, `pauseCommand`, `nextTrackCommand`, etc.

**AVFoundation**
- `AVAudioSession.Mode.voicePrompt` **[NEW]** — correct mode for navigation prompts in CarPlay
- `AVAudioSession.CategoryOptions.duckOthers`
- `AVAudioSession.CategoryOptions.interruptSpokenAudioAndMixWithOthers`

## Code Highlights

App delegate conformance for CarPlay navigation:
```swift
func application(_ application: UIApplication,
                 didConnectCarInterfaceController interfaceController: CPInterfaceController,
                 to window: CPWindow) {
    self.interfaceController = interfaceController
    self.carWindow = window
    window.rootViewController = MapViewController()
    let mapTemplate = buildRootTemplate()
    interfaceController.setRootTemplate(mapTemplate, animated: false)
}
```

Adding a bar button to the map template:
```swift
let favButton = CPBarButton(type: .image) { [weak self] _ in
    self?.displayFavoriteCategories()
}
favButton.image = UIImage(named: "favorites")
mapTemplate.trailingNavigationBarButtons = [trafficButton, favButton]
```

Starting a navigation session and providing maneuvers:
```swift
func startNavigation(trip: CPTrip, routeChoice: CPRouteChoice) {
    mapTemplate.hideTripPreviews()
    let session = mapTemplate.startNavigationSession(for: trip)
    let maneuver = CPManeuver()
    maneuver.instructionVariants = ["Turn left onto Main St"]
    maneuver.symbolImage = UIImage(named: "turn-left")
    session.upcomingManeuvers = [maneuver]
    let estimates = CPTravelEstimates(distanceRemaining: Measurement(value: 0.5, unit: UnitLength.miles),
                                      timeRemaining: 120)
    session.updateEstimates(estimates, for: maneuver)
}
```

Audio session for navigation prompts:
```swift
try AVAudioSession.sharedInstance().setCategory(.playback,
    mode: .voicePrompt,
    options: [.duckOthers, .interruptSpokenAudioAndMixWithOthers])
```

## Takeaways
- iOS 12 introduces the `CarPlay` framework — navigation apps can now appear in CarPlay with full map rendering and turn-by-turn guidance, using a template system that abstracts input hardware and screen differences.
- Audio apps need only use `MPPlayableContent` + `MPNowPlayingInfoCenter` + `MPRemoteCommandCenter`; iOS 12 delivers significant performance improvements without API changes, but use `beginUpdates/endUpdates` instead of `reloadData`.
- Navigation apps own their map window; the CarPlay framework overlays templates and calls delegates — use `safeAreaInsetsDidChange()` to keep map content visible behind the overlay.
- Use `AVAudioSession.Mode.voicePrompt` with duck/interrupt options so navigation audio prompts correctly handle both iOS and in-car audio sources.

---
_Source: WWDC18 Session 213 page (abstract, full transcript, and resource links)._
