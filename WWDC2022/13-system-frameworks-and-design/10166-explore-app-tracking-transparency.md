# Explore App Tracking Transparency
**WWDC22 · Session 10166** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10166/)

_Platforms:_ iOS 14+, iPadOS 14+, macOS 11+, tvOS 14+

## Overview
App Store policy requires apps to obtain user permission through the AppTrackingTransparency framework before tracking users across apps and websites owned by other companies. This session defines exactly what constitutes "tracking" under Apple's policy, walks through example scenarios that do and do not require permission, and explains how to implement the framework correctly — including what to declare in the privacy nutrition label and why device fingerprinting is always prohibited regardless of user consent.

The core concept is that linking user or device data from your app with data from another company's app, website, or offline property (for advertising or advertising measurement), or sharing such data with data brokers, constitutes tracking. This applies whether the linkage is done in your code or in a third-party SDK you include.

## Key Topics

**Defining tracking** — Tracking is: (1) linking user/device data from your app with data from other companies' apps, websites, or offline properties for targeted advertising or advertising measurement; or (2) sharing user/device data with data brokers. Hashing an identifier before using it to link data does not exempt the activity from this definition.

**First-party vs. cross-company data use** — Using data collected within your own app for advertising to that same user (first-party), or linking data across two apps owned by the same company, does not constitute tracking. Only cross-company data linkage requires ATT permission.

**Third-party SDK responsibility** — Developers are responsible for the behavior of their entire app, including third-party SDKs and libraries. If an SDK shares user data with an ad network that links it with data from other companies' apps, the app must request tracking permission — regardless of whether the developer uses the SDK for that purpose.

**Data brokers** — Sharing user or device data with any entity that regularly collects and sells or discloses personal information to third parties (a data broker) requires tracking permission, even if the data is not subsequently linked across companies.

**Implementing ATT** — Call `ATTrackingManager.requestTrackingAuthorization(completionHandler:)` to present the system permission alert (one-time per app install). Provide a clear `NSUserTrackingUsageDescription` key in `Info.plist`. Check `ATTrackingManager.trackingAuthorizationStatus` at every app launch. The IDFA returns all zeros if permission is not granted.

**Policy constraints** — Apps must not gate functionality on whether the user grants tracking permission. Fingerprinting — deriving a device or user identifier from device signals (browser configuration, device properties, location, network) — is prohibited under the Apple Developer Program License Agreement regardless of ATT authorization status.

## APIs & Frameworks

### AppTrackingTransparency
- `ATTrackingManager` — entry point for App Tracking Transparency
- `ATTrackingManager.requestTrackingAuthorization(completionHandler:)` — presents the one-time system permission alert
- `ATTrackingManager.trackingAuthorizationStatus` — current permission status: `.authorized`, `.denied`, `.restricted`, `.notDetermined`
- `ATTrackingManager.AuthorizationStatus` — enum of status values

### AdSupport
- `ASIdentifierManager.advertisingIdentifier` (IDFA) — returns all zeros (`00000000-0000-0000-0000-000000000000`) when tracking is not authorized

### Info.plist
- `NSUserTrackingUsageDescription` — required key; string shown in the system permission alert explaining why the app requests tracking

### Privacy Nutrition Label (App Store Connect)
- "Data Used to Track You" section — must be declared separately from ATT authorization; required if app tracks users

### Privacy-preserving ad attribution alternatives
- SKAdNetwork — privacy-preserving ad attribution (no ATT required)
- Private Click Measurement (PCM) — web-to-app and web-to-web attribution without tracking

## Code Highlights

No code samples were included in the session. The key implementation pattern is:

```swift
import AppTrackingTransparency

// Called at an appropriate time after app launch (e.g., after onboarding)
ATTrackingManager.requestTrackingAuthorization { status in
    switch status {
    case .authorized:
        // Proceed with tracking; IDFA is available
    case .denied, .restricted, .notDetermined:
        // Do not track; use contextual or first-party advertising
    @unknown default:
        break
    }
}
```

Check status on every launch:
```swift
let status = ATTrackingManager.trackingAuthorizationStatus
```

## Takeaways
- Tracking is defined by data linkage across company boundaries or data-broker sharing — not by the type of identifier or whether it is hashed.
- Developers are fully responsible for tracking behavior introduced by third-party SDKs; confirm SDK behavior with the SDK vendor before including it.
- Apps must not restrict functionality based on whether the user allows tracking; the ATT prompt is purely for informed consent.
- Fingerprinting is never permitted under the Apple Developer Program License Agreement, regardless of whether the user grants tracking permission.

---
_Source: WWDC22 Session 10166 page (abstract, chapter summaries, and resource links)._
