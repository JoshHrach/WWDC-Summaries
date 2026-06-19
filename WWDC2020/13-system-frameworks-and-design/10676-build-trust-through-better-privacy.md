# Build Trust Through Better Privacy
**WWDC20 · Session 10676** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10676/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
This session presents Apple's four privacy pillars — on-device processing, data minimization, security protections, and transparency and control — and explains how each pillar manifests in new iOS 14 platform features. The session is primarily a policy and architecture survey rather than a deep API tutorial; it surveys a wide set of new privacy features and directs developers to companion sessions for each.

The overarching theme is that better privacy is also better UX: limiting data collection reduces friction (fewer prompts, less user hesitation), while on-device processing keeps sensitive data off remote servers entirely. The session closes with SKAdNetwork v2, Apple's engineering approach to advertising attribution without cross-app user tracking — demonstrating that privacy engineering and useful business metrics are compatible goals.

## Key Topics

**On-Device Processing**
Machine learning model training via Core ML on device, private federated learning (PFL) for improving system models without sending raw user data to Apple, on-device dictation for many languages (no audio sent to Apple servers), HomeKit face recognition running entirely on the home hub. Developers should prefer on-device ML inference and consider PFL for aggregate model improvement.

**Data Minimization — Three New APIs**

_PHPicker (Photos):_ New photos picker framework replacing `UIImagePickerController`. Runs in a separate process rendered on top of the app — the app cannot screenshot it or access photos the user did not explicitly select. No Photos Library authorization prompt is required. Only selected items are returned to the app. Use for any pick-a-photo flow that does not require broad library access.

_Approximate Location (Core Location):_ Users can share only approximate location (within a few miles radius) rather than precise location. The new `NSLocationDefaultAccuracyReduced` Info.plist key requests approximate location by default. Apps that genuinely need precise location for specific features can request a temporary precision upgrade. All location authorization prompts in iOS 14 include the approximate-vs-precise choice.

_Keyboard Contact Suggestions (Contacts):_ The QuickType keyboard proactively suggests contact details (name, phone, email) in annotated text fields without the app needing Contacts framework access. Use `UITextContentType` annotations on text fields to enable suggestions.

**Security Protections — Encrypted DNS**
iOS 14 natively supports DNS over HTTPS (DoH) and DNS over TLS (DoT). Encrypted DNS prevents ISPs and network operators from reading or modifying DNS queries. Apple servers automatically use Apple's DoH server. Developers hosting web content can configure devices to resolve DNS queries via their own encrypted DNS servers. Encrypted SNI (via IETF TLS ESNI/ECH work in progress) will eventually protect the TLS handshake SNI field from observation.

**Transparency and Control**
- App Store privacy nutrition labels: starting fall 2020, apps must declare what data they collect and how it is used (including third-party SDK data) via a questionnaire; information is shown on the App Store product page.
- Pasteboard access: iOS 14 shows a system banner when an app reads another app's clipboard without the user explicitly pasting; developers should not pre-warm clipboard access.
- Camera and microphone indicator: a hardware-level status bar indicator activates when any app is recording; Control Center shows which app used camera/mic most recently.
- Local network access: apps must declare Bonjour service types in Info.plist and will prompt the user before accessing the local network.
- Private Wi-Fi addresses: iOS 14 rotates MAC addresses per network (new address every 24 hours), preventing cross-network tracking via MAC.

**App Tracking Transparency (ATT) — New Framework**
Moving forward, apps must request permission before tracking users across apps and websites of other companies. The `AppTrackingTransparency` framework presents the system prompt. If denied, the IDFA API returns all zeros. If the app is not built against the iOS 14 SDK, IDFA also returns all zeros.
- Requires `NSUserTrackingUsageDescription` Info.plist key with a clear explanation.
- Call the framework each launch before using IDFA; do not cache the IDFA value.
- Users with Limit Ad Tracking enabled carry that preference forward as "denied."

**SKAdNetwork v2 — Privacy-Preserving Ad Attribution**
An improved framework for advertising conversion measurement that requires no user tracking. Ad networks call StoreKit on-device with a campaign ID; the App Store client records the download. On first launch, the installed app signals a conversion to StoreKit; the App Store client aggregates and verifies the conversion (checking that sufficient users made the same path to prevent individual identification) and sends conversion data to the ad network — with no user-specific identifiers. Cryptographic signatures ensure integrity throughout.

## APIs & Frameworks

