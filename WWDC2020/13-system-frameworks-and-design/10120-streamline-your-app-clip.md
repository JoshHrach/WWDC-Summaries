# Streamline your App Clip
**WWDC20 · Session 10120** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10120/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
App Clips are small, focused pieces of an app that are delivered on demand and expire automatically. This session covers best practices for designing and building App Clips that feel fast and purposeful, and demonstrates three key features that streamline the transaction experience: location confirmation (verifying a physical code was scanned at the right location), ephemeral notifications (up to 8 hours without an alert prompt), and secure data migration from App Clip to full app using a shared app group.

The demo uses the Fruta sample app — a smoothie-ordering experience launched by NFC or QR code — to show the full lifecycle: code scan, location confirmation, Apple Pay checkout, ephemeral notification on order completion, Sign in with Apple, `SKOverlay` prompt, and seamless credential migration when the user upgrades to the full app.

## Key Topics

### App Clip Design Best Practices
- Keep the App Clip focused on a single, essential task — defer complex or tangential features to the full app
- Launch into a usable state immediately; do not show splash screens or block on downloads
- Delay account creation prompts until after the user has completed the transaction
- Request permissions (camera, microphone, Bluetooth) only at the moment they are needed
- App Clip and full app must share the same name and icon for a consistent experience
- Maximum App Clip binary size is 10 MB; share assets and code via shared asset catalogs

### Location Confirmation (NEW)
- Allows the App Clip to verify that a physical code (NFC/QR) was scanned at the expected location without requesting full location access
- Enabled by setting `NSAppClipRequestLocationConfirmation = YES` in the `NSAppClipDictionary` of `Info.plist`
- Uses `AppClipActivationPayload.confirmAcquired(in:)` with a `CLCircularRegion` — returns binary yes/no
- Maximum region radius: 500 meters
- User grants permission at the App Clip Card level (system UI) — no `CLLocationManager` prompt shown in-app

### Ephemeral Notifications (NEW)
- App Clips get up to 8 hours of notification permission per physical-code invocation without displaying an authorization alert
- Enabled by setting `NSAppClipRequestEphemeralUserNotification = YES` in `Info.plist`
- Check `UNAuthorizationStatus.ephemeral` (new in iOS 14) before requesting standard authorization to avoid a redundant prompt
- Standard `UNUserNotificationCenter.requestAuthorization` can still be called at any time for long-term permission

### Payments and Apple Pay
- App Clips support all standard iOS payment methods
- Apple Pay (`PKPaymentRequest`) is the recommended approach — no card entry required, keeps the experience fast

### Transitioning Users to the Full App
- `SKOverlay.AppClipConfiguration(position: .bottom)` — present a bottom overlay with App Store link after task completion
- Use `.appStoreOverlay(isPresented:content:)` SwiftUI modifier to show `SKOverlay`
- Secure app group container (`group.*`) transfers from App Clip to full app after installation — App Clip is then deleted
- Save Sign in with Apple `ASAuthorizationAppleIDCredential.user` (user ID) in the shared app group; full app reads it on first launch, verifies with `ASAuthorizationAppleIDProvider.getCredentialState`, and auto-signs the user in

## APIs & Frameworks

**AppClip framework (NEW)**
- `AppClipActivationPayload` **[NEW]** — payload object from `NSUserActivity.appClipActivationPayload`
- `AppClipActivationPayload.confirmAcquired(in: CLCircularRegion, completionHandler:)` **[NEW]** — location confirmation
- `NSAppClipRequestLocationConfirmation` (Info.plist key) **[NEW]**
- `NSAppClipRequestEphemeralUserNotification` (Info.plist key) **[NEW]**

**UserNotifications**
- `UNUserNotificationCenter.getNotificationSettings(completionHandler:)`
- `UNAuthorizationStatus.ephemeral` **[NEW]** — indicates App Clip has 8-hour ephemeral permission
- `UNUserNotificationCenter.requestAuthorization(options:completionHandler:)` — standard full permission request

**StoreKit**
- `SKOverlay` — app install overlay
- `SKOverlay.AppClipConfiguration(position:)` **[NEW]** — configuration for App Clip context
- `.appStoreOverlay(isPresented:content:)` **[NEW SwiftUI modifier]**

**AuthenticationServices**
- `ASAuthorizationAppleIDProvider` — Sign in with Apple
- `ASAuthorizationAppleIDCredential.user` — stable user ID for persistence
- `ASAuthorizationAppleIDProvider.getCredentialState(forUserID:completion:)` — verify saved credential on app launch
- `ASWebAuthenticationSession` — federated third-party login (no app-switch needed)

**CoreLocation**
- `CLCircularRegion(center:radius:identifier:)` — region for location confirmation (max radius 500m)

**Foundation**
- `FileManager.containerURL(forSecurityApplicationGroupIdentifier:)` — shared secure app group container
- `NSUserActivity.appClipActivationPayload` **[NEW]** — access App Clip activation data

## Code Highlights

Location confirmation in response to an App Clip invocation:
```swift
import AppClip
import CoreLocation

guard let payload = userActivity.appClipActivationPayload else { return }

let region = CLCircularRegion(
    center: CLLocationCoordinate2D(latitude: 37.3298193, longitude: -122.0071671),
    radius: 100,
    identifier: "apple_park"
)
payload.confirmAcquired(in: region) { inRegion, error in
    // Only allow payment if inRegion == true
}
```

Check for ephemeral notification status before requesting:
```swift
import UserNotifications

let center = UNUserNotificationCenter.current()
center.getNotificationSettings { settings in
    if settings.authorizationStatus == .ephemeral {
        // Already granted — send notifications without prompting
    }
}
```

SKOverlay after checkout (SwiftUI):
```swift
struct CheckoutView: View {
    @State private var finishedPaymentFlow = false
    var body: some View {
        NavigationView { PaymentView($finishedPaymentFlow) }
            .appStoreOverlay(isPresented: $finishedPaymentFlow) {
                SKOverlay.AppClipConfiguration(position: .bottom)
            }
    }
}
```

Saving and restoring Sign in with Apple credentials across the App Clip / full app boundary:
```swift
// In App Clip — save after successful Sign in with Apple
guard let url = FileManager.default.containerURL(
    forSecurityApplicationGroupIdentifier: "group.com.example.fruta") else { return }
save(userID: credential.user, in: url)

// In full app — verify on first launch
let provider = ASAuthorizationAppleIDProvider()
provider.getCredentialState(forUserID: savedUserID) { state, _ in
    if state == .authorized { loadUserData(userID: savedUserID) }
}
```

## Takeaways

- Use `NSAppClipRequestLocationConfirmation` + `confirmAcquired(in:)` to silently validate physical NFC/QR codes without triggering a Core Location permission prompt — protects against misplaced or spoofed tags.
- Set `NSAppClipRequestEphemeralUserNotification` to get 8 hours of notification access per invocation; check `authorizationStatus == .ephemeral` before calling `requestAuthorization` to avoid a redundant alert.
- Show `SKOverlay.AppClipConfiguration` at task completion (not at launch) so users are offered the full app only after they've seen its value.
- Persist the Sign in with Apple user ID in the shared secure app group so the full app can auto-authenticate returning users the moment they install it.

---
_Source: WWDC20 Session 10120 page (transcript, code samples, and resource links)._
