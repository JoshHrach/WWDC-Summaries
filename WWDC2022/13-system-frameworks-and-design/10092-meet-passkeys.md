# Meet Passkeys
**WWDC22 · Session 10092** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10092/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13 (apps + Safari), web (WebAuthn/JavaScript)

## Overview
Passkeys are a replacement for passwords built on the WebAuthentication (WebAuthn) standard and public-key cryptography. Introduced in iOS 16 and macOS Ventura after a developer preview in iOS 15, passkeys eliminate entire categories of account security problems: weak/reused credentials, credential server leaks, and phishing are structurally impossible with passkeys because the server never stores or sees the private key.

The user experience is dramatically simpler than passwords: a passkey suggestion appears in the QuickType bar when the username field is focused, one tap triggers Face ID or Touch ID, and sign-in is complete. Passkeys sync across all of a user's devices via iCloud Keychain and can also be shared via AirDrop. Cross-device sign-in (e.g., signing in to a website on a friend's PC using your iPhone's passkey) works via a Bluetooth proximity check and end-to-end encrypted relay — a first-class system feature requiring no app-level code.

## Key Topics
- **WebAuthn foundation** — passkeys are FIDO2/WebAuthn credentials; the server stores a public key, the device holds the private key; sign-in is a challenge-response (ES256) where the private key signs a server-issued challenge and only the signature is sent to the server
- **iCloud Keychain sync** — passkeys automatically sync across all devices running iOS 16 / macOS Ventura under the same Apple ID; AirDrop sharing lets users share passkeys with trusted contacts
- **AutoFill-assisted requests** — the primary integration path; call `controller.performAutoFillAssistedRequests()` early in the view lifecycle; passkeys appear in the QuickType bar when a `textContentType == .username` field is focused; zero additional UI required
- **Modal requests** — use `controller.performRequests()` to show a full-screen sheet listing all available passkeys; use when the user has typed a username manually; combine with an allow list to restrict to matching credentials
- **`preferImmediatelyAvailableCredentials`** — option for `performRequests(options:)` that fails silently (via delegate error callback) if no passkeys are locally available, instead of showing the cross-device QR flow; use to probe for local credentials before showing a traditional sign-in form
- **Combined credential requests** — a single `ASAuthorizationController` can handle passkeys, passwords (`ASAuthorizationPasswordProvider`), and Sign in with Apple (`ASAuthorizationAppleIDProvider`) simultaneously; the delegate callback switches on credential type
- **Cross-device sign-in** — built into the system; works via Bluetooth proximity proof + local key agreement + end-to-end encrypted relay; requires no app code; available via the key icon in the AutoFill QuickType bar or in modal passkey sheets
- **Associated Domains** — required prerequisite; declare `webcredentials` service in the `apple-app-site-association` file linking the app and server domain
- **Web (JavaScript)** — uses standard `navigator.credentials.get()` WebAuthn API; AutoFill-assisted: add `mediation: "conditional"` and annotate the input with `autocomplete="username webauthn"`; modal: standard call without `mediation`; user verification should remain `"preferred"` (default)
- **Security model** — passkeys protect against phishing (device verifies the relying party), credential leaks (server has no private key), and account compromise from reused credentials; no additional MFA factors needed

## APIs & Frameworks
**AuthenticationServices framework**
- `ASAuthorizationPlatformPublicKeyCredentialProvider` **[NEW]** — creates passkey registration and assertion requests for platform authenticators (device + iCloud Keychain)
  - `init(relyingPartyIdentifier: String)` — identifies the server domain
  - `createCredentialAssertionRequest(challenge: Data) -> ASAuthorizationPlatformPublicKeyCredentialAssertionRequest` **[NEW]** — sign-in request
  - `createCredentialRegistrationRequest(challenge:name:userID:) -> ASAuthorizationPlatformPublicKeyCredentialRegistrationRequest` **[NEW]** — registration request
- `ASAuthorizationPlatformPublicKeyCredentialAssertionRequest` **[NEW]**
  - `allowedCredentials: [ASAuthorizationPlatformPublicKeyCredentialDescriptor]` — restrict sheet to specific credential IDs
- `ASAuthorizationPlatformPublicKeyCredentialDescriptor` **[NEW]**
  - `init(credentialID: Data)` — wraps a credential ID for the allow list
