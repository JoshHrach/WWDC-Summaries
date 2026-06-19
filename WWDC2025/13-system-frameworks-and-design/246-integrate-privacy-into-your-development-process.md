# Integrate privacy into your development process

**Session ID:** 246  
**WWDC Year:** 2025  
**Folder:** `02-privacy-security`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/246/

---

## Overview

This session presents a practical framework for integrating privacy into every phase of the software development lifecycle: planning, design, development, testing, and deployment. Drawing on Apple's four privacy pillars (data minimization, on-device processing, transparency and control, security protections), it walks through concrete tools, APIs, and design patterns for each phase. Rather than treating privacy as a compliance checkbox, the session frames it as an engineering discipline with testable assurances, from defining privacy goals during planning through writing privacy-specific unit and UI tests and configuring nutrition labels before App Store submission.

---

## Key Topics

- Apple's four privacy pillars: data minimization, on-device processing, transparency and control, security protections
- Planning: defining privacy assurances tied to the four pillars
- Design: proactive expectation-setting, clear state change indicators, contextual and meaningful data choices
- Development — UI: `PhotosPicker`, `LocationButton`, out-of-process pickers
- Development — client/server: CloudKit `encryptedValues`, homomorphic encryption / PIR, Privacy Pass, DeviceCheck, `AdAttributionKit`
- Development — local resources: Core ML on-device inference, app group containers, macOS process cleanup
- Testing: unit/integration/UI test pyramid for privacy assurances; App Privacy Report
- Deployment: privacy nutrition labels in App Store Connect, privacy manifests, purpose strings

---

## APIs & Frameworks

- **`PhotosPicker`** (SwiftUI / PhotosUI) – Out-of-process photo picker; app receives only selected photos, no full library access required, no permission prompt shown.
- **`LocationButton`** (CoreLocationUI) – One-tap location sharing; system validates user intent; no `requestWhenInUseAuthorization()` call required.
- **CloudKit `CKRecord.encryptedValues`** – Property for reading/writing encrypted field values; enables Advanced Data Protection (end-to-end encryption) for app data in iCloud when the user enables it. Use encrypted data type variants (`EncryptedString`, `CKAsset`) in your CloudKit schema.
- **Swift Homomorphic Encryption** – Open-source Apple library (`https://github.com/apple/swift-homomorphic-encryption`) for PIR and computation over encrypted data; used to build privacy-preserving lookup services.
- **Private Information Retrieval (PIR)** – Protocol for querying server databases without revealing the query; implemented via Swift Homomorphic Encryption.
- **Privacy Pass** (RFC 9576) – Anonymous device validation tokens; `PrivacyPass` APIs on Apple platforms for server attestation without device identity.
- **`DeviceCheck`** framework – Associates up to 2 bits of per-device state with Apple's servers; enables fraud prevention without persistent device identifiers.
- **`AdAttributionKit`** framework – Privacy-preserving ad attribution; reports installs and re-engagements via signed postbacks without requiring App Tracking Transparency prompts.
- **Core ML** (`CoreML`) – On-device ML model inference and fine-tuning; keeps user data local; supports large language models with optimization/compression.
- **App group containers** – `FileManager` group container directories; protected on macOS from other apps without user permission. Configure via entitlement and Developer Portal app group identifier.
- **Privacy manifests** (`.xcprivacy` files) – Required for apps and third-party SDKs using sensitive APIs; declare data types collected and required reason APIs; surfaced in "Generate Privacy Report" from Xcode archive.
- **App Privacy Report** – Available in iOS Settings (15.2+); shows data access, sensor access, and network activity per app; use during testing to validate data flows.
- **Privacy nutrition labels** – Configured in App Store Connect > App Privacy; declare data types transmitted and their usage; required before App Store submission.
- **`UIPasteControl`** – Out-of-process paste button; user triggers paste explicitly, avoiding background clipboard access.

---

## Code Highlights

Out-of-process Photos picker (SwiftUI):
```swift
PhotosPicker(
    selection: $viewModel.selection,
    matching: .images,
    preferredItemEncoding: .current,
    photoLibrary: .shared()
) {
    Text("Select Photos")
}
.photosPickerStyle(.inline)
.frame(height: 340)
```

Location Button (SwiftUI):
```swift
LocationButton(LocationButton.Title.currentLocation) {
    manager.startUpdatingLocation()
}
.foregroundColor(.white)
.cornerRadius(27)
.frame(width: 210, height: 54)
```

CloudKit encrypted field:
```swift
myRecord.encryptedValues["encryptedStringField"] = "Sensitive value"
let decrypted = myRecord.encryptedValues["encryptedStringField"] as? String
```

---

## Takeaways

- Privacy is most effective when built in from the planning phase; retrofitting privacy controls into a shipped architecture is costly and often incomplete.
- `PhotosPicker` and `LocationButton` reduce permission prompt fatigue by implicitly capturing user intent; adopt them before resorting to broad library/location permission prompts.
- CloudKit `encryptedValues` enables end-to-end encryption for app data in iCloud with zero infrastructure changes; enable it for all sensitive fields.
- PIR (via Swift Homomorphic Encryption) is the right architecture when your app must query a server database without revealing which item is being looked up.
- Privacy testing belongs in the test pyramid: write unit tests for privacy logic, integration tests for data flow boundaries, and UI tests for onboarding and settings flows.
- Privacy manifests are required by App Store for apps using a defined list of sensitive APIs; generate and review the privacy report from Xcode archives before submission.
