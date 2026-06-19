# Move Beyond Passwords
**WWDC21 · Session 10106** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10106/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session introduces Apple's technology preview for passkeys in iCloud Keychain — the first step in an industry-wide transition away from passwords. Passwords are inherently insecure because they rely on a shared secret between user and server, making them vulnerable to phishing, breaches, and reuse. The WebAuthn standard replaces shared secrets with public/private key pairs: the private key never leaves the device, and the server only stores the public key, eliminating entire categories of attacks.

Passkeys in iCloud Keychain store WebAuthn credentials in iCloud Keychain, end-to-end encrypted so even Apple cannot read them. They sync automatically across all of a user's Apple devices, require no additional hardware, and are usable with a single tap using Face ID or Touch ID. They offer stronger protection than most password-plus-second-factor combinations while being faster and simpler to use.

In iOS 15 and macOS Monterey, passkeys are released as a technology preview (off by default via Developer settings) using the `ASAuthorization` API family in AuthenticationServices framework. The feature builds on the Web Authentication (WebAuthn) standard and aligns with a multi-year industry effort to replace passwords.

## Key Topics
- **Password Challenges** — Shared secrets, phishing, reuse, server breach exposure; 80%+ of hacking-related breaches involve stolen/brute-forced credentials.
- **WebAuthn and Public Key Cryptography** — Key-pair creation per account; private key never shared; server stores only public key; challenge-response signing proves identity without revealing the secret.
- **Security Keys** — Hardware fobs using WebAuthn; strong phishing resistance; new in macOS Monterey and iOS 15: native `ASAuthorization` API for security keys in all apps.
- **Passkeys in iCloud Keychain (Technology Preview)** — **[NEW]** Synced WebAuthn credentials backed by iCloud Keychain; end-to-end encrypted; one-tap sign in; works in apps and Safari on all Apple devices.
- **Account Registration** — `ASAuthorizationPlatformPublicKeyCredentialProvider` + `createCredentialRegistrationRequest`; delegate callback returns `ASAuthorizationPlatformPublicKeyCredentialRegistration`.
- **Sign In / Assertion** — `createCredentialAssertionRequest`; delegate callback returns `ASAuthorizationPlatformPublicKeyCredentialAssertion`.
- **Associated Domains** — `webcredentials` entitlement required to enable phishing resistance by binding credentials to a specific domain.
- **Technology Preview Configuration** — iOS: Developer settings toggle; macOS: Safari Develop menu "Syncing Platform Authenticator" option.

## APIs & Frameworks
- **AuthenticationServices** framework
  - `ASAuthorization` — Existing authorization object; extended with passkey credential types
  - `ASAuthorizationController` — Existing controller; accepts array of mixed request types (passwords, Sign in with Apple, passkeys)
  - `ASAuthorizationPlatformPublicKeyCredentialProvider` **[NEW]** — Provider for platform (on-device) public key credential requests
    - `init(relyingPartyIdentifier:)` — Typically your domain name
    - `createCredentialRegistrationRequest(challenge:name:userID:)` **[NEW]** — Creates a passkey registration request
    - `createCredentialAssertionRequest(challenge:)` **[NEW]** — Creates a passkey sign-in assertion request
  - `ASAuthorizationPlatformPublicKeyCredentialRegistration` **[NEW]** — Registration credential returned on successful account creation
    - `rawAttestationObject` — Attestation data for server verification
    - `rawClientDataJSON` — Client data for server verification
  - `ASAuthorizationPlatformPublicKeyCredentialAssertion` **[NEW]** — Assertion credential returned on successful sign-in
    - `signature` — Cryptographic signature to verify on server
    - `rawClientDataJSON` — Client data for server verification
  - Security Keys (physical hardware):
    - `ASAuthorizationSecurityKeyPublicKeyCredentialProvider` **[NEW]** — Provider for security key (hardware) credential requests
- **Associated Domains** (`webcredentials` entitlement) — Required for platform-level phishing resistance
- **iCloud Keychain** — Backend storage for passkeys; end-to-end encrypted sync across Apple devices
- **Web Authentication (WebAuthn) standard** — Industry standard underpinning both passkeys and security keys

## Code Highlights
Register a new passkey account:

```swift
func createAccount(with challenge: Data, name: String, userID: Data) {
    let provider = ASAuthorizationPlatformPublicKeyCredentialProvider(
            relyingPartyIdentifier: "example.com")
    let registrationRequest = provider.createCredentialRegistrationRequest(
            challenge: challenge, name: name, userID: userID)
    let controller = ASAuthorizationController(
            authorizationRequests: [ registrationRequest ])
    controller.delegate = …
    controller.presentationContextProvider = …
    controller.performRequests()
}
```

Sign in with an existing passkey:

```swift
func signIn(with challenge: Data) {
    let provider = ASAuthorizationPlatformPublicKeyCredentialProvider(
            relyingPartyIdentifier: "example.com")
    let assertionRequest = provider.createCredentialAssertionRequest(challenge: challenge)
    let controller = ASAuthorizationController(
            authorizationRequests: [ assertionRequest ])
    controller.delegate = …
    controller.presentationContextProvider = …
    controller.performRequests()
}
```

Handle the returned credential in the delegate:

```swift
func authorizationController(controller: ASAuthorizationController,
     didCompleteWithAuthorization authorization: ASAuthorization) {
    switch authorization.credential {
    case let registration as ASAuthorizationPlatformPublicKeyCredentialRegistration:
        let attestationObject = registration.rawAttestationObject
        let clientDataJSON = registration.rawClientDataJSON
        // Verify on your server and finish creating the account.
    case let assertion as ASAuthorizationPlatformPublicKeyCredentialAssertion:
        let signature = assertion.signature
        let clientDataJSON = assertion.rawClientDataJSON
        // Verify on your server and finish signing in.
    case …: …
    }
}
```

## Takeaways
- Passkeys in iCloud Keychain provide stronger security than password-plus-second-factor while being simpler to use (single tap), all built on the open WebAuthn standard.
- Private keys never leave the device; servers store only the public key, eliminating server-side credential breach risk.
- iOS 15/macOS Monterey ships this as a technology preview only — enable it via Developer settings before testing; not intended for production accounts yet.
- Set up Associated Domains with `webcredentials` to ensure OS-level phishing protection binds credentials to your specific domain.

---
_Source: WWDC21 Session 10106 page (abstract, chapter summaries, code samples, and resource links)._
