# What's New in Wallet and Apple Pay
**WWDC20 · Session 10662** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10662/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, Mac Catalyst, watchOS 7

## Overview
Apple Pay in 2020 expanded far beyond mobile Safari: it gained first-class support in native Mac apps and Mac Catalyst apps, arrived in App Clips for frictionless commerce, and received a comprehensive set of new button types and an automatic light/dark-mode button style. The session also introduced `PKSecureElementPass` as the unified replacement for `PKPaymentPass`, issuer provisioning extensions that let bank/card apps surface cards directly inside the Wallet app, and region-aware contact data formatting that standardizes address fields before they reach the merchant.

## Key Topics

### PKSecureElementPass (Replaces PKPaymentPass)
Apple Pay now covers transit, student IDs, and more in addition to credit/debit. The binding class was `PKPaymentPass`, but its "payment" name was too narrow. `PKSecureElementPass` is the unified successor — same properties, broader semantics. `PKPaymentPass` will be deprecated in a future release; developers should adopt `PKSecureElementPass` now.

### New PKPaymentButton Types and Automatic Style
`PKPaymentButton` gained context-specific button types to reflect the full range of purchase actions: **rent**, **tip**, **contribute**, **order**, **book**, **subscribe**, and more, in addition to the existing **buy** and **pay** types.

A new `PKPaymentButtonStyle.automatic` style automatically switches between light and dark appearances based on the current system appearance — no manual observation needed. Developers with specific background requirements can still use the explicit `.light`/`.dark` styles.

### Apple Pay in App Clips
App Clips enable transactional native experiences without a full app download. Apple Pay is the recommended payment method for App Clips because it is fast, secure, and requires no account creation. Best practices: use Apple Pay by default, offer guest checkout, integrate Sign In with Apple for post-purchase account creation, keep the clip lightweight, and load assets like maps on demand.

### Native Mac and Mac Catalyst Support
Apple Pay now works in Mac Catalyst apps and native (non-Catalyst) macOS apps — not just Safari/WebKit. `PKPaymentButton` including all new types and the automatic style is available on macOS. The authentication flow mirrors the web model: the app must establish a merchant session by posting to Apple's server.

A new **static merchant validation URL** replaces the old WebKit-callback approach: apps post to a single static URL rather than the dynamically provided URL from WebKit. This works for native and web payments alike; the old per-session URL approach will be removed in a future release.

`PKPaymentAuthorizationController` on Mac requires the delegate to implement `presentationWindow(for:)` returning the `UIWindow` that should host the payment sheet. Catalyst apps (which use the web security model) must implement `paymentAuthorizationController(_:didRequestMerchantSessionUpdate:)` instead of embedding merchant IDs in entitlements.

### Contact Data Formatting
Users accumulate years of inconsistently formatted address data in Apple Pay. Starting in iOS 14 and macOS Big Sur, the payment sheet applies basic formatting by region before delivering contact data to the merchant:
- Street and city: alphanumeric, punctuation, whitespace only
- State: two-letter code
- Zip/postal code: five- or nine-digit numeric (US); region-specific elsewhere
- ISO country code: uppercase two-letter code
- Validation errors (incomplete zip, invalid phone) are surfaced in the UI before Touch ID/Face ID authentication

Initial regions: Australia, Canada, United Kingdom, United States; more to follow.

### Issuer Provisioning Extensions
Card issuers can now surface cards directly inside the Wallet app via a two-extension model:

1. **Non-UI extension** — subclasses `PKIssuerProvisioningExtensionHandler`; reports extension status, available passes, and performs card-data lookups
2. **UI extension** (optional) — required only if re-authentication is needed; view controller conforms to `PKIssuerProvisioningExtensionAuthorizationProviding`

Both extensions require dedicated entitlements. Issuers interested in this feature must contact Apple to request entitlement access.

### WebKit and WKWebView Changes
`WKWebView` instances now behave more like `SFSafariViewController` — script injection outside app-defined domains is restricted to protect user privacy. For Apple Pay: previously, any script injection into a `WKWebView` blocked Apple Pay transactions on iOS; with the new restrictions, more pages will be free of injected scripts and therefore allowed to perform Apple Pay transactions.

## APIs & Frameworks

### PassKit
- `PKSecureElementPass` **[NEW]** — replaces `PKPaymentPass`; same properties, unified semantics across transit, student ID, payment
- `PKPaymentButtonType` — new values: `.rent`, `.tip`, `.contribute`, `.order`, `.book`, `.subscribe`, `.reload`, `.addMoney`, `.topUp`, `.pay`, `.support`, `.donate` **[NEW]**
- `PKPaymentButtonStyle.automatic` **[NEW]** — auto light/dark based on system appearance
- `PKPaymentAuthorizationController` — now available on macOS (native + Catalyst) **[NEW]**
- `PKPaymentAuthorizationControllerDelegate.presentationWindow(for:) -> UIWindow?` **[NEW — macOS]** — returns window for payment sheet presentation
- `PKPaymentAuthorizationControllerDelegate.paymentAuthorizationController(_:didRequestMerchantSessionUpdate:handler:)` **[NEW — Catalyst]** — requests merchant session; replaces entitlement-based merchant ID on Catalyst
- `PKPaymentRequestMerchantSessionUpdate` **[NEW]** — encapsulates merchant session result and status passed to the update handler
- `PKIssuerProvisioningExtensionHandler` **[NEW]** — base class for non-UI issuer extension
- `PKIssuerProvisioningExtensionAuthorizationProviding` **[NEW]** — protocol for UI issuer extension view controller

### Apple Pay on the Web
- Static merchant validation URL **[NEW]** — single URL for native and web payments; replaces dynamically provided WebKit URL

## Code Highlights

Mac/Catalyst payment authorization (presentation window + merchant session):
```swift
// PKPaymentAuthorizationControllerDelegate

func presentationWindow(for controller: PKPaymentAuthorizationController) -> UIWindow? {
    return yourViewController.view.window
}

func paymentAuthorizationController(
    _ controller: PKPaymentAuthorizationController,
    didRequestMerchantSessionUpdate handler: @escaping (PKPaymentRequestMerchantSessionUpdate) -> Void
) {
    // Fetch merchant session from your server, which obtains it from Apple
    if let dict = try? JSONSerialization.jsonObject(with: data, options: .allowFragments) as? [String: Any] {
        let session = PKPaymentMerchantSession(dictionary: dict)
        let update = PKPaymentRequestMerchantSessionUpdate(status: .success, merchantSession: session)
        handler(update)
    }
}
```

New button types and automatic style:
```swift
// Automatic style switches with system light/dark mode
let button = PKPaymentButton(paymentButtonType: .rent, paymentButtonStyle: .automatic)
```

## Takeaways

- `PKSecureElementPass` replaces `PKPaymentPass` as the unified class for all Secure Element-backed passes — adopt it now before `PKPaymentPass` is deprecated.
- Native Mac and Catalyst apps can now accept Apple Pay using `PKPaymentAuthorizationController` with the new static merchant validation URL; merchant session handling replaces entitlement-based merchant IDs on Catalyst.
- New `PKPaymentButtonType` values (rent, tip, contribute, subscribe, etc.) and the `automatic` style make it practical to match the button to the purchase action and support dark mode with zero extra code.
- Region-aware contact data formatting improves address accuracy for merchants in AU/CA/GB/US without any developer changes; use the Error APIs to surface invalid fields before authentication.
- Issuer provisioning extensions let bank apps surface their cards inside Wallet directly, expanding card discoverability beyond manual scan and in-app flows.

---
_Source: WWDC20 Session 10662 page (abstract, transcript, code samples, and resource links)._
