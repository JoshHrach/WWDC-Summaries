# Introducing iPad Apps for Mac
**WWDC19 · Session 205** · [Watch](https://developer.apple.com/videos/play/wwdc2019/205/)

_Platforms:_ macOS Catalina 10.15, iOS 13 / iPadOS 13 (Mac Catalyst)

## Overview
iPad Apps for Mac (later branded as Mac Catalyst) lets developers bring existing iPad apps to macOS by rebuilding them natively against a new UIKit-on-Mac layer — with a single shared codebase and a single checkbox in Xcode 11. Rather than using the iOS Simulator's separate runtime stack, the technology integrates UIKit as a native peer to AppKit on macOS, sharing lower-level frameworks (CoreGraphics, Foundation, etc.), databases (Photos, Contacts, Preferences), and system services (copy/paste, file coordination, kernel) with the rest of the Mac system.

The session introduces the architecture, the Xcode workflow, the catalog of things that work automatically (menu bars, window management, dark mode, scrollbars, document pickers, settings pref panes, text scaling), and the API differences to be aware of — especially around touch vs. mouse input, unavailable hardware-tied frameworks, and data protection.

The companion session "Taking iPad Apps for Mac to the Next Level" covers distribution, advanced Mac customization, and lifecycle management.

## Key Topics

**Architecture**
UIKit is not unified with AppKit; instead, the full iOS framework layer runs natively on the Mac alongside the AppKit layer. Lower-level infrastructure (Foundation, CoreGraphics, Darwin, services, databases) is unified into a single copy. The iOS SDK's @available attributes automatically imply availability for iPad Apps for Mac; unavailable APIs are annotated with `@unavailable(UIKitForMac)`.

**Getting Started in Xcode 11**
Check the "Mac" checkbox under Deployment Info (requires iPad support). Xcode automatically:
- Adds a "My Mac" run destination in the scheme selector.
- Prefixes the Mac app's bundle identifier.
- Adds required entitlements (e.g., `com.apple.security.network.client`) based on Info.plist usage description strings.
- Excludes unavailable frameworks and Apple Watch content from the Mac build.

**XCFrameworks**
New in Xcode 11: XCFramework bundles allow library vendors to package a single distributable that targets multiple platforms. Third-party Simulator-targeted frameworks are not compatible; vendors must provide XCFramework versions for Mac builds.

**What You Get For Free**
- Default menu bar with standard Mac menu items.
- Window management: resize, full screen, Split View, window stoplight.
- Dark mode.
- Overlay scrollbars; inactive-window gesture scrolling.
- Settings bundles automatically mapped to Mac preference panes (via an Xcode-generated menu item).
- Basic Touch Bar support; `AVPlayerViewController` and `UITextView` provide rich Touch Bar content automatically.
- `UIDocumentPickerViewController` mapped to `NSOpenPanel`.
- Multiple windows / multitasking APIs from iOS 13 carry over.
- Form sheets, `UISwitch`, and other UIKit controls render at 77% scale (from iOS 17pt to Mac 13pt baseline).

**Mouse and Touch Input Mapping**
- Left mouse button drags synthesize a single touch sequence (recognized by tap, pan, long-press gesture recognizers).
- Pinch and rotate gestures are synthesized from standard Mac trackpad gestures.
- Scroll gesture triggers `UIScrollView` scrolling without synthesizing touches.
- Custom multi-touch behavior and custom gesture recognizers do not map automatically — must provide Mac alternatives.
- New `UIHoverGestureRecognizer` for detecting mouse cursor hover over views.

**API Differences and Unavailability**
Unavailable frameworks (hardware-tied or iOS-specific):
- `ARKit` — no AR hardware on Mac.
- `HealthKit`, `HomeKit` — underlying functionality not present.
- `ClassKit` — Schoolwork app doesn't exist on Mac.
- `CoreMotion` (most sensors) — no accelerometer/gyroscope.
- `CoreLocation` — available but without GPS chip; less precise.
- `MediaPlayer` — Now Playing / Remote Command available; library access and playback not available.
- `UIWebView` — not available; migrate to `WKWebView`.
- Deprecated frameworks — not guaranteed on this new platform.

Media capture: use `UIImagePickerController` to capture from built-in Mac FaceTime camera.

**Data Protection**
iOS data protection writing options (`completeFileProtection`, etc.) compile and run but are non-functional on macOS. Use:
- Keychain for passwords.
- `CryptoKit` `AES.GCM` for encrypting file data before writing.

**Conditional Compilation**
- Swift: `#if targetEnvironment(UIKitForMac)`
- Objective-C: `#if TARGET_OS_UIKITFORMAC`

**Bundle Format**
Mac builds produce a macOS-style deep bundle (not the flat iOS bundle). Use `NSBundle` APIs for resource lookups rather than hard-coded relative paths.

**Supported Extensions**
Share, Action, Document Provider, Photo Editing, and others work on Mac and appear alongside AppKit app extensions. Custom Keyboards, Sticker Packs, iMessage extensions, and some others are unavailable.

## APIs & Frameworks

**Mac Catalyst / UIKit for Mac** **[NEW]**
- Mac checkbox in Xcode 11 project settings **[NEW]**
- `UIHoverGestureRecognizer` — hover events for mouse cursor **[NEW]**
- `@available(UIKitForMac X.X, *)` annotation **[NEW]**
- `@unavailable(UIKitForMac)` annotation **[NEW]**
- `#if targetEnvironment(UIKitForMac)` / `TARGET_OS_UIKITFORMAC` **[NEW]**
- XCFramework bundle format **[NEW in Xcode 11]**
- Bundle ID prefix for Mac builds (automatic) **[NEW]**
- Settings bundle → Mac preference pane (automatic mapping) **[NEW]**
- `UIDocumentPickerViewController` → `NSOpenPanel` (automatic) **[NEW behavior on Mac]**

**CryptoKit** **[NEW in iOS 13 / macOS 10.15]**
- `AES.GCM.seal(_:using:)` — symmetric encryption **[NEW]**
- `AES.GCM.open(_:using:)` — decryption **[NEW]**
- `SymmetricKey` **[NEW]**

**Frameworks with differences on Mac**
- `CoreLocation` — available; reduced precision (no GPS)
- `AVFoundation` — capture via `UIImagePickerController` from Mac camera
- `Metal` — largely identical; new GPU family API for conditional feature use
- `MediaPlayer` — Now Playing Info Center and Remote Command Center only

**Standard iOS frameworks (available and unchanged)**
- `UIKit`, `Foundation`, `CoreGraphics`, `CoreData`, `MapKit` (UIKit variant), `SceneKit` (UIKit variant), `WebKit` (UIKit variant, WKWebView), `UserNotifications`

## Code Highlights

Conditionally compiling out ARKit for Mac:
```swift
#if !targetEnvironment(UIKitForMac)
import ARKit
// AR session setup
#endif
```

Adding a custom Mac menu item via IBAction:
```swift
// In UIViewController
@IBAction func toggleFavorite(_ sender: Any) {
    // Toggle favorite state
}

func validateMenuItem(_ menuItem: NSMenuItem) -> Bool {
    menuItem.title = recipe.isFavorite ? "Remove Favorite" : "Make Favorite"
    return true
}
```

Encrypting file data with CryptoKit (replacement for data protection):
```swift
import CryptoKit

let key = SymmetricKey(size: .bits256)
let sealedBox = try AES.GCM.seal(data, using: key)
// Write sealedBox.combined to disk, store key in Keychain
```

## Takeaways
- Checking the Mac checkbox in Xcode 11 is the literal starting point; most apps build with minimal code changes as long as they target iPad and avoid unavailable frameworks.
- Custom multi-touch gesture recognizers must be replaced or supplemented with Mac-appropriate alternatives (menu items, keyboard shortcuts, `UIHoverGestureRecognizer`).
- iOS data protection APIs are silently non-functional on Mac; use CryptoKit + Keychain instead.
- This technology is best for iPad apps without an existing, well-maintained AppKit counterpart; AppKit remains the better choice for apps that need full Mac API surface coverage.

---
_Source: WWDC19 Session 205 page (abstract, chapter summaries, code samples, and resource links)._
