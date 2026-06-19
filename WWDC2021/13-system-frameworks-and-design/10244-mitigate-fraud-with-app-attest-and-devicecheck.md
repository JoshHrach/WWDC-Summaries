# Mitigate Fraud with App Attest and DeviceCheck
**WWDC21 · Session 10244** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10244/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
App Attest and DeviceCheck are Apple's hardware-backed anti-fraud tools available under the DeviceCheck framework. DeviceCheck provides persistent per-device state (two bits + timestamp) to track promotional offer redemptions across reinstalls and device transfers. App Attest builds on Secure Enclave cryptography to let servers verify that requests originate from a genuine, unmodified version of your app running on real Apple hardware.

App Attest satisfies three core properties: the request came from a genuine Apple device, from your genuine (unmodified) application, and that the payload has not been tampered with in transit. Keys are unique per app installation, never backed up, and never synced, preserving user privacy. Attestations are anonymous with no hardware identifiers.

The typical flow involves generating an App Attest key once, attesting it with Apple's servers once (using a server-issued challenge), and then generating lightweight on-device assertions for each sensitive request thereafter. A Risk Metric Service receipt is returned during attestation and can be periodically redeemed to detect key-farm attacks.

## Key Topics
- **DeviceCheck** — Two persistent bits and a timestamp per device, shared across all apps from a developer; survives reinstall and "Erase all contents."
- **App Attest Key Creation** — `generateKey` creates a Secure Enclave-backed key pair; guard all calls with `isSupported`.
- **Key Attestation** — `attestKey` sends a hardware attestation request to Apple and returns an anonymous attestation object; verify certificate chain, nonce, and app identity hash on your server.
- **Assertion Generation** — `generateAssertion` signs a payload digest on-device without contacting Apple servers; server validates signature, app identity hash, and an ever-increasing counter to prevent replay attacks.
- **Risk Metric Service** — Receipt embedded in attestation can be redeemed to learn how many App Attest keys have been generated on a device for your app (key-farm detection).
- **App Clips Support** — App Clips and their full-app upgrade share the same App Attest identity in iOS 15; clip key invalidation mirrors app uninstallation.
- **Graceful Fallbacks** — `isSupported` can return false for App Extensions and older devices; treat failures as risk signals rather than hard blocks.
- **Gradual Rollout** — `attestKey` generates a network call per install; large install bases must ramp up gradually to avoid rate limiting.

## APIs & Frameworks
- **DeviceCheck** framework
  - `DCAppAttestService` — **[NEW]** shared service for App Attest operations
  - `DCAppAttestService.shared` — Singleton access point
  - `DCAppAttestService.isSupported` — Property; false for App Extensions, devices without Secure Enclave
  - `DCAppAttestService.generateKey(completionHandler:)` — **[NEW]** Creates Secure Enclave key pair; returns `keyId`
  - `DCAppAttestService.attestKey(_:clientDataHash:completionHandler:)` — **[NEW]** Submits attestation to Apple; returns attestation object and risk metric receipt
  - `DCAppAttestService.generateAssertion(_:clientDataHash:completionHandler:)` — **[NEW]** On-device assertion signing; returns assertion object
  - `DCDevice` — Existing DeviceCheck device token API (two bits + timestamp)
  - `DCDevice.current.generateToken(completionHandler:)` — Generates a device token for server-side bit access
- **Secure Enclave** — Hardware backing for App Attest key storage; private key never leaves the enclave
- **Web Authentication (WebAuthn)** standard — Attestation object format
- **PKCS7** — Container format for the risk metric receipt
- **Apple Private PKI** — Source of the App Attest root certificate for certificate chain validation

## Code Highlights
Create an App Attest key, guarded by `isSupported`:

```swift
let appAttestService = DCAppAttestService.shared

if appAttestService.isSupported {
    appAttestService.generateKey { keyId, error in
        guard error == nil else { /* Handle the error. */ }
        // Cache keyId for subsequent operations.
    }
} else {
    // Handle fallback as untrusted device
}
```

Attest the key with a server-issued challenge:

```swift
appAttestService.attestKey(keyId, clientDataHash: clientDataHash) { attestationObject, error in
    guard error == nil else { /* Handle error. */ }
    // Send the attestation object to your server for verification.
}
```

Generate an assertion for each sensitive request:

```swift
appAttestService.generateAssertion(keyId, clientDataHash: clientDataHash) { assertionObject, error in
    guard error == nil else { /* Handle error. */ }
    // Send assertion object with your data to your server for verification
}
```

## Takeaways
- Always validate attestations and assertions on your server, never on-device; modified apps can disable on-device checks.
- Use a one-time server challenge in every attestation and assertion flow to prevent network replay attacks.
- Treat `isSupported == false` and other failures as risk signals feeding into a broader fraud assessment — do not hard-block unconditionally.
- Ramp up `attestKey` usage gradually across your install base to stay within Apple's rate limits; at scale this means days or weeks, not a single release.

---
_Source: WWDC21 Session 10244 page (abstract, chapter summaries, code samples, and resource links)._