- `ASAuthorizationController` — existing class; extended with:
  - `performAutoFillAssistedRequests()` **[NEW]** — starts a passive AutoFill request
  - `performRequests(options: ASAuthorizationController.RequestOptions)` **[NEW]**
  - `ASAuthorizationController.RequestOptions.preferImmediatelyAvailableCredentials` **[NEW]** — silent fallback if no local credentials
- `ASAuthorizationPlatformPublicKeyCredentialAssertion` **[NEW]** — credential type returned on successful passkey sign-in
  - `signature: Data` — ES256 signature over challenge + client data
  - `rawClientDataJSON: Data` — JSON blob to send to server for verification
  - `credentialID: Data`, `userID: Data`, `relyingParty: String`
- `ASAuthorizationError.Code.canceled` — returned in `didCompleteWithError` when no credentials available (with `preferImmediatelyAvailableCredentials`) or user dismissed the sheet

**Associated Domains (existing, prerequisite)**
- `webcredentials` service in `apple-app-site-association`:
  ```json
  { "webcredentials": { "apps": ["A1B2C3D4E5.com.example.Shiny"] } }
  ```

**UIKit**
- `UITextField.textContentType = .username` — signals to the system to offer passkey (and password) AutoFill suggestions for this field

**Web (JavaScript / WebAuthn standard)**
- `PublicKeyCredential.isConditionalMediationAvailable()` — feature detect AutoFill-assisted WebAuthn support
- `navigator.credentials.get({ publicKey: { challenge }, mediation: "conditional" })` — AutoFill-assisted passkey request
- `navigator.credentials.get({ publicKey: { challenge } })` — modal passkey request
- HTML annotation: `autocomplete="username webauthn"` on the username input field

## Code Highlights
AutoFill-assisted passkey sign-in (iOS/macOS native):
```swift
func signIn() {
    let challenge: Data = /* fetch from server */
    let provider = ASAuthorizationPlatformPublicKeyCredentialProvider(
        relyingPartyIdentifier: "example.com")
    let request = provider.createCredentialAssertionRequest(challenge: challenge)

    let controller = ASAuthorizationController(authorizationRequests: [request])
    controller.delegate = self
    controller.presentationContextProvider = self
    controller.performAutoFillAssistedRequests()  // call early in viewDidLoad/viewDidAppear
}
```

Handling the passkey assertion:
```swift
func authorizationController(controller: ASAuthorizationController,
     didCompleteWithAuthorization authorization: ASAuthorization) {
    guard let assertion = authorization.credential
            as? ASAuthorizationPlatformPublicKeyCredentialAssertion else { return }
    // Send assertion.signature + assertion.rawClientDataJSON to server for verification
}
```

Combined passkey + password + Sign in with Apple request:
```swift
let passkeyRequest = passkeyProvider.createCredentialAssertionRequest(challenge: challenge)
let passwordRequest = ASAuthorizationPasswordProvider().createRequest()
let siwaRequest = ASAuthorizationAppleIDProvider().createRequest()
let controller = ASAuthorizationController(authorizationRequests: [
    passkeyRequest, passwordRequest, siwaRequest])
controller.performRequests()
```

AutoFill-assisted WebAuthn (JavaScript):
```javascript
if (PublicKeyCredential.isConditionalMediationAvailable?.()) {
    navigator.credentials.get({
        publicKey: { challenge: /* from server */ },
        mediation: "conditional"
    }).then(assertion => { /* send to server */ });
}
```

## Takeaways
- Passkeys eliminate phishing, credential leaks, and weak/reused passwords simultaneously — no additional MFA needed; adopt `ASAuthorizationPlatformPublicKeyCredentialProvider` with `performAutoFillAssistedRequests()` to add passkeys to an existing sign-in flow with minimal code changes.
- Always start the AutoFill-assisted request early (before the username field is focused) so the passkey suggestion is ready when the keyboard appears.
- Use `performRequests(options: .preferImmediatelyAvailableCredentials)` to silently probe for local passkeys before showing a traditional form; handle `ASAuthorizationError.canceled` to fall through gracefully.
- Cross-device sign-in (QR code + Bluetooth proximity proof) is a built-in system feature — no app code required; ensure it remains reachable via AutoFill or a modal request path.

---
_Source: WWDC22 Session 10092 page (abstract, chapter summaries, code samples, and resource links)._
