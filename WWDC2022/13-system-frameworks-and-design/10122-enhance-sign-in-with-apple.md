# Enhance your Sign in with Apple experience
**WWDC22 · Session 10122** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10122/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
Sign in with Apple provides fast, secure, one-tap account setup backed by Apple ID's two-factor authentication. This session shows how to prevent duplicate accounts by surfacing existing credentials at launch, how to manage the full lifecycle of Apple ID credentials (identity tokens, refresh tokens, session changes), and how to bring Sign in with Apple to websites and non-Apple platforms using a JavaScript framework or REST API.

The session covers two complementary angles: the native iOS/macOS Authentication Services API (including the new iOS 16 `preferImmediatelyAvailableCredentials` option) and the web-side Sign in with Apple JS framework plus Apple ID REST endpoints for authorization and token management.

Account deletion is highlighted as a new first-class concern — apps must provide an in-app deletion path, and a new REST revocation endpoint lets servers cleanly invalidate tokens and user sessions as part of that flow.

## Key Topics

**Preventing duplicate accounts** — Present both `ASAuthorizationAppleIDProvider` and `ASAuthorizationPasswordProvider` requests at launch using `.preferImmediatelyAvailableCredentials` (iOS 16+) so users are guided to an existing credential rather than creating a second account. The `Account Authentication Modification Extension` supports upgrading password accounts to Sign in with Apple.

**Apple ID credential deep dive** — `ASAuthorizationAppleIDCredential` fields: `user` (stable, team-scoped identifier), `fullName`, `email` (real or Hide My Email relay), `realUserStatus` (likelyReal / unknown / unsupported), `identityToken` (JWT signed by Apple), and `authorizationCode` (short-lived OAuth-style code). `fullName`, `email`, and `realUserStatus` are only returned on initial account creation — cache them until server-side account creation is confirmed.

**Token and session management** — Exchange `authorizationCode` for refresh/access tokens via the `auth/token` REST endpoint. Monitor credential state with `getCredentialState(forUserID:)` and observe `credentialRevokedNotification`. Subscribe to server-to-server notifications for `email-disabled`, `consent-revoked`, and `account-delete` events.

**Account deletion** — New `auth/revoke` REST endpoint accepts a refresh token or access token to instantly invalidate all sessions associated with the user's account in your app.

**Sign in with Apple on the web** — Configure a Services ID in the Apple Developer Portal with registered domains and a redirect URI. Embed the Sign in with Apple JS framework to render a customizable button and handle authorization via `AppleID.auth.init(...)` and DOM events (`AppleIDSignInOnSuccess`, `AppleIDSignInOnFailure`). Alternatively, use the REST authorize endpoint directly.

## APIs & Frameworks

### AuthenticationServices (native)
- `ASAuthorizationController` — orchestrates authorization requests **[existing]**
- `ASAuthorizationAppleIDProvider` — creates Sign in with Apple requests
- `ASAuthorizationAppleIDProvider.createRequest()` — returns `ASAuthorizationAppleIDRequest`
- `ASAuthorizationPasswordProvider` — creates password credential requests
- `ASAuthorizationPasswordProvider.createRequest()` — surfaces existing keychain passwords
- `ASAuthorizationController.performRequests(options:)` — new `options:` parameter **[NEW]**
- `ASAuthorizationController.PerformRequestsOptions.preferImmediatelyAvailableCredentials` **[NEW]** — only surfaces immediately available credentials; intended for app launch
- `ASAuthorizationControllerDelegate.authorizationController(_:didCompleteWithAuthorization:)`
- `ASAuthorizationControllerDelegate.authorizationController(_:didCompleteWithError:)`
- `ASAuthorizationAppleIDCredential` — result object containing `user`, `fullName`, `email`, `realUserStatus`, `identityToken`, `authorizationCode`
- `ASAuthorizationAppleIDCredential.realUserStatus` — `.likelyReal`, `.unknown`, `.unsupported`
- `ASPasswordCredential` — returned when user selects a password-based account
- `ASAuthorizationAppleIDProvider.getCredentialState(forUserID:completionHandler:)` — returns `.authorized`, `.revoked`, `.notFound`, `.transferred`
- `ASAuthorizationAppleIDProvider.credentialRevokedNotification` — `NotificationCenter` notification name

### Sign in with Apple REST API
- `POST /auth/token` — exchange `authorizationCode` for `refresh_token`, `access_token`, `id_token`
- `POST /auth/revoke` **[NEW]** — revoke a refresh or access token to delete user account association
- `GET /auth/authorize` — REST-based authorization request (web/non-Apple platforms)
- Server-to-server notification events: `email-disabled`, `consent-revoked`, `account-delete`

### Sign in with Apple JS (web)
- `appleid.auth.js` — CDN-hosted JavaScript framework
- `AppleID.auth.init({ clientId, scope, redirectURI, state, nonce, usePopup })` — initialize and trigger authorization
- `AppleIDSignInOnSuccess` DOM event
- `AppleIDSignInOnFailure` DOM event
- Apple ID Button API — REST endpoints for center-aligned, left-aligned, and logo-only button PNG images
- `data-color`, `data-border`, `data-type`, `data-mode` HTML attributes for button customization

## Code Highlights

Presenting existing credentials at launch (iOS 16+):
```swift
let controller = ASAuthorizationController(authorizationRequests: [
    ASAuthorizationAppleIDProvider().createRequest(),
    ASAuthorizationPasswordProvider().createRequest()
])
controller.delegate = self
controller.presentationContextProvider = self
if #available(iOS 16.0, *) {
    controller.performRequests(options: .preferImmediatelyAvailableCredentials)
} else {
    controller.performRequests()
}
```

Checking credential state on app launch:
```swift
let appleIDProvider = ASAuthorizationAppleIDProvider()
appleIDProvider.getCredentialState(forUserID: "currentUserIdentifier") { (credentialState, error) in
    switch credentialState {
    case .authorized: break   // valid session
    case .revoked:    break   // sign out
    case .notFound:   break   // show login UI
    case .transferred: break  // team transferred
    }
}
```

Web integration with Sign in with Apple JS:
```html
<script src="https://appleid.cdn-apple.com/appleauth/static/jsapi/appleid/1/en_US/appleid.auth.js"></script>
<div id="appleid-signin" data-color="black" data-border="true" data-type="sign in"/>
<script>
  AppleID.auth.init({ clientId: '[CLIENT_ID]', scope: '[SCOPES]',
    redirectURI: '[REDIRECT_URI]', state: '[STATE]', nonce: '[NONCE]', usePopup: true });
</script>
```

## Takeaways
- Use `.preferImmediatelyAvailableCredentials` (iOS 16) at app launch to surface existing Apple ID or password credentials and prevent duplicate accounts.
- Cache `fullName`, `email`, and `realUserStatus` from the credential — they are only provided on first account creation, not on subsequent sign-ins.
- Implement server-to-server notifications and the new `auth/revoke` endpoint to handle session revocation and app-initiated account deletion cleanly.
- Sign in with Apple on the web uses the same OAuth concepts (authorizationCode, identityToken) as the native SDK — configure a Services ID and embed the Sign in with Apple JS framework to get a first-class Safari experience.

---
_Source: WWDC22 Session 10122 page (abstract, chapter summaries, code samples, and resource links)._
