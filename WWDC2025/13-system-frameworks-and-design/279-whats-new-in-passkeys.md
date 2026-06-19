# What's new in passkeys
**WWDC25 · Session 279** · [Watch](https://developer.apple.com/videos/play/wwdc2025/279/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, visionOS 26

## Overview
This session covers five significant passkey enhancements in iOS, iPadOS, macOS, and visionOS 26, all oriented toward accelerating the industry transition away from passwords. The talk frames this as a "passkey journey" with stages: adding passkeys as an option, making them the default, and finally eliminating passwords entirely.

The five updates are: a new account creation API that produces a passkey from the moment of sign-up; Signal APIs that keep credential managers up-to-date when account data changes; automatic passkey upgrades that silently create a passkey after a password sign-in; passkey management endpoints that let credential managers deep-link directly to your enrollment pages; and a new secure import/export mechanism for transferring passkeys between credential manager apps.

## Key Topics

### Account creation API
`ASAuthorizationAccountCreationProvider` presents a pre-filled system sheet requesting name, email/phone, and a passkey — replacing the traditional multi-step sign-up form. The API is available on iOS, iPadOS, macOS, and visionOS, and works with both the built-in Passwords app and third-party credential managers. Three specific errors must be handled: `deviceNotConfiguredForPasskeyCreation` (device has no passcode), `canceled` (user dismissed), and `preferSignInWithApple` (existing Apple account detected — trigger Sign in with Apple instead).

### Keep passkeys up-to-date (Signal APIs)
New `ASCredentialUpdater` methods let apps notify credential managers of account changes:
- `reportPublicKeyCredentialUpdate` — user name / label changed
- `reportAllAcceptedPublicKeyCredentials` — revoke stale credentials by providing the still-valid set
- `reportUnusedPasswordCredential` — signal that a password is no longer needed for an account

Equivalent web APIs exist on `PublicKeyCredential`: `signalCurrentUserDetails`, `signalAllAcceptedCredentials`. Credential manager apps adopt listener APIs on `ASCredentialProviderViewController` to receive these signals.

### Automatic passkey upgrades
After a successful password sign-in, call `createCredentialRegistrationRequest` with `requestStyle: .conditional`. The system checks silently whether conditions are met (credential manager available, password was just used, device configured), then creates a passkey in the background and notifies the user — no UI interruption if conditions aren't met.

### Passkey management endpoints
Serve a JSON file at `/.well-known/passkey-endpoints`. Response must be `200 OK` with `Content-Type: application/json`. Fields: `enroll` (URL to add a passkey) and `manage` (URL for passkey management). Credential managers use this to surface direct upgrade links.

### Import/export
Passkeys can now be transferred between participating credential manager apps via a system-secured, user-initiated flow using Face ID — no plain-text CSV/JSON file is written. The data schema is standardized by the FIDO Alliance. Credential manager developers implement `ASCredentialExportManager` and `ASCredentialImportManager`.

## APIs & Frameworks

### AuthenticationServices
- **`ASAuthorizationAccountCreationProvider`** **[NEW]** — the provider for streamlined account creation
- **`ASAuthorizationAccountCreationProvider.createPlatformPublicKeyCredentialRegistrationRequest(acceptedContactIdentifiers:shouldRequestName:relyingPartyIdentifier:challenge:userID:)`** **[NEW]**
- `ASAuthorizationController.performRequest(_:)` (existing, used with above)
- `ASAuthorizationResult.passkeyAccountCreation` — result case containing account + passkey data
- `ASAuthorizationError.deviceNotConfiguredForPasskeyCreation` **[NEW]**
- `ASAuthorizationError.canceled` (existing)
- `ASAuthorizationError.preferSignInWithApple` **[NEW]**
- **`ASCredentialUpdater`** **[NEW]** — class for signaling credential changes to managers
- `ASCredentialUpdater.reportPublicKeyCredentialUpdate(relyingPartyIdentifier:userHandle:newName:)` **[NEW]**
- `ASCredentialUpdater.reportAllAcceptedPublicKeyCredentials(relyingPartyIdentifier:userHandle:acceptedCredentialIDs:)` **[NEW]**
- `ASCredentialUpdater.reportUnusedPasswordCredential(domain:username:)` **[NEW]**
- `ASAuthorizationPlatformPublicKeyCredentialProvider.createCredentialRegistrationRequest(challenge:name:userID:requestStyle:)` — `.conditional` style **[NEW]**
- **`ASCredentialExportManager`** **[NEW]** — for credential manager apps: export passkeys
- **`ASCredentialImportManager`** **[NEW]** — for credential manager apps: import passkeys
- `ASCredentialProviderViewController` — gains new listener API for update signals **[NEW]**

### Web (WebAuthn / Safari 19)
- `PublicKeyCredential.signalCurrentUserDetails({ rpId, userId, name, displayName })` **[NEW]**
- `PublicKeyCredential.signalAllAcceptedCredentials({ rpId, userId, allAcceptedCredentialIds })` **[NEW]**

### SwiftUI
- `@Environment(\.authorizationController) var authorizationController` (existing)

## Code Highlights

```swift
// Account creation with passkey from day one
let provider = ASAuthorizationAccountCreationProvider()
let request = provider.createPlatformPublicKeyCredentialRegistrationRequest(
    acceptedContactIdentifiers: [.email, .phoneNumber],
    shouldRequestName: true,
    relyingPartyIdentifier: "example.com",
    challenge: try await fetchChallenge(),
    userID: try await fetchUserID()
)
let result = try await authorizationController.performRequest(request)

// Signal a username change to credential managers
try await ASCredentialUpdater()
    .reportPublicKeyCredentialUpdate(
        relyingPartyIdentifier: "example.com",
        userHandle: userHandle,
        newName: "new@example.com"
    )

// Automatic passkey upgrade after password sign-in
let request = provider.createCredentialRegistrationRequest(
    challenge: ..., name: ..., userID: ..., requestStyle: .conditional
)
```

## Takeaways
- Adopt `ASAuthorizationAccountCreationProvider` for new apps to give every new account a passkey from day one — no password ever set.
- Integrate `ASCredentialUpdater` signal APIs whenever the user changes their email, revokes a passkey, or removes their password, so credential managers stay accurate.
- Fire `createCredentialRegistrationRequest(requestStyle: .conditional)` after every password sign-in to silently upgrade accounts — the system handles UI and no error handling is needed on failure.
- Deploy the `/.well-known/passkey-endpoints` JSON on your server — it takes minutes and makes your passkey enrollment page discoverable directly inside credential manager apps.

---
_Source: WWDC25 Session 279 page (abstract, chapter summaries, code samples, and resource links)._
