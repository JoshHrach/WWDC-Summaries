# Introducing Sign In with Apple
**WWDC19 · Session 706** · [Watch](https://developer.apple.com/videos/play/wwdc2019/706/)

_Platforms:_ iOS 13, iPadOS 13, macOS 10.15, tvOS 13, watchOS 6, Web (JavaScript)

## Overview
Sign In with Apple is a new authentication mechanism letting users sign into third-party apps with their existing Apple ID in one tap, without creating a new password. The system pre-fills the user's name and email, performs Face ID or Touch ID biometric confirmation, and returns a stable unique identifier plus a verified email address and a JWT identity token. Every account is automatically 2FA-protected at no cost to the developer or the user.

From a privacy perspective, Apple provides a "Hide My Email" relay option so users can share a randomly generated email address that forwards to their real inbox without exposing it. Apple does not track user interaction with third-party apps after authentication. From a security perspective, the system also returns a boolean "real user indicator" (`.likelyReal` vs. `.unknown`) derived from on-device signals to help apps detect automated bot sign-ups.

The AuthenticationServices framework provides a native Swift API on all Apple platforms and a JavaScript API for web and other platforms (Android, Windows).

## Key Topics

**User Experience Flow**
1. User taps `ASAuthorizationAppleIDButton` in the app.
2. A system sheet appears pre-filled with name and email (user can edit or hide email).
3. User confirms with Face ID / Touch ID. No password is entered.
4. App receives an `ASAuthorization` with `ASAppleIDCredential`.

**ASAppleIDCredential Contents**
- `user: String` — stable, team-scoped user identifier (persists across devices, reinstalls, and platforms).
- `identityToken: Data` — short-lived signed JWT for server-side validation.
- `authorizationCode: Data` — one-time code to exchange for a refresh token on the server.
- `fullName: PersonNameComponents?` — only returned on first sign-in (nil on subsequent sign-ins; store it).
- `email: String?` — verified email or relay address (only returned on first sign-in).
- `realUserStatus: ASUserDetectionStatus` — `.likelyReal`, `.unknown`, or `.unsupported`.

**Credential State Checking**
Call `ASAuthorizationAppleIDProvider.getCredentialState(forUserID:completion:)` on every app launch using the stored user identifier. Results: `.authorized` (proceed normally), `.revoked` (sign user out), `.notFound` (user has no relationship — show login).
Subscribe to `ASAuthorizationAppleIDProvider.credentialRevokedNotification` to handle revocations while the app is running.

**Quick Sign-In (Existing Accounts)**
Call `ASAuthorizationController` with both an `ASAuthorizationAppleIDRequest` and an `ASAuthorizationPasswordRequest` on viewDidAppear before showing the login UI. If either credential exists (prior Apple ID sign-in or iCloud Keychain password), the system will surface it in a non-intrusive sheet. Handle `ASPasswordCredential` in the delegate for Keychain passwords. This prevents duplicate account creation.

**Email Privacy (Hide My Email)**
If the user selects "Hide My Email," the delivered email is an Apple-generated relay address. The relay forwards emails to the user's real address and handles replies. Apple does not retain message content. The developer must whitelist their email-sending domains in the Developer Portal for the relay to forward successfully.

**Cross-Platform (JavaScript API)**
Include the Apple JS library, add a `<div>` with the sign-in button configuration parameters (`data-redirect-uri`, `data-scope`, `data-client-id`). After sign-in completes, Apple POSTs form-encoded results to the redirect URI containing the user identifier, identity token, auth code, and optionally name/email. Safari presents a native Apple Pay-style sheet; other browsers redirect to Apple's sign-in page.

**Capability Setup**
Add "Sign In with Apple" capability in Xcode project settings (or in the Apple Developer portal). Add "Associated Domains" capability if also requesting iCloud Keychain passwords, to narrow auto-filled credentials to the app's domains.

**Best Practices**
- Don't require sign-in to use the app at all — let users explore before needing an account.
- Only request `fullName` and `email` scopes if your app actually requires them.
- Name and email are only delivered once (at first sign-in). Store them immediately.
- Respect the user's email choice — if they gave a relay address, use it; don't ask for their real email.
- Call credential state check at every launch; do not rely on local state.
- Implement on all platforms where your app ships — users expect parity.

## APIs & Frameworks

**AuthenticationServices** (iOS 13, macOS 10.15, tvOS 13, watchOS 6) **[NEW]**

Button:
- `ASAuthorizationAppleIDButton` **[NEW]**
  - `ASAuthorizationAppleIDButton.ButtonType`: `.signIn`, `.continue`, `.default` **[NEW]**
  - `ASAuthorizationAppleIDButton.Style`: `.black`, `.white`, `.whiteOutline` **[NEW]**

Authorization request and controller:
- `ASAuthorizationAppleIDProvider` **[NEW]**
  - `ASAuthorizationAppleIDProvider.createRequest() -> ASAuthorizationAppleIDRequest` **[NEW]**
- `ASAuthorizationAppleIDRequest` **[NEW]**
  - `requestedScopes: [ASAuthorization.Scope]?` — `.fullName`, `.email` **[NEW]**
- `ASAuthorizationPasswordProvider` **[NEW]** — for iCloud Keychain password request
- `ASAuthorizationPasswordRequest` **[NEW]**
- `ASAuthorizationController(authorizationRequests:)` **[NEW]**
  - `ASAuthorizationController.delegate: ASAuthorizationControllerDelegate?` **[NEW]**
  - `ASAuthorizationController.presentationContextProvider: ASAuthorizationControllerPresentationContextProviding?` **[NEW]**
  - `ASAuthorizationController.performRequests()` **[NEW]**

Delegate:
- `ASAuthorizationControllerDelegate` **[NEW]**
  - `authorizationController(_:didCompleteWithAuthorization:)` **[NEW]**
  - `authorizationController(_:didCompleteWithError:)` **[NEW]**
- `ASAuthorizationControllerPresentationContextProviding` **[NEW]**
  - `presentationAnchor(for:) -> ASPresentationAnchor` **[NEW]**

Credential types:
- `ASAuthorization.credential` — typed credential
- `ASAppleIDCredential` **[NEW]**
  - `.user: String` **[NEW]**
  - `.identityToken: Data?` **[NEW]**
  - `.authorizationCode: Data?` **[NEW]**
  - `.fullName: PersonNameComponents?` **[NEW]**
  - `.email: String?` **[NEW]**
  - `.realUserStatus: ASUserDetectionStatus` **[NEW]**
- `ASPasswordCredential` **[NEW]** (from iCloud Keychain)
  - `.user: String` **[NEW]**
  - `.password: String` **[NEW]**

Credential state:
- `ASAuthorizationAppleIDProvider.getCredentialState(forUserID:completion:)` **[NEW]**
- `ASAuthorizationAppleIDProvider.credentialRevokedNotification: Notification.Name` **[NEW]**
- `ASAuthorizationAppleIDProvider.CredentialState`: `.authorized`, `.revoked`, `.notFound`, `.transferred` **[NEW]**

Real user indicator:
- `ASUserDetectionStatus`: `.likelyReal`, `.unknown`, `.unsupported` **[NEW]**

## Code Highlights

Adding the Sign In with Apple button:
```swift
import AuthenticationServices

let button = ASAuthorizationAppleIDButton(authorizationButtonType: .signIn,
                                          authorizationButtonStyle: .black)
button.addTarget(self, action: #selector(handleAuthorizationAppleIDButtonPress), for: .touchUpInside)
loginProviderStackView.addArrangedSubview(button)
```

Performing the authorization request:
```swift
@objc func handleAuthorizationAppleIDButtonPress() {
    let appleIDProvider = ASAuthorizationAppleIDProvider()
    let request = appleIDProvider.createRequest()
    request.requestedScopes = [.fullName, .email]
    
    let controller = ASAuthorizationController(authorizationRequests: [request])
    controller.delegate = self
    controller.presentationContextProvider = self
    controller.performRequests()
}
```

Handling the credential in the delegate:
```swift
func authorizationController(controller: ASAuthorizationController,
                             didCompleteWithAuthorization authorization: ASAuthorization) {
    if let appleIDCredential = authorization.credential as? ASAppleIDCredential {
        let userID = appleIDCredential.user
        // Store userID in Keychain for future credential state checks
        // fullName and email are only delivered on first sign-in — store them now
        let fullName = appleIDCredential.fullName
        let email = appleIDCredential.email
        let realUserStatus = appleIDCredential.realUserStatus
    }
}
```

Quick sign-in on app launch (existing credentials):
```swift
func performExistingAccountSetupFlows() {
    let appleIDRequest = ASAuthorizationAppleIDProvider().createRequest()
    let passwordRequest = ASAuthorizationPasswordProvider().createRequest()
    
    let controller = ASAuthorizationController(authorizationRequests: [appleIDRequest, passwordRequest])
    controller.delegate = self
    controller.presentationContextProvider = self
    controller.performRequests()
}
```

Checking credential state at launch:
```swift
let provider = ASAuthorizationAppleIDProvider()
provider.getCredentialState(forUserID: storedUserID) { credentialState, error in
    switch credentialState {
    case .authorized:
        break // user is still signed in, proceed normally
    case .revoked:
        // sign user out, show login
        break
    case .notFound:
        // show login UI
        break
    }
}
```

## Takeaways
- Name and email are returned only on the first sign-in — immediately persist them to your backend or Keychain; subsequent sign-ins return nil for those fields.
- Call `getCredentialState` on every app launch using the stored `user` identifier; never rely on locally cached sign-in state alone, because the user can revoke access at any time from Settings.
- Include both `ASAuthorizationAppleIDRequest` and `ASAuthorizationPasswordRequest` in the startup call to surface existing credentials, preventing duplicate accounts and providing a one-tap re-entry on new devices.
- The `realUserStatus` field (`.likelyReal` vs `.unknown`) provides a privacy-preserving fraud signal requiring no additional user data — route new `.likelyReal` accounts through a streamlined onboarding, while `.unknown` accounts get the standard verification flow.

---
_Source: WWDC19 Session 706 page (transcript, chapter summaries, and resource links)._
