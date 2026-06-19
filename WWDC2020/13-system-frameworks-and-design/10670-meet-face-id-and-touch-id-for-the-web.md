# Meet Face ID and Touch ID for the web
**WWDC20 · Session 10670** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10670/)

_Platforms:_ Safari (iOS 14, iPadOS 14, macOS Big Sur 11)

## Overview
Safari in iOS 14 / iPadOS 14 / macOS Big Sur gains support for the W3C Web Authentication (WebAuthn) API, enabling websites to use Face ID and Touch ID as a passwordless or second-factor authentication mechanism. Apple's platform authenticator combines biometric verification (Face ID / Touch ID) with the Secure Enclave — which stores private keys and guarantees they can never leave the device — making every sign-in an implicit multi-factor event (something you have + something you are) in a single tap.

The session covers the full end-to-end flow: feature detection, user onboarding (credential creation), and sign-in (assertion), including the corresponding server-side validation checklist for each step. Apple also announces Apple Anonymous Attestation — a privacy-preserving attestation service that generates a unique certificate per credential so websites cannot track users across the web.

## Key Topics

### Web Authentication Concepts
- **Relying party** — the website requesting authentication
- **Public key credential** — a JavaScript object containing authentication data backed by a public/private key pair
- **Platform authenticator** — an authenticator built into the device (iPhone/Mac), as opposed to a roaming security key
- **Attestation** — an optional cryptographic proof from the device manufacturer confirming the authenticator's properties; Apple Anonymous Attestation (coming soon) generates a unique certificate per credential to prevent cross-site tracking
- Private keys are managed entirely by the Secure Enclave and cannot be exported

### Feature Detection
- Call `PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable()` — returns a `Promise<Boolean>`
- Only proceed with enrollment or sign-in UI if it resolves `true`
- Use feature detection (not user-agent strings)

### Enrollment (Credential Creation)
- Prompt the user after a successful traditional sign-in — e.g., show a banner or full-page notification
- Build an options object with: `rp` (relying party name), `user` (name, id buffer, displayName), `pubKeyCredParams` (algorithm — `-7` for ES256), `challenge` (server-generated random buffer), `authenticatorSelection.authenticatorAttachment: "platform"` (required to target Face ID / Touch ID), and optionally `attestation: "direct"`
- Call `navigator.credentials.create(options)` within a user-activated event; Safari presents the confirmation sheet and biometric prompt
- Server-side: validate `clientDataJSON`, validate `authenticatorData`, optionally validate the attestation statement, then persist the `credentialId` and public key

### Sign-in (Assertion)
- Optionally store a server-side `Secure; HttpOnly` cookie to remember that Face ID / Touch ID is enrolled for a given account on a device, enabling a fast-path sign-in button
- Build options with: `challenge`, `allowCredentials` array containing the stored `credentialId` + `type: "public-key"` + `transports: ["internal"]`
- Call `navigator.credentials.get(options)` within a user-activated event
- Server-side: verify the user ID, confirm the credential ID is associated with that user, validate metadata (`clientDataJSON`, `authenticatorData`), then verify the cryptographic signature

### Best Practices
- Always offer Face ID / Touch ID as an **alternative** to passwords — never as the sole sign-in method (private key is device-bound; losing the device locks the user out)
- Use feature detection, not user-agent sniffing
- Always call `create` and `get` inside user-activated events
- Store enrollment state in a server-side `Secure; HttpOnly` cookie for longest-term persistence (localStorage can be cleared)
- For sites that already support security keys via WebAuthn, consider whether to present both options — UX differs significantly between platform and roaming authenticators

### Availability
- Available in Safari, `SFSafariViewController`, and `ASWebAuthenticationSession` on macOS, iPadOS, and iOS

## APIs & Frameworks

- **Web Authentication API (JavaScript / WebKit)**
  - `PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable()` **[NEW in Safari / iOS 14]** — `Promise<Boolean>`; detects platform authenticator support
  - `navigator.credentials.create(options)` **[NEW for platform authenticator in Safari]** — creates a public key credential (enrollment); returns a `Promise<PublicKeyCredential>`
  - `navigator.credentials.get(options)` **[NEW for platform authenticator in Safari]** — retrieves a credential for assertion (sign-in); returns a `Promise<PublicKeyCredential>`
  - `PublicKeyCredential` — JavaScript object; contains `id` (Base64url credential identifier), `rawId` (ArrayBuffer), and `response`
  - `AuthenticatorAttestationResponse` — response type for `create`; contains `clientDataJSON`, `attestationObject` (public key + attestation statement)
  - `AuthenticatorAssertionResponse` — response type for `get`; contains `clientDataJSON`, `authenticatorData`, `signature`, `userHandle`
  - `authenticatorSelection.authenticatorAttachment: "platform"` — option that selects Face ID / Touch ID over a roaming security key
  - `transports: ["internal"]` — hint to browser to use the platform authenticator during `get`
  - `attestation: "direct"` — optional; requests Apple Anonymous Attestation (coming soon)
- **WebKit / Safari**
  - `SFSafariViewController` — hosts WebAuthn in iOS/iPadOS
  - `ASWebAuthenticationSession` — supports WebAuthn for OAuth-style flows
- **Apple Anonymous Attestation** **[NEW, coming soon]** — generates a unique certificate per credential; prevents cross-site user tracking via attestation certificates

## Code Highlights

Feature detection:
```javascript
const isAvailable = await PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable();
if (isAvailable) {
    // Continue to enrollment or sign in
}
```

Enrollment:
```javascript
const options = {
    publicKey: {
        rp: { name: "example.com" },
        user: {
            name: "john.appleseed@example.com",
            id: userIdBuffer,
            displayName: "John Appleseed"
        },
        pubKeyCredParams: [{ type: "public-key", alg: -7 }],
        challenge: challengeBuffer,
        authenticatorSelection: { authenticatorAttachment: "platform" },
        attestation: "direct"
    }
};
const publicKeyCredential = await navigator.credentials.create(options);
```

Sign-in:
```javascript
const options = {
    publicKey: {
        challenge: challengeBuffer,
        allowCredentials: [{
            type: "public-key",
            id: credentialIdBuffer,
            transports: ["internal"]
        }]
    }
};
const publicKeyCredential = await navigator.credentials.get(options);
```

## Takeaways
- Safari's WebAuthn platform authenticator turns Face ID / Touch ID into a strong multi-factor web sign-in — one tap replaces password + SMS 2FA.
- Three JavaScript calls cover the full lifecycle: `isUserVerifyingPlatformAuthenticatorAvailable()`, `credentials.create()` (enrollment), and `credentials.get()` (sign-in); always call create/get inside user-activated events.
- Always offer this as an alternative, not sole authentication method — the private key is device-bound and irrecoverable if the device is lost.
- Apple Anonymous Attestation (coming soon) allows requesting device attestation without enabling cross-site user tracking.

---
_Source: WWDC20 Session 10670 page (abstract, transcript, and code samples)._
