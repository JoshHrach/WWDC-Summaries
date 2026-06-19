# Designing for Privacy
**WWDC19 · Session 708** · [Watch](https://developer.apple.com/videos/play/wwdc2019/708/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, Safari 13

## Overview
This session establishes Apple's privacy philosophy — data minimization, user control, transparency, and security as a foundation — and walks through how that philosophy materialized into concrete iOS 13 and macOS Catalina changes. Three new user-facing experiences (Sign In with Apple, Location permission improvements, Bluetooth consent) are explained in detail alongside API updates for five existing areas (Wi-Fi SSID access, Contacts notes field, custom URL schemes, Game Center player IDs, and NSUserActivity contextual info). The session closes with three case studies showing privacy as a design challenge that leads to innovation: Private Federated Learning for ML, encrypted offline device finding (Find My), and end-to-end encrypted HomeKit Secure Video.

## Key Topics

- **Sign In with Apple** — Zero-credential single sign-on. Only request data you actually need; name/email can be deferred to billing (via Apple Pay). Provides a verified email relay address per developer (unique per developer, user-controlled). `ASAuthorizationAppleIDCredential.realUserStatus` gives a single bit indicating likely-real user for fraud detection. Supported on iOS, macOS, watchOS, tvOS, and JavaScript. **[NEW]**
- **Location: Allow Once** — iOS 13 adds a third choice to location prompts: "Allow Once." App gets the same While-in-Use access until the system considers the app no longer in use, then must prompt again on next launch. **[NEW]**
- **Location: Always deferred** — Apps requesting Always access now receive the same While-in-Use prompt. If the user selects While-in-Use, the delegate receives `authorizedAlways` and the system will prompt to upgrade to Always separately (only from Home Screen). Periodic reminders when Always access is in use. Existing apps with Always access will receive an upgrade-reminder prompt post-iOS 13 install. No breaking API changes.
- **Bluetooth consent** — iOS 13 requires user consent for any use of Core Bluetooth APIs (previously only required for peripheral manager in background). `NSBluetoothAlwaysUsageDescription` Info.plist key is now required; missing key causes crash on apps linked against iOS 13. **[NEW requirement]**
- **macOS Catalina expanded consent** — New user consent required for Desktop/Documents/Downloads, iCloud Drive, third-party cloud storage, Network/Removable volumes, Screen Recording, and keyboard input monitoring. Additions to existing Mojave protections.
- **Wi-Fi SSID (CNCopyCurrentNetworkInfo)** — Access restricted to apps with location authorization, NEHotspotConfiguration, or VPN entitlement. Others receive nil.
- **Contacts notes field** — Removed from CNContact access even with Contacts authorization. Entitlement request required if truly needed.
- **Custom URL Schemes** — `sourceApplication` in `openURL` options now only reveals the launching app if it is from the same developer team.
- **Game Center player IDs** — Existing `playerID` deprecated. New `teamPlayerID` (team-scoped) and `gamePlayerID` (game-scoped) replace it for privacy.
- **NSUserActivity / SiriKit cleanup** — Apps that surface content via NSUserActivity or INInteraction must call corresponding deletion APIs when content is removed (otherwise Siri suggests deleted content).
- **Privacy Preserving Ad Click Attribution** — New Safari web standard: anchor tags with `adDestination` and `adCampaignID` attributes; Safari records click, detects conversion via well-known URL, reports with 24–48 hour random delay in ephemeral no-cookie session. CampaignID and ConversionID each limited to 64 values (4,096 combinations total). On by default in iOS 13/macOS Catalina. Submitted to W3C. **[NEW]**
- **Speech Recognizer offline transcription** — `SFSpeechRecognizer.isAvailableForLocalRecognition` + `SFSpeechRecognitionRequest.requiresOnDeviceRecognition = true` enables on-device transcription without sending audio to Apple. **[NEW]**
- **Private Federated Learning** — Devices receive a global ML model, train locally, send differential-privacy-noised model updates (not user data) to Apple servers; server aggregates into improved model. Used in iOS 13 for QuickType keyboard personalization and Hey Siri improvements. No user identifiers needed; model updates protected by on-device and central Differential Privacy.
- **Find My offline** — Lost device broadcasts rotating ephemeral Bluetooth public key; nearby devices encrypt their GPS location with that key and upload to Apple (Apple cannot decrypt); owner retrieves encrypted blob and decrypts locally. Apple learns neither device nor finder location.
- **HomeKit Secure Video** — On-device computer vision (person/object detection) on HomePod, Apple TV, or iPad; recordings stored end-to-end encrypted. Apple cannot view the video.

## APIs & Frameworks

### Authentication Services **[NEW]**
- `ASAuthorizationAppleIDProvider` — initiates Sign In with Apple request
- `ASAuthorizationAppleIDCredential` — result credential
  - `.user` — stable user identifier
  - `.email` — verified address or relay address
  - `.fullName` — `PersonNameComponents`, only on first sign-in
  - `.realUserStatus` — `.likelyReal`, `.unknown`, `.unsupported` — fraud indicator **[NEW]**
- `ASAuthorizationController` — presents sign-in sheet

### Core Location **[NEW behaviors]**
- `CLLocationManager.requestWhenInUseAuthorization()` — now triggers Allow-Once/While-in-Use sheet (no Always option in-app)
- `CLLocationManager.requestAlwaysAuthorization()` — presents While-in-Use prompt; defers Always upgrade to Home Screen prompt
- `CLAuthorizationStatus.authorizedWhenInUse` — delegate called with this status initially when Always is deferred

### Core Bluetooth **[NEW requirement]**
- `NSBluetoothAlwaysUsageDescription` — required Info.plist key for any CoreBluetooth use in iOS 13
- `CBCentralManager.authorization` / `CBPeripheralManager.authorization` — `CBManagerAuthorization` enum **[NEW]**

### Speech **[NEW]**
- `SFSpeechRecognizer.isAvailableForLocalRecognition` — Bool; indicates on-device recognition available
- `SFSpeechRecognitionRequest.requiresOnDeviceRecognition` — Bool; set `true` to mandate offline transcription

### GameKit **[NEW]**
- `GKPlayer.teamPlayerID` — team-scoped identifier (replaces `playerID`) **[NEW]**
- `GKPlayer.gamePlayerID` — game-scoped identifier **[NEW]**
- `GKPlayer.playerID` — deprecated in iOS 13

### Core ML **[NEW]**
- Updatable Core ML models — on-device training/fine-tuning; `MLUpdateTask` **[NEW]**
- Background Task API — `BGTaskScheduler` for scheduling background ML training

### WebKit / Safari
- `adDestination` / `adCampaignID` HTML attributes on anchor tags — Privacy Preserving Ad Click Attribution **[NEW web standard]**

## Code Highlights

Sign In with Apple request:

```swift
let provider = ASAuthorizationAppleIDProvider()
let request = provider.createRequest()
request.requestedScopes = [.email] // omit .fullName if not needed

let controller = ASAuthorizationController(authorizationRequests: [request])
controller.delegate = self
controller.presentationContextProvider = self
controller.performRequests()

func authorizationController(controller: ASAuthorizationController,
                             didCompleteWithAuthorization authorization: ASAuthorization) {
    if let credential = authorization.credential as? ASAuthorizationAppleIDCredential {
        let realUserStatus = credential.realUserStatus // .likelyReal / .unknown / .unsupported
        // Store credential.user as the stable identifier
    }
}
```

Offline speech recognition:

```swift
let recognizer = SFSpeechRecognizer()
guard recognizer?.isAvailableForLocalRecognition == true else { return }

let request = SFSpeechURLRecognitionRequest(url: audioFileURL)
request.requiresOnDeviceRecognition = true
recognizer?.recognitionTask(with: request) { result, error in
    // result.bestTranscription processed entirely on device
}
```

## Takeaways

- Ask what you _should_ collect, not what you _can_ collect: Sign In with Apple demonstrates that a full login flow can work with zero PII if you design carefully.
- Privacy challenges are design opportunities — the Find My offline-location problem led to a cryptographically novel ephemeral-key scheme that protects both the lost device and the finder.
- Test the new Allow-Once and deferred-Always location behaviors explicitly — your app must handle state transitions gracefully without assuming Always access comes immediately.
- `NSBluetoothAlwaysUsageDescription` is no longer optional in iOS 13; any app instantiating a CBCentralManager or CBPeripheralManager without it will crash.

---
_Source: WWDC19 Session 708 page (abstract, full transcript, and resource links)._