### PhotosUI **[NEW]**
- `PHPickerViewController` **[NEW]** — separate-process photo picker; no PHPhotoLibrary authorization required
- `PHPickerConfiguration` **[NEW]** — configures selection limits, filter types
- `PHPickerResult` **[NEW]** — result containing `NSItemProvider` for selected items
- `PHPickerViewControllerDelegate.picker(_:didFinishPicking:)` **[NEW]** — delivers selected items

### Core Location
- `NSLocationDefaultAccuracyReduced` **[NEW]** — Info.plist key; requests approximate location by default
- `CLLocationManager.requestTemporaryFullAccuracyAuthorization(withPurposeKey:)` **[NEW]** — requests temporary precise location upgrade
- `CLAccuracyAuthorization` **[NEW]** — `.fullAccuracy` or `.reducedAccuracy` indicating current precision grant
- `CLLocationManagerDelegate.locationManagerDidChangeAuthorization(_:)` **[NEW]** — replaces `didChangeAuthorization:` for iOS 14

### AppTrackingTransparency **[NEW]**
- `ATTrackingManager` **[NEW]** — requests and reads tracking authorization status
- `ATTrackingManager.requestTrackingAuthorization(completionHandler:)` **[NEW]** — presents the system tracking permission prompt
- `ATTrackingManager.AuthorizationStatus` **[NEW]** — `.authorized`, `.denied`, `.restricted`, `.notDetermined`
- `ATTrackingManager.trackingAuthorizationStatus` **[NEW]** — reads current status without prompting
- Info.plist key: `NSUserTrackingUsageDescription` **[NEW]** — required purpose string for tracking prompt

### AdSupport / SKAdNetwork
- `ASIdentifierManager.advertisingIdentifier` — returns IDFA; all-zeros if tracking not authorized or if app not built against iOS 14 SDK
- `SKAdNetwork` (StoreKit) — privacy-preserving ad conversion attribution; new in v2: 100 campaign IDs, postback conversion values, view-through attribution
- `SKAdNetwork.updateConversionValue(_:)` **[NEW in SKAdNetwork v2]** — updates conversion value from installed app

### NetworkExtension / DNS
- Encrypted DNS profiles (DNS over HTTPS, DNS over TLS) — configurable via MDM or `NEDNSSettingsManager`
- `NSBonjourServices` Info.plist key — declares Bonjour service types used by the app; required for local network access prompt

### NearbyInteraction **[NEW]**
- `NISession` **[NEW]** — session-scoped ultra-wideband ranging between devices (U1 chip); prompts for session-based access without Bluetooth/network permission

### UIKit
- `UITextContentType` — annotate `UITextField` for QuickType contact suggestions (no Contacts framework access needed)
- Pasteboard banner: automatic system UI; triggered by `UIPasteboard.general` access from another app's clipboard

## Code Highlights

Requesting tracking authorization:
```swift
import AppTrackingTransparency
import AdSupport

ATTrackingManager.requestTrackingAuthorization { status in
    switch status {
    case .authorized:
        let idfa = ASIdentifierManager.shared().advertisingIdentifier
        // use idfa
    case .denied, .restricted, .notDetermined:
        // do not use idfa
    }
}
```

Requesting approximate location by default (Info.plist):
```xml
<key>NSLocationDefaultAccuracyReduced</key>
<true/>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Used to show nearby friends.</string>
```

Using PHPicker without photos authorization:
```swift
import PhotosUI

var config = PHPickerConfiguration()
config.selectionLimit = 1
config.filter = .images
let picker = PHPickerViewController(configuration: config)
picker.delegate = self
present(picker, animated: true)

func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
    picker.dismiss(animated: true)
    guard let provider = results.first?.itemProvider, provider.canLoadObject(ofClass: UIImage.self) else { return }
    provider.loadObject(ofClass: UIImage.self) { image, _ in
        // use image
    }
}
```

## Takeaways
- Adopt `PHPickerViewController` for any photo-picking flow that does not require broad library access — it eliminates the Photos authorization prompt entirely and is more private and less friction for users.
- Integrate `AppTrackingTransparency` before using the IDFA; call `requestTrackingAuthorization` on every launch and never cache the IDFA, as user consent can change at any time.
- Set `NSLocationDefaultAccuracyReduced` in Info.plist if your primary location features work with approximate location (e.g., finding nearby content, weather) — users are more likely to grant reduced-accuracy permission than precise.
- Declare `NSBonjourServices` in Info.plist before using Bonjour/mDNS and provide context-appropriate UI before triggering local network access; the system prompt will appear at first access and cannot be retriggered if denied.

---
_Source: WWDC20 Session 10676 page (transcript and resource links)._
