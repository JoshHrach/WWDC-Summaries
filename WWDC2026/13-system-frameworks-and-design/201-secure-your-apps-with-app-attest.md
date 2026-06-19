# Secure your apps with App Attest
**WWDC26 · Session 201** · [Watch](https://developer.apple.com/videos/play/wwdc2026/201/)

_Platforms:_ iOS, macOS, watchOS, tvOS, visionOS

## Overview
App Attest lets your server verify that requests originate from a genuine, unmodified copy of your app running on real Apple hardware. This session explains how attackers exploit modified apps — injecting false quiz submissions, game cheats, or spoofed data — and how the App Attest flow closes those gaps.

The core mechanism involves generating a Secure Enclave–bound key, obtaining a one-time hardware-backed attestation certificate, and then signing each sensitive request payload as an assertion. Servers validate both the attestation object and per-request assertion signatures, including a monotonic counter that detects replay attacks.

iOS/watchOS/tvOS/visionOS have supported App Attest for some time; macOS 27 adds support, expanding the platform coverage. New in iOS 27, additional authenticator-data extensions are available to strengthen server-side validation. The session also covers the receipt-based fraud metric, which gives a 30-day approximate count of unique attested keys seen on a device — a strong signal when a device acts as a request broker for many modified clients.

## Key Topics

### Protections
App Attest provides three layers of defense: verifying the app runs on genuine Apple hardware, detecting app modifications via the app-identity hash in the attestation certificate, and binding every sensitive payload to the attested key via assertions.

### Availability
`DCAppAttestService.isSupported` gates usage. Unexpected `false` results — e.g., from extension contexts that do not support App Attest — should themselves be treated as a risk signal on the server. macOS 27 now supports App Attest with an additional key access control property in the attestation.

### Key Generation
A Secure Enclave–bound key ID is created with a single async call and should be stored in the Keychain for later reuse. The key is hardware-bound and non-exportable.

### Attestation
`attestKey` submits the key to Apple's servers and returns an attestation object. The session walks through validating the CBOR-encoded attestation: checking the certificate chain, verifying the app-identity hash, and — new in iOS 27 — reading new extensions inside the W3C Authenticator Data structure. Attestations are one-time; once validated, store the public key server-side.

### Assertion
For every sensitive request, `generateAssertion` signs a hash of the request payload with the attested key. Servers verify the signature and check that the counter has strictly increased since the last valid assertion.

### Common Pitfalls
Handle key rotation gracefully (existing users may not have an attested key). Degrade gracefully on rejection rather than hard-blocking immediately — assess risk first. Do not re-attest on every request; attest once and assert per request.

### Fraud Metric
A per-device receipt encodes an approximate 30-day count of unique attested keys (`fraudMetric`). A very high count indicates the device has provisioned keys for many app instances — a strong signal of a brokering attack.

## APIs & Frameworks

### DeviceCheck
- `DCAppAttestService` — shared singleton for all App Attest operations
- `DCAppAttestService.shared.generateKey()` **[NEW on macOS 27]** — async, returns `String` key ID bound to Secure Enclave
- `DCAppAttestService.shared.attestKey(keyId:clientDataHash:)` — returns attestation `Data` to forward to your server
- `DCAppAttestService.shared.generateAssertion(keyId:clientDataHash:)` — returns assertion `Data` to include with each request
- `DCAppAttestService.isSupported` — Bool property to gate usage; treat unexpected `false` as fraud signal
- Fraud metric / receipt API — receipt-encoded `fraudMetric` field (approximate 30-day unique key count)
- Authenticator Data extensions (W3C format) **[NEW in iOS 27]** — additional fields in attestation for stronger server validation
- macOS key access control property **[NEW in macOS 27]** — present in attestation for Mac-specific validation

### External References
- W3C Authenticator Data spec (CBOR structure used in attestation)
- System Integrity Protection (relevant to macOS attestation validity)

## Code Highlights

Generate a Secure Enclave–bound key:
```swift
import DeviceCheck
let keyID = try await DCAppAttestService.shared.generateKey()
```

Attest the key (call once per key, pass `clientDataHash` = SHA-256 of a server-provided challenge):
```swift
let attestation = try await DCAppAttestService.shared.attestKey(
    keyId: keyId, clientDataHash: clientDataHash)
```

Sign a request payload as an assertion:
```swift
let assertion = try await DCAppAttestService.shared.generateAssertion(
    keyId: keyId, clientDataHash: clientDataHash)
```

## Takeaways
- Attest once per key; use assertions for every high-value request — keep the counter rising to block replays.
- macOS 27 and iOS 27 expand App Attest coverage with new platform support and new authenticator-data extensions respectively.
- Treat `isSupported == false` from extension contexts and unexpectedly high fraud metric values as risk signals in your server pipeline.
- Degrade gracefully on rejection; use risk assessment before hard-blocking legitimate users with new or rotated keys.

---
_Source: WWDC26 Session 201 page (abstract, chapter summaries, code samples, and resource links)._
