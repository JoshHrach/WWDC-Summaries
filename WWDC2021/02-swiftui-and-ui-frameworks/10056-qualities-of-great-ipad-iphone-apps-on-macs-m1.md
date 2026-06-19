# Qualities of Great iPad and iPhone Apps on Macs with M1
**WWDC21 · Session 10056** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10056/)

_Platforms:_ macOS Monterey 12, macOS Big Sur 11, iOS 15, iPadOS 15

## Overview
Over one million iPad and iPhone apps are already available on the Mac App Store for M1 Macs, running unmodified. This session explains the system-level bridging that makes this possible — keyboard input, menus, drag and drop, printing, settings bundles, scenes, and window management — and highlights best practices that simultaneously improve the iPad app and its Mac experience.

The session covers improvements shipped in macOS Big Sur 11.3 (window sizing, game controllers, Touch Alternatives, Window Zoom) and macOS Monterey additions including Apple Pay support, enhanced full-screen AV playback, SiriKit Shortcuts availability, and TestFlight for M1 Mac apps. It closes with App Store Connect guidance for verifying compatibility, adjusting macOS availability, and increasing Mac App Store discoverability.

## Key Topics

### Automatic API Bridging
- `UIKeyCommand` keyboard shortcuts and `UIPress` API on `UIResponder` work automatically via the physical Mac keyboard.
- `UIMenuBuilder` customizations (introduced iOS 13) are reflected in the Mac main menu bar.
- Drag and drop via `UIDragInteraction` / `UIDropInteraction` bridges to Mac desktop drag-and-drop.
- `UIPrintInteractionController` bridges to the Mac print dialog; the new `UIApplicationSupportsPrintCommand` Info.plist key adds Print/Export as PDF menu items automatically.
- Settings bundles become Mac-style preference panels; credits entries move to the About box.

### Window and Scene Behavior
- `UIApplicationSupportsMultipleScenes` in Info.plist enables multiple desktop windows; add a New Window menu item.
- iPad multitasking support automatically yields live-resizable windows on Mac.
- Restrict window sizes with `UIWindowScene.sizeRestrictions.minimumSize` / `maximumSize`.
- Use `UIScreen.bounds` not `UIScreen.main.bounds` for layout; the screen size does not change when the window is resized.
- macOS Big Sur 11.3: largest supported device size is chosen at launch; Window Zoom toggles between natural size and pixel-perfect scale.

### Touch Alternatives (macOS Big Sur 11.3+)
- Maps keyboard/trackpad to Multi-Touch, drag, tap, swipe, tilt, and trackpad capture interactions.
- Apps can auto-enable Touch Alternatives by bundling `com.apple.uikit.inputalternatives.plist` with `defaultEnablement = true` and a `requiredOnboarding` array listing only applicable interaction styles.
- macOS Monterey adds sensitivity slider and pointer hiding to the Touch Alternatives preference panel.

### macOS Monterey Improvements
- **Apple Pay**: now available via `PKPaymentAuthorizationControllerDelegate`; implement `paymentAuthorizationController(_:didRequestMerchantSessionUpdate:)` for the cross-platform entitlement.
- **AVKit full screen**: `AVPlayerViewController` and `AVPlayerView` now support separate full-screen windows automatically; new delegate callbacks in `AVPlayerViewDelegate` / `AVPlayerViewControllerDelegate`.
- **AVFoundation**: HDR playback and streaming on M1 Macs — no code changes needed.
- **SiriKit Shortcuts**: custom `INIntent`-based shortcuts now work in iPad/iPhone apps on M1 Macs.
- **TestFlight**: beta testing now available for iPhone/iPad apps running on M1 Macs.

### App Store Deployment
- Verify compatibility in App Store Connect to replace "Not verified for macOS" with "Designed for iPad".
- Apps previously opted out should reconsider given the improvements in Monterey.
- Set a custom minimum macOS version via App Store Connect (no resubmit needed) or via `LSMinimumSystemVersion` in Info.plist.
- iPad/iPhone apps are now searchable by name on the Mac App Store without switching tabs.

## APIs & Frameworks

- `UIKeyCommand` — keyboard shortcut support (automatic)
- `UIPress` / `UIResponder` — low-level keypress handling
- `UIMenuBuilder` — semantic menu structure (bridges to Mac menu bar)
- `UIDragInteraction` / `UIDropInteraction` — drag-and-drop (automatic on Mac)
- `UIPrintInteractionController` — print (bridges to Mac print dialog)
- `UIApplicationSupportsPrintCommand` Info.plist key **[NEW]** — adds Print/Export as PDF to menu bar
- `UIWindowScene.sizeRestrictions` — `minimumSize`, `maximumSize` properties
- `UIWindowScene.sizeRestrictions.minimumSize` / `.maximumSize` (`CGSize`)
- `UIApplicationSupportsMultipleScenes` Info.plist key
- `com.apple.uikit.inputalternatives.plist` **[NEW]** — bundle file to auto-enable Touch Alternatives
- `GameController` framework — controller support; keyboard/trackpad as virtual controller
- `PKPaymentAuthorizationController` — Apple Pay (now supported on M1 Macs)
- `PKPaymentAuthorizationControllerDelegate.paymentAuthorizationController(_:didRequestMerchantSessionUpdate:)` **[NEW requirement]**
- `PKPaymentRequestMerchantSessionUpdate`
- `AVPlayerViewController` / `AVPlayerView` — full-screen video in separate window (macOS Monterey)
- `AVPlayerViewDelegate` **[NEW callbacks]** — full-screen event hooks
- `AVPlayerViewControllerDelegate` **[NEW callbacks]** — full-screen event hooks
- `AVFoundation` — HDR playback/streaming on M1 (automatic)
- `AVCaptureDeviceDiscoverySession` — enumerate cameras by capability rather than front/rear assumption
- `ARConfiguration.isSupported` — gate ARKit features to supported devices
- `LSMinimumSystemVersion` Info.plist key — custom minimum macOS version

## Code Highlights

Constraining window resize range:
```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
           options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = scene as? UIWindowScene,
          let sizeRestrictions = windowScene.sizeRestrictions else { return }
    sizeRestrictions.minimumSize = CGSize(width: 640, height: 480)
    sizeRestrictions.maximumSize = CGSize(width: 1920, height: 1080)
}
```

Auto-enabling Touch Alternatives (`com.apple.uikit.inputalternatives.plist`):
```xml
<key>defaultEnablement</key><true/>
<key>version</key><real>1</real>
<key>requiredOnboarding</key>
<array>
    <string>Tilt</string>
    <string>Multi-Touch</string>
</array>
```

## Takeaways
- iPad/iPhone apps run on M1 Macs unmodified; improvements to the iPad experience automatically translate to Mac.
- Adopt `UIMenuBuilder`, keyboard shortcuts, drag-and-drop, and multitasking to get the best out-of-box Mac experience.
- Bundle `com.apple.uikit.inputalternatives.plist` if your app uses CoreMotion or Multi-Touch to unlock Touch Alternatives and reach a wider Mac audience.
- Verify compatibility in App Store Connect and reconsider any previous opt-outs given macOS Monterey's expanded API bridge.

---
_Source: WWDC21 Session 10056 page (abstract, chapter summaries, code samples, and resource links)._
