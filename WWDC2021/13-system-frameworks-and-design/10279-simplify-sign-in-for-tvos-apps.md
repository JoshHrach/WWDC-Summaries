# Simplify Sign In for Your tvOS Apps
**WWDC21 · Session 10279** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10279/)

_Platforms:_ tvOS 15, iOS 15, iPadOS 15

## Overview
Typing passwords on Apple TV with a remote is one of the most frustrating experiences in the tvOS ecosystem. tvOS 15 introduces a new system sign-in view backed by an iPhone/iPad handoff flow that lets users sign in on Apple TV using Face ID or Touch ID on their nearby iOS/iPadOS device — without typing a single character on the TV.

The session presents the complete implementation: Associated Domains configuration for the tvOS app, the `ASAuthorizationController` request flow, handling both success and error cases, and a new `customAuthorizationMethods` API for adding additional sign-in options (manual entry, TV provider, restore purchase) to the system sign-in view.

## Key Topics

### The System Sign-In View (NEW in tvOS 15)
- A consistent, system-provided sign-in UI displayed by tvOS apps.
- Informs users they can complete the sign-in on their iPhone or iPad.
- Shows a notification on the nearby iOS/iPadOS device; user taps it, sees iCloud Keychain credential suggestions, authenticates with Face ID/Touch ID, and the credential is securely passed back to Apple TV.
- Developer adopts this by using `ASAuthorizationController` — the same API used on iOS and macOS.

### Configuration: Associated Domains
- The tvOS app's `appID` must be listed in the `webcredentials` key of the domain's Apple App Site Association (AASA) file.
- Add the **Associated Domains** capability to the tvOS Xcode target.
- Add `webcredentials:yourdomain.com` to the associated domains list.
- This secure link allows the iPhone/iPad to safely suggest matching iCloud Keychain credentials for your domain.

### ASAuthorizationController Flow
1. Create `ASAuthorizationPasswordProvider().createRequest()` (and optionally `ASAuthorizationAppleIDProvider().createRequest()` for Sign in with Apple).
2. Instantiate `ASAuthorizationController(authorizationRequests:)` with the request array.
3. Set `controller.delegate = self`.
4. Call `controller.performRequests()` — tvOS displays the system sign-in view and initiates the iPhone/iPad handoff.

### Custom Authorization Methods (NEW)
- `ASAuthorizationController.customAuthorizationMethods` property — array of `ASAuthorizationCustomMethod` values.
- `.other` — shows a generic alternate sign-in option (use for manual username/password entry or a custom sign-in selector).
- `.videoSubscriberAccount` — shows a TV provider sign-in option.
- `.restorePurchase` — shows a Restore Purchase option.
- When the user selects a custom method, `authorizationController(_:didCompleteWithCustomMethod:)` is called on the delegate.

### Delegate Methods
- `authorizationController(_:didCompleteWithAuthorization:)` — credential delivered; cast to `ASPasswordCredential` (or `ASAuthorizationAppleIDCredential`) to extract `user` and `password`.
- `authorizationController(_:didCompleteWithError:)` — check for `ASAuthorizationError.canceled` to silently return; otherwise show a retry error to the user.

### Best Practices
- Provide a single "Sign In" button that directly triggers `ASAuthorizationController.performRequests()`.
- Limit the number of custom sign-in options to avoid decision paralysis.
- Always include the iPhone/iPad handoff flow (via `ASAuthorizationPasswordProvider`) as the primary option.

## APIs & Frameworks

- `AuthenticationServices` framework
- `ASAuthorizationController` — controller for authorization requests
- `ASAuthorizationController(authorizationRequests:)` — initializer with array of requests
- `ASAuthorizationPasswordProvider` — provides password-based requests
- `ASAuthorizationPasswordProvider().createRequest()` — creates password credential request
- `ASAuthorizationAppleIDProvider().createRequest()` — optional Sign in with Apple request
- `ASAuthorizationController.performRequests()` — triggers the system sign-in UI
- `ASAuthorizationController.customAuthorizationMethods` **[NEW]** — `[ASAuthorizationCustomMethod]`
- `ASAuthorizationCustomMethod` **[NEW]** enum — `.other`, `.videoSubscriberAccount`, `.restorePurchase`
- `ASAuthorizationControllerDelegate` protocol
- `authorizationController(_:didCompleteWithAuthorization:)` — success callback
- `authorizationController(_:didCompleteWithError:)` — failure callback
- `authorizationController(_:didCompleteWithCustomMethod:)` **[NEW]** — custom method selection callback
- `ASAuthorization` — authorization result container
- `ASPasswordCredential` — `user` and `password` properties
- `ASAuthorizationError.canceled` — user dismissed sign-in UI
- Associated Domains capability — `webcredentials:yourdomain.com`
- Apple App Site Association (AASA) file — `webcredentials` key

## Code Highlights

Basic password sign-in on tvOS:
```swift
let controller = ASAuthorizationController(authorizationRequests: [
    ASAuthorizationPasswordProvider().createRequest()
])
controller.delegate = self
controller.performRequests()
```

Handling the credential:
```swift
func authorizationController(controller: ASAuthorizationController,
    didCompleteWithAuthorization authorization: ASAuthorization) {
    if let credential = authorization.credential as? ASPasswordCredential {
        // Use credential.user and credential.password to sign in
    }
}
```

Adding custom sign-in options:
```swift
controller.customAuthorizationMethods = [.other, .restorePurchase]
```

Handling a custom method selection:
```swift
func authorizationController(controller: ASAuthorizationController,
    didCompleteWithCustomMethod customMethod: ASAuthorizationCustomMethod) {
    switch customMethod {
    case .other: showManualSignIn()
    case .restorePurchase: restorePurchase()
    default: break
    }
}
```

## Takeaways
- The new tvOS 15 system sign-in view + iPhone/iPad handoff eliminates text-entry friction; adopt it with a few lines of `ASAuthorizationController` code.
- Associated Domains must be configured on the tvOS target (not just iOS) for iCloud Keychain credential suggestions to work.
- Use `customAuthorizationMethods` to surface additional sign-in types without building a custom sign-in selector UI from scratch.
- A single "Sign In" button leading directly to `performRequests()` is the recommended UX — do not fragment the entry flow.

---
_Source: WWDC21 Session 10279 page (abstract, chapter summaries, code samples, and resource links)._
