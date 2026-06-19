# What's New in Authentication
**WWDC19 · Session 516** · [Watch](https://developer.apple.com/videos/play/wwdc2019/516/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, Safari 13

## Overview
Authentication in iOS 13 and macOS Catalina receives significant improvements across five areas: the introduction of Sign In with Apple, a new streamlined password sign-in API in Authentication Services, Password AutoFill for iPad apps running on Mac, new features for `ASWebAuthenticationSession` (now available on macOS), and USB FIDO2 security key support via WebAuthentication in Safari 13.

The session's core message is that apps should make sign-in a one-tap experience. The Authentication Services framework now provides a single `ASAuthorizationController` that can simultaneously check for Sign In with Apple credentials, saved passwords, and other credential types, removing the need for a traditional login UI when the user already has a valid credential.

A secondary focus is weak password management: Safari 13 proactively prompts users to upgrade weak passwords at sign-in time and can navigate them directly to a site's Change Password page via the well-known URL standard (`/.well-known/change-password`).

## Key Topics

**Sign In with Apple**
A privacy-first single sign-on system backed by the user's Apple ID and strong two-factor authentication. Every account is verified, includes a fraud-confidence indicator, and can use Hide My Email for email relay. Available on iOS, iPadOS, macOS, watchOS, tvOS, and the web. Introduced fully in Session 706; this session provides the integration context.

**Streamlined Password Sign-In via Authentication Services**
New in iOS 13: `ASAuthorizationPasswordProvider` and `ASAuthorizationPasswordRequest` allow apps to surface a one-tap credential picker (same UI as Sign In with Apple) even for password-based accounts. Combined with `ASAuthorizationController`, apps can request both Sign In with Apple and passwords in a single call, presenting whichever is available.

**Password AutoFill for iPad Apps for Mac**
iPad apps brought to Mac via Catalyst automatically get Password AutoFill if the app's new Mac App ID is listed in the associated domains / Apple App Site Association file. For `webcredentials`, add the new app ID to the `apps` array. For universal links, use the new `appIDs` key (array of strings) introduced in iOS 13, while keeping the existing `apps` key for backward compatibility with iOS 12.

**ASWebAuthenticationSession Improvements**
The OAuth/web SSO session API gains macOS Catalina support (uses the user's preferred browser if it supports the API). New features:
- `prefersEphemeralWebBrowserSession = true` — skips sharing Safari cookies/data for a more private, non-SSO experience and removes the confirmation dialog
- `presentationContextProvider` — required for multi-window support on iPadOS and macOS; must provide an `ASPresentationAnchor` (window)
- `SFAuthenticationSession` (from Safari Services) is deprecated; switch to `ASWebAuthenticationSession`

**Weak Password Upgrade Flow**
Safari 13 detects weak passwords at sign-in and offers to navigate users to the Change Password page. Sites implement a redirect at `/.well-known/change-password` (client or server-side) pointing to their actual change-password URL. Twitter, GitHub, and WordPress.com are early adopters.

**USB Security Keys / WebAuthentication**
Safari 13 on macOS Catalina adds support for USB-based FIDO2-compliant security keys via the W3C WebAuthentication standard. Available as an experimental feature in Seed 1, enabled by default in Seed 2. Requires a robust account recovery story before adoption.

## APIs & Frameworks

**AuthenticationServices**
- `ASAuthorizationAppleIDProvider` — Sign In with Apple credential provider (see Session 706)
- `ASAuthorizationPasswordProvider` **[NEW]** — provides saved-password credential requests
- `ASAuthorizationPasswordRequest` **[NEW]** — request type for password-based credentials
- `ASAuthorizationController` — coordinates multiple credential request types simultaneously **[updated to support password requests]**
- `ASAuthorizationControllerDelegate` — `authorizationController(_:didCompleteWithAuthorization:)` receives `ASAuthorizationPasswordCredential` or `ASAuthorizationAppleIDCredential`
- `ASAuthorizationPasswordCredential` **[NEW]** — contains `user` (username) and `password`
- `ASWebAuthenticationSession` — OAuth / web SSO session
  - `prefersEphemeralWebBrowserSession: Bool` **[NEW]** — ephemeral (no shared cookies) mode; skips confirmation dialog
  - `presentationContextProvider: ASWebAuthenticationPresentationContextProviding` **[NEW]** — required for multi-window apps
- `ASWebAuthenticationPresentationContextProviding` **[NEW]** — protocol; `presentationAnchor(for:)` returns `ASPresentationAnchor`
- `SFAuthenticationSession` (Safari Services) — **DEPRECATED**; migrate to `ASWebAuthenticationSession`

**Associated Domains / Apple App Site Association**
- `webcredentials` service type in apple-app-site-association — add Mac app ID to `apps` array for Catalyst Password AutoFill
- `appIDs` key **[NEW in iOS 13]** — array of app IDs for universal links; keeps `apps` key for iOS 12 backward compatibility

**Web Standards**
- `/.well-known/change-password` redirect **[NEW proposal]** — server-side or client-side redirect to the Change Password page
- WebAuthentication (W3C) with USB FIDO2 devices — supported in Safari 13 macOS Catalina

## Code Highlights

Requesting both Sign In with Apple and saved passwords simultaneously:

```swift
import AuthenticationServices

func requestCredentials() {
    let appleRequest = ASAuthorizationAppleIDProvider().createRequest()
    appleRequest.requestedScopes = [.email, .fullName]

    let passwordRequest = ASAuthorizationPasswordProvider().createRequest()

    let controller = ASAuthorizationController(authorizationRequests: [appleRequest, passwordRequest])
    controller.delegate = self
    controller.presentationContextProvider = self
    controller.performRequests()
}

func authorizationController(controller: ASAuthorizationController,
                              didCompleteWithAuthorization authorization: ASAuthorization) {
    switch authorization.credential {
    case let credential as ASAuthorizationAppleIDCredential:
        // Handle Sign In with Apple
        signIn(with: credential.user)
    case let credential as ASAuthorizationPasswordCredential:
        // Handle saved password
        signIn(username: credential.user, password: credential.password)
    default: break
    }
}
```

Ephemeral OAuth session (no shared browser cookies):

```swift
let session = ASWebAuthenticationSession(url: authURL, callbackURLScheme: "myapp") { url, error in
    // handle callback
}
session.prefersEphemeralWebBrowserSession = true
session.presentationContextProvider = self
session.start()
```

## Takeaways
- The new `ASAuthorizationPasswordProvider` + `ASAuthorizationController` pattern lets apps offer a one-tap sign-in picker for both Sign In with Apple and saved passwords without showing a traditional login form.
- If your app uses Catalyst, add the Mac app ID to your apple-app-site-association file to enable Password AutoFill on Mac — no other code changes required.
- Implement `/.well-known/change-password` as a simple redirect; Safari 13 will use it automatically when prompting users to upgrade weak passwords.
- `prefersEphemeralWebBrowserSession = true` removes the shared-cookie confirmation dialog and is the right choice for OAuth flows where SSO is not desired.

---
_Source: WWDC19 Session 516 page (abstract, transcript, and resource links)._
