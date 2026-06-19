# Explore App Clips
**WWDC20 · Session 10174** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10174/)

_Platforms:_ iOS 14 (App Clips — new feature)

## Overview
App Clips are a new iOS 14 technology that lets users experience a focused slice of an app on demand, without installing it. This introductory session defines the three core concepts — the full app, App Clip Experiences (URLs), and the App Clip binary — explains how to design App Clip flows, walks through creating an App Clip target in Xcode using the Fruta sample app, and covers the key differences between App Clips and full apps including lifecycle, data, privacy restrictions, and migration.

## Key Topics

### The Three Core Concepts
1. **Full app** — a prerequisite; App Clips are additive and cannot exist without a corresponding app.
2. **App Clip Experiences** — URLs registered in App Store Connect (not via Apple-App-Site-Association) that, when triggered on iOS 14, launch an App Clip instead of a web browser. Surfaces include QR codes, NFC tags, Safari/Messages links, Maps business details, and the new App Clip codes (a combined NFC+visual code coming later in 2020).
3. **App Clip binary** — a separate Xcode application target (`.clip` extension) that is built alongside and submitted with the full app. Must be under 10 MB after thinning. When the full app is installed, the app handles the same experiences; the App Clip is not needed.

### Designing App Clip Flows
App Clips should radically simplify an app's navigation hierarchy:
- Remove top-level navigation (tab bars, sidebars) and expose individual experiences via distinct URLs.
- Deep link directly to the task — e.g., a specific store location's ordering flow, not a store chooser.
- Keep interactions short and linear; guide the user to one goal per experience, then optionally offer the full app via `SKOverlay`.
- Optional account features (reward programs, profiles) should appear only after the primary task is complete.

### Creating an App Clip Target in Xcode
1. File → New Target → App Clip; Xcode auto-fills the bundle ID (`<appBundleID>.Clip`) and embeds it in the app.
2. Share code and assets selectively via target membership checkboxes on source files and shared asset catalogs.
3. Use conditional compilation (`#if APPCLIP`) with a custom Swift compiler flag defined in build settings to exclude code paths that reference APIs or data not available in the App Clip.
4. The App Clip receives an `NSUserActivity` on launch (same as Universal Links); use `webpageURL` to identify which experience to handle.

### Key Differences from Full Apps
- **Lifecycle**: iOS deletes an App Clip and its data when unused for a period. If frequently revisited, iOS may retain it. App Clips are not included in iCloud backups.
- **Privacy restrictions**: Access to sensitive data (HealthKit, etc.) is restricted. Apps should check API availability rather than checking for an `isAppClip` property (there is none).
- **Cannot register**: custom URL schemes, document types, Universal Links, or bundled extensions. Use `ASWebAuthenticationSession` for OAuth flows that would normally use custom URL scheme callbacks.
- **Location**: a new location confirmation API allows confirming the user's location without requesting full location access (see companion session "Streamline your App Clip").

### Data Migration to Full App
When a user installs the full app after using an App Clip:
- Camera, microphone, and Bluetooth authorizations are automatically migrated.
- App data is migrated via a new **shared App Group data container** automatically accessible to both the App Clip and the corresponding app. The App Clip writes data to this container; after the full app is installed (and the App Clip is deleted), the container persists until the app copies its contents.
- The App Clip's standard data container is deleted when the App Clip is removed.

### Recommended Technologies
- **Apple Pay** — ideal for in-context purchases without manual card entry.
- **Sign in with Apple / ASAuthorizationController** — for optional account association; enables password-based sign in or new Apple account creation.
- **SKOverlay / `.appStoreOverlay()` (SwiftUI)** — the preferred API to encourage full app installation after a successful App Clip experience.
- **SwiftUI** — its composable, reusable components are well suited to sharing between app and App Clip targets.

## APIs & Frameworks

### App Clips (new in iOS 14)
- App Clip target type in Xcode — creates `.clip` bundle; must include target in app submission
- `NSUserActivity.webpageURL` — the App Clip Experience URL passed at launch
- App Clip Experiences — registered in App Store Connect (not AASA file)
- App Clip shared App Group container — new container type for data migration
- `#if APPCLIP` / Swift custom compiler flag — conditional compilation for App Clip–only code paths
- Size limit: < 10 MB after thinning

### AuthenticationServices
- `ASWebAuthenticationSession` — required for OAuth/federated sign-in in App Clips (no custom URL scheme registration)
- `ASAuthorizationController` — Sign in with Apple and password-based sign-in

### StoreKit
- `SKOverlay` — API to present App Store overlay for full app download
- `.appStoreOverlay(isPresented:configuration:)` — SwiftUI modifier equivalent

### AppClip framework
- Location confirmation API (see "Streamline your App Clip" session for details)

## Takeaways
- App Clips require a full app target as a prerequisite; they are built and submitted together but are mutually exclusive at runtime — once the app is installed, it handles the same experiences.
- Design App Clip experiences around a single linear task with deep-link URLs per experience; omit tab bars and top-level navigation.
- App Clip targets reuse standard Xcode target membership mechanisms for sharing code and assets; use `#if APPCLIP` for conditional compilation of incompatible code paths.
- Use the shared App Group data container for data migration; automatic migration of location/camera/microphone/Bluetooth authorizations happens at app install time.
- App Clips cannot register custom URL schemes — use `ASWebAuthenticationSession` for all federated sign-in OAuth flows.

---
_Source: WWDC20 Session 10174 page (abstract and transcript)._
