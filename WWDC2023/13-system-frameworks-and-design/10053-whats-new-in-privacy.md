# What's New in Privacy
**WWDC23 · Session 10053** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10053/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, Safari 17, visionOS 1

## Overview
This session surveys new privacy-enhancing technologies and platform changes across Apple's 2023 platforms, organized around four privacy pillars: data minimization, on-device processing, transparency and control, and security protections. New APIs covered include an embeddable Photos picker with metadata-sharing controls, `SCContentSharingPicker` for permissionless screen capture, an add-only Calendar permission, Oblivious HTTP (OHTTP) for IP-address anonymization, and the new `SensitiveContentAnalysis` framework for on-device nudity detection. Platform changes include macOS app sandbox cross-app data access controls, Advanced Data Protection in CloudKit, Safari 17 fingerprinting/tracking protections in Private Browsing, per-site Safari app extension permissions, and a detailed breakdown of how visionOS's spatial input model was engineered for privacy by design.

## Key Topics

### Photos Picker Enhancements (iOS 17 / macOS Sonoma)
- Fully embeddable into app UI — rendered by the system, never granting the app access to unselected photos **[NEW]**.
- New display modes: no chrome (raw picker), single horizontal scrolling row, or full inline picker.
- New Options menu: users control whether photo metadata (captions, location) is shared alongside selected photos **[NEW]**.
- Redesigned full-library permission dialog: shows count and sample of photos that will be shared; system periodically reminds users if full access was previously granted.
- Preferred approach: use picker for individual photo access rather than requesting full library permission.

### Screen Capture Picker — SCContentSharingPicker (macOS Sonoma)
- `SCContentSharingPicker` **[NEW]** — `ScreenCaptureKit` API that shows a system window picker on behalf of the app; no screen recording permission required from the app.
- macOS presents the picker and shares only user-selected windows/screens for the duration of the capture session.
- New screen sharing menu bar item **[NEW]** — visible while recording; shows a preview of shared content and allows users to add/remove content or stop the session.
- Customization options: preferred selection modes, restricted application lists.

### Calendar Add-Only Permission (iOS 17 / macOS Sonoma)
- `EventKitUI` view controllers now render outside the app's process — no calendar permission required for apps that only create events using system UI **[NEW behavior]**.
- New add-only calendar permission **[NEW]** — allows apps with custom event-creation UI to write events without read access. Apps can later request full access upgrade when user intent is clear.
- Apps previously granted Calendar access default to write-only on upgrade to iOS 17/macOS Sonoma.

### Oblivious HTTP (OHTTP)
- OHTTP **[NEW]** — standardized internet protocol proxying encrypted application-layer messages through a relay, separating client IP identity from request content.
- Architecture: relay knows client IP + destination server name but not content; application server knows content but not client IP. No single party has full visibility.
- Use cases: anonymous analytics, features requiring identity isolation, privacy-preserving DNS (already used by iCloud Private Relay).
- Adopt via the Network relay APIs (see session 10002 "Ready, Set, Relay").
- Complements Private Access Tokens (replacing IP-based reputation) and encrypted DNS.

### Sensitive Content Analysis Framework (NEW)
- `SensitiveContentAnalysis` framework **[NEW]** — on-device ML-based nudity detection using system-provided models; no content is sent to servers.
- `SCSensitivityAnalyzer` **[NEW]** — main class; check `analysisPolicy` to determine whether analysis is needed and what intervention the system expects.
- `SCSensitivityAnalyzer.analyzeImage(at:)` / `analyzeImage(_:)` **[NEW]** — analyze a photo by URL or `CGImage`.
- `SCSensitivityAnalyzer.videoAnalysis(forFileAt:)` **[NEW]** — returns a `SCSensitivityAnalysis` handler for video files; call `hasSensitiveContent()` to get the result.
- `result.isSensitive` — `true` if nudity is likely detected; app should blur/obfuscate and provide a reveal option.
- Extends Communication Safety (originally Messages-only) to AirDrop, FaceTime voicemail, contact posters, Photos picker.
- Available to all users (not just children) via Sensitive Content Warning.

### macOS App Sandbox — Cross-App Data Protection
- macOS Sonoma requires user permission before an app can access files in another app's data container **[NEW behavior]**.
- Permission is valid for the current app session; resets on quit.
- Apps not in App Sandbox: macOS will prompt at access time.
- Apps with App Sandbox: receive the protection automatically for their own containers; use `NSOpenPanel` or `NSDataAccessSecurityPolicy` (Info.plist key **[NEW]**) for cross-app file access.
- `NSDataAccessSecurityPolicy` — replaces the default same-team-ID policy with an explicit AllowList of processes/installers that can access the app's container without prompting.
- Full Disk Access and same-Team-ID apps continue to work without additional prompts.

### Advanced Data Protection + CloudKit
- CloudKit automatically encrypts data with Advanced Data Protection when users enable it — no key management required from developers.
- Required: use encrypted CloudKit data types (`EncryptedString`, `CKAsset`, etc.) in schema; use `encryptedValues` API on `CKRecord` to read/write encrypted fields.
- `CKRecord.encryptedValues` — subscript for accessing encrypted field values **[existing, highlighted]**.

