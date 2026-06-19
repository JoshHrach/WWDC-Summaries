# Get the Most Out of Sign in with Apple
**WWDC20 · Session 10173** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10173/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This session covers best practices for a complete, secure Sign in with Apple integration and introduces three new additions in 2020: server-to-server developer notifications, a SwiftUI `SignInWithAppleButton` view, and the "Upgrade to Sign in with Apple" extension-based API for migrating existing username/password accounts.

The first half reviews secure request construction using `nonce` and `state` properties, proper handling of the identity token JWT on the server (including nonce verification to prevent replay attacks), credential state checking on every launch, and email relay behavior. The second half introduces `ASAccountAuthenticationModificationExtension` — a new app extension that enables seamless, one-tap upgrade of existing weak credentials to Sign in with Apple, triggered from iOS Settings, AutoFill, or in-app.

## Key Topics

**Secure Authorization Requests**
- Always set `nonce` (unique per-request, verified from `identityToken` on server) to prevent replay attacks
- Always set `state` (returned in credential for local request-to-credential matching)
- `requestedScopes` (.fullName, .email) only returned on first authorization — cache them immediately
- `realUserStatus` in identity token: 0=unsupported, 1=unknown, 2=likelyReal

**Identity Token Verification (Server-Side)**
- Decode JWT to get: `sub` (userIdentifier), `nonce`, `email`, `real_user_status`
- Verify nonce matches what was sent; verify token expiry; verify signature with Apple public key
- Exchange `authorizationCode` with Apple ID servers → receive `refresh_token`, `access_token`, new `identityToken`
- Verify refresh token up to once per day to confirm Apple ID is still in good standing

**Email Relay**
- Team-scoped: same relay address for a user across all of your apps
- Routed to user's verified Apple ID email — no need to verify yourself
- Users can reply to emails; service has no downtime

**Credential State**
- Call `getCredentialState(forUserID:)` on every app launch and foreground transition
- `.authorized` → fast-track to app
- `.revoked` → sign out, present login
- `.notFound` → present login
- `.transferred` **[NEW]** — app was transferred between development teams; re-run sign-in flow with stored userIdentifier to get new team-scoped identifier (silent, no user interaction)

**Server-to-Server Notifications (New)**
- Register endpoint on Apple Developer website
- Events sent as Apple-signed JWTs: `email-disabled`, `email-enabled`, `consent-revoked`, `account-delete`
- `consent-revoked`: user disconnected app from Settings — treat as sign-out
- `account-delete`: Apple ID deleted — userIdentifier no longer valid

**SwiftUI SignInWithAppleButton (New)**
- `SignInWithAppleButton(.signIn)` with `onRequest:` and `onCompletion:` closures
- Label options: `.signIn`, `.continue`, `.signUp`
- `.signInWithAppleButtonStyle(.black / .white / .whiteOutline)`
- Online button customization portal: `appleid.apple.com/signinwithapple/button`

**Upgrade to Sign in with Apple (New)**
- Extension-based: `ASAccountAuthenticationModificationExtension` app extension type
- `ASAccountAuthenticationModificationViewController` — subclass as `NSExtensionPrincipalClass`
- Invoked from: iOS Settings security recommendations, AutoFill on weak credential, or in-app via new API
- On success: existing password credential automatically removed from system
- Supports optional step-up UI for 2FA flows before requesting Apple ID credential

## APIs & Frameworks

### AuthenticationServices
- `ASAuthorizationAppleIDProvider` — creates Apple ID authorization provider
- `ASAuthorizationAppleIDProvider().createRequest()` — creates authorization request
- `ASAuthorizationAppleIDRequest.requestedScopes` — `[.fullName, .email]`
- `ASAuthorizationAppleIDRequest.nonce` — unique string per request **[USE ALWAYS]**
- `ASAuthorizationAppleIDRequest.state` — opaque string for local request matching **[USE ALWAYS]**
- `ASAuthorizationAppleIDRequest.user` — set stored userIdentifier for `.transferred` state migration **[NEW]**
- `ASAuthorizationController` — presents authorization UI
- `ASAuthorizationControllerDelegate.authorizationController(controller:didCompleteWithAuthorization:)`
- `ASAuthorizationAppleIDCredential` — result containing:
  - `.user` — userIdentifier (stable, team-scoped)
  - `.fullName`, `.email` — only present on first authorization; cache immediately
  - `.realUserStatus` — `.likelyReal`, `.unknown`, `.unsupported`
  - `.state` — matches request state for local verification
  - `.identityToken` — JWT; verify nonce on server
  - `.authorizationCode` — exchange on server for refresh/access tokens
