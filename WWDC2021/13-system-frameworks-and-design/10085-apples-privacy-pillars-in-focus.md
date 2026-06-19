# Apple's Privacy Pillars in Focus
**WWDC21 · Session 10085** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10085/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8

## Overview
This session presents Apple's four privacy pillars — data minimization, on-device processing, transparency and control, and security — and shows how they apply to new platform features in iOS 15 and macOS Monterey. It also walks through iCloud Private Relay's architecture as a detailed case study of all four pillars working together.

The session is organized around developer actions: what new APIs to adopt, what to audit in your app (via Record App Activity), and how to ensure third-party SDKs behave consistently with your Privacy Nutrition Label. The practical goal is ensuring that building great features and respecting privacy aren't a tradeoff.

## Key Topics

### Pillar 1: Data Minimization

**Location Button (`CLLocationButton`) — New in iOS 15**
- Add a Share Current Location button directly in your app UI
- Grants one-time, session-scoped location access when tapped (like "Allow Once" but seamless)
- No permission prompt on subsequent taps during the same session
- Customizable: background color, text color, arrow color, corner radius
- Available on iOS, iPadOS, watchOS, macOS Catalyst
- Replaces "while-in-use" for apps where only specific features need location

**Paste Access Changes**
- Secure Paste confirms that the paste button was tapped directly by the user
- iOS 15: the paste transparency notice (introduced in iOS 14) is no longer shown when paste is explicitly user-triggered via edit menu or keyboard
- Notice only shown for programmatic pastes not triggered by the user
- Use data detectors to check pasteboard content relevance before accessing it

### Pillar 2: On-Device Processing
- iOS 15 moves Siri's automatic speech recognition entirely on-device — audio no longer leaves iPhone/iPad by default
- On-device speech recognition enables offline Siri requests and faster responses
- New in iOS 15: Create ML framework allows training models on-device (iPhone/iPad), using the Apple Neural Engine — sensitive data like photos stays local
- Developers can train personalized models per user without sending training data to servers

### Pillar 3: Transparency and Control

**Hide My Email with iCloud+**
- Extends Sign in with Apple's email hiding to all websites (via Safari) and Mail
- Apps may receive Hide My Email addresses — do not require real email addresses if not essential

**Mail Privacy Protection**
- iOS 15 option: remotely-hosted images in mail loaded through Apple's infrastructure, hiding IP address, read time, and device type from senders
- Remote images may be pre-fetched after delivery — time-of-open metrics will no longer be accurate
- Open rates will increase (images pre-fetched regardless of whether the user read the message)

**Focus Status**
- Apps can request read access to a user's Focus status via `NSFocusStatusUsageDescription` Info.plist key and the User Notifications Communication capability
- Used for communication apps to show availability within the app's experience

**macOS Monterey Microphone Indicator**
- Orange dot in status bar whenever built-in mic, W1, or H1 audio device is active
- Stop audio capture when app is muted

**App Privacy Report and Record App Activity**
- "Record App Activity" developer tool in iOS 15: exports a JSON file listing all sensor/data accesses and all domains contacted by your app and its frameworks
- App Privacy Report: system feature (upcoming software update) that surfaces this data to users in Privacy Settings
- Use Record App Activity in QA to verify: only expected data accessed, only expected domains contacted, SDK behavior aligns with your Privacy Nutrition Label
- Tag connections as website-initiated via `NSURLRequest.attribution = .user` on `NSMutableURLRequest`, `NWConnection`, sockets, and via WKWebView's new `NSURLRequest`-accepting methods

**Privacy Nutrition Labels and App Tracking Transparency**
- Labels must be inclusive of all practices, features, OS versions, and SDK behaviors
- App Tracking Transparency (required since iOS 14.5): must ask permission for any cross-app/website tracking, not just IDFA use
- Fingerprinting disallowed even with user permission
- All app functionality must remain available if tracking is denied

**Privacy-Preserving Ad Attribution**
- New pingback directly to advertisers (no ad network required) for campaign performance measurement without tracking

### Pillar 4: Security

**CloudKit Encrypted Values — New in iOS 15**
- `CKRecord.encryptedValues` **[NEW]** — store/retrieve fields with strong encryption managed by CloudKit
- Supported types: String, Number, Date, CLLocation, Array
- Apple manages encryption and key management; no server-side key access
- Previously only CKAssets were auto-encrypted; now record fields can be too

**iCloud Private Relay (Design Case Study)**
- Two-hop proxy architecture: Ingress Proxy (hides IP from websites) and Egress Proxy (hides destination from Ingress Proxy)
- RSA Blinded Signatures for anonymous network access tokens
- No single party (including Apple) sees both identity and browsing destination
- IP-based approximate location preserved for local content; "Country and Time Zone" option removes even that hint
- End-to-end encrypted via layered encryption on each hop

## APIs & Frameworks

### Core Location
- `CLLocationButton` **[NEW]** — tappable button granting session-scoped location access
  - `backgroundColor`, `tintColor`, `cornerRadius` — customization
  - `label: CLLocationButtonLabel` — text options (e.g., `.currentLocation`, `.shareMyCurrentLocation`)

### CloudKit
- `CKRecord.encryptedValues` **[NEW]** — subscript-based API for encrypted field storage
  - Set: `record.encryptedValues["fieldName"] = value`
  - Get: `let v = record.encryptedValues["fieldName"] as? Type`
  - Used with `CKModifyRecordsOperation` (write) and `CKFetchRecordsOperation` (read)

### Foundation / Network
- `NSMutableURLRequest.attribution` **[NEW]** — `.developer` (default/app traffic) or `.user` (user-initiated web content)
- `NWConnection` — supports `.attribution` for traffic tagging
- `WKWebView` — new methods accepting `NSURLRequest` for proper attribution tagging

### User Notifications
- `NSFocusStatusUsageDescription` Info.plist key — request Focus status read access
- User Notifications Communication capability required for Focus status

### Create ML
- Training models on-device (iPhone/iPad) using Apple Neural Engine **[NEW]**

## Code Highlights

CloudKit encrypted field:
```swift
// Write
myRecord.encryptedValues["encryptedStringField"] = "Sensitive value"
// Read
let decryptedString = myRecord.encryptedValues["encryptedStringField"] as? String
```

## Takeaways
- The four privacy pillars (data minimization, on-device processing, transparency and control, security) are a practical framework for designing any feature — ask the corresponding question at each stage.
- `CLLocationButton` offers a lower-commitment, session-scoped location access path that's better for user trust than upfront "while-in-use" permission requests.
- Record App Activity (iOS 15) is an essential QA tool — run it before shipping to verify your app and its SDKs behave exactly as your Privacy Nutrition Label claims.
- CloudKit's new `encryptedValues` API makes strong field-level encryption trivial — no key management required from the developer.

---
_Source: WWDC21 Session 10085 page (abstract, chapter summaries, code samples, and resource links)._