### Safari 17 Private Browsing Protections
- **Known tracker/fingerprinter blocking** **[NEW]** — Safari blocks loading of known tracking and fingerprinting resources in Private Browsing. Logged in Web Inspector as "Blocked connection to known tracker."
- **Tracking parameter removal** **[NEW]** — identifies and strips known tracking query parameters from URLs during navigation and link copying, while preserving non-identifying URL components.
- **Private Click Measurement in Private Browsing** **[NEW]** — privacy-preserving ad attribution (no disk writes, single browsing context, tab-isolated) now available in Private Browsing mode.

### Safari App Extension Permissions (Safari 17)
- Per-site permission model **[NEW]** — Safari app extensions now use the same per-site permission model as web extensions: users grant access per website.
- Private Browsing control **[NEW]** — users can explicitly allow or deny extensions from running in Private Browsing mode.

### visionOS Spatial Input Model — Privacy by Design
- Eye tracking data stays in an isolated system process; app processes never receive raw eye camera data.
- Hover feedback (element highlighting) rendered by the OS rendering engine outside the app's process — apps do not learn what users look at.
- Apps receive only a standard tap event for the highlighted element when a pinch gesture is detected.
- No new permissions required for basic interaction in existing UIKit/SwiftUI apps.
- Hover effect customization (type, shape, element scope) available via UIKit, SwiftUI, and RealityKit APIs while maintaining the same privacy protections.

## APIs & Frameworks

- `PHPickerViewController` — inline/embedded mode **[NEW]** (iOS 17 / macOS Sonoma)
- `PHPickerFilter`, `PHPickerConfiguration` — picker configuration options
- `SCContentSharingPicker` **[NEW]** — system screen content picker (ScreenCaptureKit)
- `SCContentSharingPickerConfiguration` **[NEW]** — picker customization
- `EventKitUI` — `EKEventEditViewController` now renders out-of-process (no permission needed)
- `EKEventStore.requestFullAccessToEvents(completion:)` — full calendar access request (EventKit)
- `EKEventStore.requestWriteOnlyAccessToEvents(completion:)` **[NEW]** — add-only calendar permission
- `SensitiveContentAnalysis` framework **[NEW]**
- `SCSensitivityAnalyzer` **[NEW]**
- `SCSensitivityAnalyzer.analysisPolicy` **[NEW]** — `SCSensitivityAnalysisPolicy` enum
- `SCSensitivityAnalyzer.analyzeImage(at:)` **[NEW]** — async, URL-based
- `SCSensitivityAnalyzer.analyzeImage(_:)` **[NEW]** — async, CGImage-based
- `SCSensitivityAnalyzer.videoAnalysis(forFileAt:)` **[NEW]** — video analysis handler
- `SCSensitivityAnalysis.isSensitive` **[NEW]** — Bool result
- `SCSensitivityAnalysis.hasSensitiveContent()` **[NEW]** — async Bool for video
- `NSDataAccessSecurityPolicy` (Info.plist key) **[NEW]** — macOS cross-app container access policy
- `NSOpenPanel` — file picker granting persistent cross-app file access (existing)
- `CKRecord.encryptedValues` — encrypted CloudKit field access (existing)
- `Network` framework / `NERelayManager` — OHTTP relay configuration (see session 10002)
- `NSPrivacyAccessedAPITypes` (Info.plist key) — privacy manifest for API usage
- `WKWebView` private click measurement — available in Private Browsing (Safari 17)

## Code Highlights

```swift
// Sensitive Content Analysis — photo
let analyzer = SCSensitivityAnalyzer()
let policy = analyzer.analysisPolicy

let result = try await analyzer.analyzeImage(at: photoURL)
// or: let result = try await analyzer.analyzeImage(cgImage)

if result.isSensitive {
    intervene(policy)  // blur image, show reveal option
}

// Sensitive Content Analysis — video
let handler = analyzer.videoAnalysis(forFileAt: videoURL)
let result = try await handler.hasSensitiveContent()
if result.isSensitive {
    intervene(policy)
}
```

```swift
// Add-only Calendar access (no full read required)
let store = EKEventStore()
try await store.requestWriteOnlyAccessToEvents()
// Use EventKitUI to show system event editor with no permission at all:
let editVC = EKEventEditViewController()
editVC.eventStore = store
present(editVC, animated: true)
```

## Takeaways
- `SCSensitivityAnalyzer` brings on-device nudity detection to any app in a few lines of code, using the same ML models as Communication Safety — no server upload, no model maintenance.
- `SCContentSharingPicker` eliminates the need to request screen recording permission for screen-sharing features; the system picker handles consent and scope limiting automatically.
- The add-only Calendar permission removes the all-or-nothing access problem for event-creation apps — users can grant write access without exposing their full calendar history.
- OHTTP (accessible via NERelayManager) provides a standardized, low-overhead way to decouple client identity from request content, suitable for anonymous analytics and privacy-preserving API calls.
- visionOS's spatial input model demonstrates that privacy-by-design (process isolation, on-device processing, permissionless defaults) can deliver both great UX and strong privacy simultaneously.

---
_Source: WWDC23 Session 10053 page (abstract, chapters, transcript, and code samples)._