- `ASAuthorizationAppleIDProvider.getCredentialState(forUserID:completionHandler:)` — check current credential state
- `ASAuthorizationAppleIDProvider.CredentialState` — `.authorized`, `.revoked`, `.notFound`, `.transferred` **[NEW]**

### SwiftUI — Sign in with Apple Button (New)
- `SignInWithAppleButton` **[NEW]** — SwiftUI view wrapping sign-in button
  - `init(_ label:onRequest:onCompletion:)` — label: `.signIn`, `.continue`, `.signUp`
  - `.signInWithAppleButtonStyle(_:)` — `.black`, `.white`, `.whiteOutline`

### AuthenticationServices — Account Modification Extension (New)
- `ASAccountAuthenticationModificationExtension` **[NEW]** — app extension type
- `ASAccountAuthenticationModificationViewController` **[NEW]** — view controller base class
  - `convertAccountToSignInWithAppleWithoutUserInteraction(for:existingCredential:)` — non-interactive upgrade attempt
  - `viewDidLoad()` — set up intermediary UI for step-up flow
  - `prepareInterfaceToConvertAccountToSignInWithApple(for:existingCredential:)` — called before step-up view appears
- `ASAccountAuthenticationModificationExtensionContext` **[NEW]**
  - `getSignInWithAppleAuthorizationWithState(_:nonce:completionHandler:)` — requests Apple ID credential with nonce/state
  - `completeUpgradeToSignInWithApple()` — signals successful upgrade; removes password credential
  - `cancelRequest(withError:)` — cancels flow; errors: `.failed`, `.userInteractionRequired`, `.userCanceled`
- `ASCredentialServiceIdentifier` — identifies the service/account being upgraded
- `ASPasswordCredential` — existing username/password pair passed to extension
- `ASExtensionError` — error type with codes `.failed`, `.userInteractionRequired`, `.userCanceled`

## Code Highlights

Secure authorization request:
```swift
let request = ASAuthorizationAppleIDProvider().createRequest()
request.requestedScopes = [.fullName, .email]
request.nonce = myNonceString()   // unique per request
request.state = myStateString()   // for local request matching
let controller = ASAuthorizationController(authorizationRequests: [request])
controller.delegate = self
controller.performRequests()
```

SwiftUI button:
```swift
SignInWithAppleButton(.signIn) { request in
    request.requestedScopes = [.fullName, .email]
    request.nonce = myNonceString()
    request.state = myStateString()
} onCompletion: { result in
    switch result {
    case .success(let authorization): break  // handle
    case .failure(let error): break  // handle
    }
}.signInWithAppleButtonStyle(.black)
```

Credential state with `.transferred` handling:
```swift
provider.getCredentialState(forUserID: storedUserID) { state, error in
    switch state {
    case .authorized: /* proceed */
    case .revoked: /* sign out */
    case .notFound: /* show login */
    case .transferred: /* re-run sign-in with request.user = storedUserID */
    }
}
```

## Takeaways
- Always set `nonce` and `state` on authorization requests; verify the nonce server-side from the identity token JWT to prevent replay attacks.
- Call `getCredentialState` on every launch and foreground transition; handle the new `.transferred` state by re-running sign-in with the stored userIdentifier to get the updated team-scoped identifier.
- Server-to-server notifications (registered on the Apple Developer portal) let you react to `consent-revoked` and `account-delete` events without waiting for the user to reopen the app.
- The new `ASAccountAuthenticationModificationExtension` enables seamless account upgrades from password to Sign in with Apple, triggered automatically by iOS from Settings or AutoFill when a weak credential is detected.

---
_Source: WWDC20 Session 10173 page (abstract, chapter summaries, code samples, and resource links)._
