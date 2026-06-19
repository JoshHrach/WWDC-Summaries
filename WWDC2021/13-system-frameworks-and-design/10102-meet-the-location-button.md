# Meet the Location Button
**WWDC21 · Session 10102** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10102/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12 (Catalyst), watchOS 8

## Overview
iOS 15 introduces `CLLocationButton` (UIKit) and `LocationButton` (SwiftUI), a new secure UI element from the `CoreLocationUI` framework. The Location Button grants a one-time ("Allow Once") location authorization each time the user taps it — without presenting the standard system authorization prompt — providing a low-friction way to share location only at the moment it is needed. This is designed for apps where requesting persistent location authorization feels excessive, such as a single "find nearby" action.

The session also notes that Region-based user notifications come to watchOS 8, allowing Apple Watch apps to deliver spatially relevant notifications without requiring "always" authorization.

## Key Topics

**The Problem with Existing Authorization**
Users frequently select "Allow Once" or "Don't Allow" when shown location authorization prompts for apps that only need location for a specific, one-time action. Repeated Allow Once grants (which expire when the app backgrounds) cause multiple prompts. Selecting "Don't Allow" requires redirecting users to Settings. The Location Button sidesteps both issues.

**How Location Button Works**
- Every successful tap grants one-time location authorization without a system prompt.
- The app receives a delivery equivalent to `requestWhenInUseAuthorization` + one location fix, but without the prompt for each tap.
- The first tap ever (when the app is in `notDetermined` state) presents a one-time confirmation prompt explaining the button behavior. Subsequent taps are silent.
- If a user previously selected "Don't Allow", tapping Location Button shows a re-introduction prompt offering them the chance to restore `notDetermined` status and use the button going forward.
- If the app already has `whenInUse` or better authorization, Location Button behaves like a normal button with no authorization side effects.

**Legibility Requirements**
The system enforces strict legibility requirements to prevent the button from being disguised:
- Sufficient color contrast between `tintColor` (foreground) and `backgroundColor`
- `alpha` must be high enough to be clearly visible
- Button must be large enough to display its content legibly (text + icon)
- Failures are logged as console messages; in Xcode/Interface Builder, issues appear in the Issue navigator. The button target-action is called but no authorization is granted when requirements are not met.
- Test with multiple languages (text expands in German and others) and multiple Dynamic Type sizes to ensure sizing remains correct.

**Privacy Design Guidance**
Apps should ask: does my use case require traditional persistent authorization, or is a one-time grant sufficient? Use Location Button when location is only needed for a discrete, user-initiated action (e.g., "find nearby stores", "tag this photo with my current location").

**Region-Based User Notifications on watchOS 8**
watchOS apps can now schedule region-based local notifications (e.g., "you've arrived at the gym") without requiring "always" location authorization. The system delivers these on the app's behalf. Details in the "What's New in watchOS" session.

## APIs & Frameworks

### CoreLocationUI Framework **[NEW]**

**UIKit**
- `CLLocationButton: UIControl` — secure location-granting button **[NEW]**
  - `icon: CLLocationButtonIcon` — `.arrow` or `.arrowFilled` **[NEW]**
  - `label: CLLocationLabel` — `.currentLocation`, `.sendCurrentLocation`, `.sendMyCurrentLocation`, `.shareCurrentLocation`, `.shareMyCurrentLocation` **[NEW]**
  - `cornerRadius: CGFloat` — 0 = square, value = rounded; set to `width/2` for circle **[NEW]**
  - `fontSize: CGFloat` — label text size **[NEW]**
  - `backgroundColor: UIColor` — button fill **[NEW]**
  - `tintColor: UIColor` — icon and label color **[NEW]**
  - Inherits `UIControl.addTarget(_:action:for:)` for `.touchUpInside`

**SwiftUI**
- `LocationButton` — SwiftUI equivalent of `CLLocationButton` **[NEW]**
  - `LocationButton(.currentLocation) { ... }` — action closure runs on tap **[NEW]**
  - `.symbolVariant(_:)` modifier — `.fill` for filled arrow **[NEW]**
  - `.tint(_:)` modifier — sets foreground/background color treatment **[NEW]**
  - `.cornerRadius(_:)` modifier **[NEW]**

### Core Location — Authorization Interaction
- `CLLocationManager.authorizationStatus` — `notDetermined` apps: first tap shows confirmation; `whenInUse`/`always` apps: button behaves normally
- `CLLocationManager.startUpdatingLocation()` — call after receiving button tap; authorization already granted by button
- No change to existing `requestWhenInUseAuthorization()` / `requestAlwaysAuthorization()` APIs

### watchOS 8 — Region-Based User Notifications **[NEW on watchOS]**
- `CLCircularRegion` + `UNLocationNotificationTrigger` — schedule location-triggered local notifications on watchOS without "always" authorization **[NEW on watchOS]**

## Code Highlights

Replace a UIButton with CLLocationButton (UIKit):
```swift
import CoreLocationUI

// Before (UIButton):
// let button = UIButton()
// button.setTitle("Current Location", for: .normal)
// button.addTarget(self, action: #selector(showNearbyParks), for: .touchUpInside)

// After (CLLocationButton):
let locationButton = CLLocationButton()
locationButton.label = .currentLocation
locationButton.icon = .arrowFilled
locationButton.cornerRadius = 8
locationButton.addTarget(self, action: #selector(showNearbyParks), for: .touchUpInside)
view.addSubview(locationButton)

@objc func showNearbyParks() {
    // No need to call requestWhenInUseAuthorization() — button already granted it
    locationManager.startUpdatingLocation()
    // render map...
}
```

Create a circular Location Button:
```swift
let button = CLLocationButton()
button.label = .currentLocation
button.icon = .arrowFilled
button.backgroundColor = .white
button.tintColor = .black
button.cornerRadius = 25   // half of a 50pt width = perfect circle
```

SwiftUI LocationButton:
```swift
import CoreLocationUI
import SwiftUI

LocationButton(.currentLocation) {
    locationManager.startUpdatingLocation()
}
.symbolVariant(.fill)
.tint(.blue)
.cornerRadius(8)
```

## Takeaways
- `CLLocationButton` / `LocationButton` grant one-time location authorization per tap with no system prompt (after a one-time introduction), making them ideal for discrete, user-initiated location actions.
- The button enforces strict legibility rules at runtime; monitor Xcode issue navigator and console logs to catch color contrast, alpha, and sizing failures during development.
- If users have previously selected "Don't Allow", tapping Location Button offers them a path to re-engage without sending them to Settings — a meaningful UX recovery.
- For apps that only ever need location once per user action (find nearby, photo tagging), Location Button may eliminate the need for a traditional "while in use" authorization flow entirely.

---
_Source: WWDC21 Session 10102 page (abstract, chapter summaries, code samples, and resource links)._
