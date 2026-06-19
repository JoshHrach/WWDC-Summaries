# Get more mileage out of your app with CarPlay
**WWDC22 · Session 10016** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10016/)

_Platforms:_ iOS 16, iPadOS 16

## Overview
CarPlay connects iPhone to a vehicle's display and input controls, providing a safe, glanceable interface for drivers. This session covers two new CarPlay app categories introduced in iOS 16 (Fueling and Driving Task), a brand-new desktop testing tool called CarPlay Simulator, and expanded navigation support via an instrument cluster API that mirrors the existing CarPlay Dashboard pattern.

All CarPlay apps are built on a template system: the app provides data and the system renders the UI, ensuring consistent appearance, appropriate font sizes, and compatibility across every vehicle regardless of screen size or input type. An entitlement is required for each app category and must be requested at developer.apple.com/carplay.

## Key Topics

**New app type: Fueling** — Extends the existing EV Charging category to cover all fueling scenarios (gasoline, alternative fuels). Fueling apps should go beyond location finding — for example, enabling a user to activate a gas pump directly from their CarPlay UI.

**New app type: Driving Task** — A lightweight category for apps that help with simple, seconds-long tasks needed while actively driving: controlling car accessories, displaying road status/information, or capturing data at the start/end of a drive. Best-practice design is a single-screen, minimum-functionality UI; complex configuration or non-driving features belong only in the iPhone app.

**CarPlay Simulator** — A standalone Mac application (downloaded via "Additional Tools for Xcode" on the developer website). Connect an iPhone by USB cable and CarPlay starts exactly as it would in a real vehicle. Benefits: run on an actual iPhone without a car; use Xcode debugger and Instruments simultaneously; test audio mixing between app voice guidance and the car's native audio source; simulate different display sizes, light/dark appearance, knob-based input, and disconnect/reconnect scenarios.

**Instrument cluster support** — Navigation apps can now draw a live map or turn card in a vehicle's digital instrument cluster. The implementation mirrors the existing Dashboard API: declare a new `CPTemplateApplicationInstrumentClusterSceneSessionRoleApplication` scene in Info.plist, implement `CPTemplateApplicationInstrumentClusterSceneDelegate` and `CPInstrumentClusterControllerDelegate`, and receive a `UIWindow` to draw into. Safe-area insets communicate which portion of the cluster view is guaranteed visible.

## APIs & Frameworks

### CarPlay (CarPlay framework)

**Templates (all app types)**
- `CPGridTemplate` — grid of buttons; used by Driving Task apps for simple 2–4 button UIs
- `CPListTemplate` — scrollable table list
- `CPInformationTemplate` — static information display with optional action buttons **[used for Driving Task]**
- `CPPointOfInterestTemplate` — map with selectable points of interest **[used for Driving Task road-status apps]**
- `CPTabBarTemplate`, `CPNowPlayingTemplate`, `CPMapTemplate`, etc. — type-specific templates

**Instrument cluster (new in iOS 16)**
- `CPSupportsInstrumentClusterNavigationScene` (Info.plist key) **[NEW]** — declares instrument cluster support
- `CPTemplateApplicationInstrumentClusterSceneSessionRoleApplication` (Info.plist scene role) **[NEW]** — scene role for instrument cluster
- `CPTemplateApplicationInstrumentClusterScene` **[NEW]** — UIScene subclass for the instrument cluster window
- `CPTemplateApplicationInstrumentClusterSceneDelegate` **[NEW]** — delegate notified when cluster connects/disconnects
- `CPInstrumentClusterController` **[NEW]** — controller passed to delegate; provides the UIWindow for map drawing
- `CPInstrumentClusterControllerDelegate` **[NEW]** — callbacks for zoom, compass visibility, speed-limit visibility, and safe-area changes

**Existing (reference)**
- `CPTemplateApplicationScene` — main CarPlay scene
- `CPTemplateApplicationSceneDelegate` — main CarPlay scene delegate
- `CPTemplateApplicationDashboardScene` — Dashboard scene (iOS 13+)
- `CPTemplateApplicationDashboardSceneDelegate` — Dashboard scene delegate
- `CPSupportsDashboardNavigationScene` (Info.plist key) — declares Dashboard support

### UIKit / Foundation (used within CarPlay)
- `UIWindow` — passed by the cluster controller for custom map drawing
- `viewSafeAreaInsetsDidChange()` — override to respond to cluster safe-area changes
- `safeAreaLayoutGuide` — constrain critical UI to the guaranteed-visible region of the cluster view
- `UIApplicationSupportsMultipleScenes` (Info.plist key) — required for multiple CarPlay scenes

### Xcode / Tools
- CarPlay Simulator (Additional Tools for Xcode) **[NEW]** — standalone app for desk-based testing
  - Simulates display sizes, light/dark appearance, knob input, limit-UI mode, disconnect/reconnect
  - Cluster display option adds a second window for instrument cluster content

## Code Highlights

Info.plist scene manifest with instrument cluster support:
```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <key>CPSupportsDashboardNavigationScene</key><true/>
    <key>CPSupportsInstrumentClusterNavigationScene</key><true/>
    <key>UIApplicationSupportsMultipleScenes</key><true/>
    <key>UISceneConfigurations</key>
    <dict>
        <!-- Instrument cluster scene -->
        <key>CPTemplateApplicationInstrumentClusterSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationInstrumentClusterScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay-Instrument-Cluster</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppCarPlayInstrumentClusterSceneDelegate</string>
            </dict>
        </array>
    </dict>
</dict>
```

Implementing the instrument cluster scene delegate:
```swift
extension TemplateApplicationSceneDelegate: CPTemplateApplicationInstrumentClusterSceneDelegate {

    func templateApplicationInstrumentClusterScene(
        _ scene: CPTemplateApplicationInstrumentClusterScene,
        didConnect instrumentClusterController: CPInstrumentClusterController) {
        TemplateManager.shared.clusterController(
            instrumentClusterController,
            didConnectWith: scene.contentStyle)
    }

    func instrumentClusterControllerDidConnect(_ instrumentClusterWindow: UIWindow) {
        self.instrumentClusterWindow = instrumentClusterWindow
    }
}
```

## Takeaways
- Two new CarPlay entitlement categories — Fueling and Driving Task — expand the app types that can appear in CarPlay; apply at developer.apple.com/carplay.
- Driving Task apps should be a single screen with minimal functionality; leave all non-driving tasks in the full iPhone app.
- The new CarPlay Simulator (in Additional Tools for Xcode) enables full desk-based testing on a real iPhone, including Xcode and Instruments integration, without a physical vehicle.
- Instrument cluster support follows the same pattern as Dashboard: add a scene role in Info.plist, implement the delegate, draw into the provided `UIWindow`, and respect `safeAreaLayoutGuide` to avoid cluster UI overlaps.

---
_Source: WWDC22 Session 10016 page (abstract, transcript, code samples, and resource links)._
