# Safeguard Your Accounts, Promotions, and Content
**WWDC21 · Session 10110** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10110/)

_Platforms:_ iOS 14+, iOS 15, tvOS 15

## Overview
This session provides a practical overview of the trust and safety tools Apple provides to help developers protect their apps, accounts, and content from fraud and abuse. It identifies three key threat categories — account takeover, promotion abuse, and content theft/API abuse — and maps each to concrete Apple frameworks and APIs that address them.

The session is aimed at app developers who offer account creation, user-generated content, promotional pricing, or premium content, all of which can attract fraud. The key Apple tools discussed are Sign in with Apple, DeviceCheck, and App Attest — each operating at a different layer of the trust stack.

## Key Topics

### Account Risks and Sign in with Apple
- Account takeover vectors: phishing, credential stuffing from third-party leaks, misplaced trust. Phishing attacks doubled in 2020 per APWG data.
- Fake account creation for rating manipulation, spam, and infrastructure overload.
- **Sign in with Apple**: backed by Apple's two-factor authentication; users authenticate with Face ID or Touch ID.
- Returns a **real user indicator** (`ASUserDetectionStatus`) on first authorization: `.likelyReal`, `.unsupported`, `.unknown`. Use this to skip friction for real users and apply challenges for uncertain ones.
- The real user indicator uses a private on-device ML model combined with server-side inferences.

### Promotion Abuse and DeviceCheck
- Promotions (free items, discounted trials) can be abused by re-downloading apps to claim multiple redemptions.
- **DeviceCheck**: allows setting and reading two persistent bits per device, tied to device authenticity and surviving full device resets.
- Use case: check the bits at promotion redemption time; set a bit to mark the device as having redeemed; optionally reset after a developer-controlled time window.
- Alternative use: rate-limit account creation per device by setting bits at sign-up.
- Privacy-preserving: bits are associated with the device, not a user identity.

### Content Protection and App Attest
- Unauthorized modified apps can bypass in-app purchase checks to access premium content illegitimately.
- **App Attest** (iOS 14+, tvOS 15): uses the Secure Enclave to produce a cryptographic attestation of the app's identity.
- Server verifies the attestation to confirm: (1) the app is unmodified, (2) it is the version published on the App Store, (3) it is running on a genuine Apple device.
- Only a few server round-trips required to establish app identity before serving sensitive content.

## APIs & Frameworks

- `AuthenticationServices` framework — Sign in with Apple
- `ASAuthorizationAppleIDCredential` — credential returned after Sign in with Apple authorization
- `ASAuthorizationAppleIDCredential.realUserStatus` — `ASUserDetectionStatus` enum: `.likelyReal`, `.unsupported`, `.unknown` **[key fraud signal]**
- `ASUserDetectionStatus` enum
- `DeviceCheck` framework
- `DCDevice` — DeviceCheck device token generation
- `DCDevice.current.generateToken(completionHandler:)` — generates a token to send to Apple's servers
- DeviceCheck server API — set/query two per-device bits (`bit0`, `bit1`, `lastUpdateTime`)
- `DCAppAttestService` — App Attest service **[introduced iOS 14]**
- `DCAppAttestService.shared.isSupported` — checks device support
- `DCAppAttestService.shared.generateKey(completionHandler:)` — generates attestation key
- `DCAppAttestService.shared.attestKey(_:clientDataHash:completionHandler:)` — produces attestation object
- `DCAppAttestService.shared.generateAssertion(_:clientDataHash:completionHandler:)` — generates per-request assertion after initial attestation

## Code Highlights

Checking the real user indicator from Sign in with Apple:
```swift
// In ASAuthorizationControllerDelegate
let credential = authorization.credential as? ASAuthorizationAppleIDCredential
switch credential?.realUserStatus {
case .likelyReal:
    // No additional challenge needed
case .unsupported, .unknown, .none:
    // Show CAPTCHA or other challenge
}
```

No code samples for DeviceCheck or App Attest were included in the session (see companion session "Mitigate fraud with App Attest and DeviceCheck" for implementation detail).

## Takeaways
- Sign in with Apple's real user indicator provides a privacy-preserving fraud signal at account creation with no user friction.
- DeviceCheck is the right tool for per-device rate limiting — it survives device resets and is privacy-preserving, making it ideal for promotion abuse prevention.
- App Attest closes the "modified app" attack surface by verifying the app's cryptographic identity on your server before serving premium content.
- Using all three tools in concert covers the major fraud vectors: account abuse, promotion abuse, and content theft.

---
_Source: WWDC21 Session 10110 page (abstract, chapter summaries, and resource links)._
