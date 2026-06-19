# Accelerate Your App with CarPlay
**WWDC20 · Session 10635** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10635/)

_Platforms:_ iOS 14

## Overview
CarPlay received a significant expansion in iOS 14, opening up the CarPlay framework to entirely new categories of apps. Previously limited to navigation apps (and audio apps via the older Playable Content API), the CarPlay template system now supports audio apps, communication apps, EV charging, parking, and quick food ordering apps. This session walks through the new templates, app lifecycle changes, and design principles required to build great CarPlay experiences.

The CarPlay framework is template-based: apps configure pre-defined template objects rather than drawing custom UI, ensuring the car screen remains safe and glanceable. Apps must adopt UIScene to use the CarPlay framework, and must apply for a specific entitlement matching their app category. The Playable Content API is deprecated in iOS 14 but can coexist with CarPlay templates for backward compatibility with iOS 13 and earlier.

Three new app categories debut in iOS 14: EV charging, parking, and quick-service restaurants — each enabled by a new Point of Interest template and Information template. Communication apps gain a new Message List Item type and a Contact template for displaying address book information directly on the car screen.

## Key Topics
- **New app categories in iOS 14** — EV charging, parking, and quick food ordering apps can now provide native CarPlay experiences for the first time.
- **UIScene adoption required** — CarPlay framework apps must declare a `CPTemplateApplicationSceneSessionRoleApplication` scene configuration in Info.plist; legacy UIWindow/UIApplicationDelegate APIs are not supported.
- **Playable Content deprecation** — The existing Playable Content API is deprecated in iOS 14; audio apps should migrate to CarPlay audio templates. Coexistence is supported for iOS 13 backward compatibility.
- **Tab Bar Template** — A new container template providing tab-bar navigation across multiple sub-templates; supports dynamic tab updates including badges.
- **Now Playing Template** — A shared singleton template with new customizable playback buttons; the system may launch an app solely to present this template when the user taps the Now Playing button on the CarPlay home screen.
- **Communication apps** — Messaging and VoIP apps can now display contact lists and message threads natively; Siri is still invoked for compose/read/reply flows.
- **Point of Interest Template** — Combines an interactive MapKit map with an information panel; supports up to 12 POIs, pan/zoom, delegate callbacks for map region changes, and selection buttons.
- **Information Template** — Full-screen text and button template used for order summaries, confirmations, and station details.
- **Design principles** — CarPlay is for drivers; tasks must be glanceable, purposeful, and completable in a few seconds. App first-run setup must be completed on iPhone before driving.

## APIs & Frameworks

### CarPlay Framework
- **`CPTemplateApplicationScene`** **[NEW for audio/communication/EV/parking/QSR]** — Scene class for CarPlay template apps
- **`CPTemplateApplicationSceneDelegate`** **[NEW]** — Delegate protocol; `templateApplicationScene(_:didConnect:)` and `templateApplicationScene(_:didDisconnect:)` lifecycle methods
- **`CPInterfaceController`** — Root controller for managing the template stack; `setRootTemplate(_:animated:)`, `pushTemplate(_:animated:)`
- **`CPListTemplate`** — List-style template with sections and items; `title`, `sections`
- **`CPListSection`** — Groups list items within a list template
- **`CPListItem`** — Single row item; `text`, `detailText`, `image` (now writable **[NEW]**), `listItemHandler`; supports dynamic property updates **[NEW]**
- **`CPListImageRowItem`** **[NEW]** — List item subtype displaying a horizontal grid of images; `listItemHandler`, `listImageRowHandler`
- **`CPTabBarTemplate`** **[NEW]** — Container template for tab-bar navigation; `updateTemplates(_:)`
- **`CPTemplate`** tab bar properties **[NEW]** — `tabTitle`, `tabSystemItem`, `tabImage`, `showsTabBadge` on all template subclasses
- **`CPNowPlayingTemplate`** **[NEW]** — Shared singleton Now Playing screen; `shared`, `updateNowPlayingButtons(_:)`, optional "Playing Next" and "Album Artist" buttons
- **`CPNowPlayingPlaybackRateButton`** **[NEW]** — Custom Now Playing button for playback rate control
- **`CPMessageListItem`** **[NEW]** — List item subclass for communication apps; leading indicator options (unread, pin, star, icon), trailing indicator options (mute, text, image); invokes Siri automatically on tap
- **`CPContactTemplate`** **[NEW]** — Full-screen contact display template; up to 3 lines of text, up to 4 action buttons, navigation bar buttons
- **`CPPointOfInterestTemplate`** **[NEW]** — Interactive MapKit map + info panel; up to 12 `CPPointOfInterest` items, `selectedIndex`, `setPointsOfInterest(_:selectedIndex:)`
- **`CPPointOfInterestTemplateDelegate`** **[NEW]** — `pointOfInterestTemplate(_:didChangeMapRegion:)` callback
- **`CPPointOfInterest`** **[NEW]** — Model for a map location; `location` (MKMapItem), `title`, `subtitle`, `informativeText`, `image`, `primaryButton`, `secondaryButton`
- **`CPPointOfInterestButton`** **[NEW]** — Tappable button on a POI information panel
- **`CPInformationTemplate`** **[NEW]** — Full-screen text + button template; one or two column label layout, footer buttons
- **`CPGridTemplate`** — Existing grid template (used with tab bar)

### Related Frameworks
- **MapKit** — `MKCoordinateRegion` used in POI template delegate map region callbacks
- **SiriKit** — Still required for voice/messaging flows in communication apps
- **CallKit** — Still required for telephony in communication (VoIP) apps

## Code Highlights

CarPlay scene manifest (Info.plist):
```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <key>UISceneConfigurations</key>
    <dict>
        <key>CPTemplateApplicationSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationScene</string>
                <key>UISceneConfigurationName</key>
                <string>MyApp—Car</string>
                <key>UISceneDelegateClassName</key>
                <string>MyApp.CarPlaySceneDelegate</string>
            </dict>
        </array>
    </dict>
</dict>
```

Scene delegate lifecycle:
```swift
class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {
    var interfaceController: CPInterfaceController?
    func templateApplicationScene(_ scene: CPTemplateApplicationScene,
            didConnect interfaceController: CPInterfaceController) {
        self.interfaceController = interfaceController
        let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles")
        let section = CPListSection(items: [item])
        let listTemplate = CPListTemplate(title: "Albums", sections: [section])
        interfaceController.setRootTemplate(listTemplate, animated: true)
    }
}
```

Dynamic list item update:
```swift
item.image = fetchedArtwork  // writable in iOS 14; CarPlay reloads only affected rows
```

Point of Interest delegate:
```swift
func pointOfInterestTemplate(_ template: CPPointOfInterestTemplate,
                             didChangeMapRegion region: MKCoordinateRegion) {
    self.locationManager.locations(for: region) { locations in
        template.setPointsOfInterest(locations, selectedIndex: 0)
    }
}
```

## Takeaways
- iOS 14 opens CarPlay to three entirely new app categories (EV charging, parking, quick food ordering) and greatly expands the audio and communication app story.
- All CarPlay template apps must adopt UIScene and apply for a category-specific entitlement; apps are single-category.
- The Playable Content API is deprecated; audio apps should migrate to the new template-based API while keeping Playable Content for iOS 13 backward compat.
- The `CPNowPlayingTemplate` is a shared singleton and must be configured immediately on launch, because the system may present it without the app choosing to do so.

---
_Source: WWDC20 Session 10635 page (abstract, chapter summaries, code samples, and resource links)._
