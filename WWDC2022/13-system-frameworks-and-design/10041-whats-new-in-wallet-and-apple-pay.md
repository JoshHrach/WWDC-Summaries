# What's new in Wallet and Apple Pay
**WWDC22 · Session 10041** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10041/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9

## Overview
Wallet and Apple Pay received four major additions in 2022: new SwiftUI buttons for Wallet integration, multi-merchant payment support for marketplaces and travel apps, a revamped automatic payments API (subscriptions, installments, and store card top-ups), and a new Order Tracking feature so customers can monitor purchases directly from Wallet. Additionally, iOS 16 introduced the Identity Verification API, allowing apps and App Clips to securely request age verification or identity data from IDs in Wallet (driver's licenses and State IDs in supported US states).

The macOS Apple Pay payment sheet was also redesigned using SwiftUI, bringing parity with the iOS sheet redesign from the prior year. All new features work on Mac as well.

## Key Topics

### SwiftUI PassKit Buttons (New)
`AddPassToWalletButton` and `PayWithApplePayButton` are new SwiftUI views that replace UIKit-based button integration. They support customizable sizes and styles and include a `fallback` view for devices that don't support the feature. `PayWithApplePayButton` takes a `PKPaymentRequest` and an `onPaymentAuthorizationChange` closure handling all phases.

### Multi-Merchant Payments
`PKPaymentTokenContext` **[NEW]** — describes a sub-merchant in a multi-merchant transaction (identifier, name, domain, amount). Set as `paymentRequest.multiTokenContexts`. Each merchant receives its own payment token. The payment sheet shows a per-merchant breakdown. Sum of sub-merchant amounts must be ≤ total payment request amount.

### Automatic Payments
Two new automatic payment types: **Recurring** (`PKRecurringPaymentRequest`, `PKRecurringPaymentSummaryItem`) for subscriptions/installments with optional trial period, and **Automatic Reload** (`PKAutomaticReloadPaymentRequest`, `PKAutomaticReloadPaymentSummaryItem`) for balance top-ups with a threshold amount. Both types can receive an **Apple Pay Merchant Token** — a token tied to the user's Apple ID rather than a specific device, improving reliability for ongoing charges. Token lifecycle notifications are sent to a `tokenNotificationURL` via the Apple Pay Merchant Token Management API.

### Order Tracking
Apps create an **Order Type ID** in the developer account and build cryptographically signed **order packages** (JSON + images + localizations, compressed). After an Apple Pay transaction, the app returns `PKPaymentOrderDetails` (Order Type ID, Order ID, web service URL, authentication token) in the `PKPaymentAuthorizationResult`. The device asynchronously downloads the order package from the merchant's server. Orders support push notification updates, iCloud sync (end-to-end encrypted), and display shipping/tracking info, line items, and a barcode for pickup orders.

### Identity Verification with IDs in Wallet
New `PKIdentityRequest` / `PKIdentityDriversLicenseDescriptor` API lets apps request specific data elements from a user's driver's license or State ID. Privacy-preserving: apps can request a Boolean `age(atLeast:)` element rather than the full date of birth. Users review and approve via Face ID / Touch ID before any data is shared. Response is encrypted with the merchant's encryption certificate and must be decrypted and verified on the developer's server using HPKE (RFC 9180) and the mdoc format (ISO 18013-5).

## APIs & Frameworks

**PassKit — SwiftUI** **[NEW]**
- `AddPassToWalletButton` **[NEW]** — SwiftUI button to add passes
- `AddPassToWalletButtonStyle` — `.black`, `.blackOutline`
- `PayWithApplePayButton` **[NEW]** — SwiftUI Apple Pay button
- `PayWithApplePayButtonLabel` — 17 label options (`.plain`, `.buy`, `.checkout`, etc.)
- `PayWithApplePayButtonStyle` — `.automatic`, `.black`, `.white`, `.whiteOutline`
- `PayWithApplePayButtonPaymentAuthorizationPhase` **[NEW]** — enum for auth phases
- `VerifyIdentityWithWalletButton` **[NEW]** — SwiftUI identity verification button
- `VerifyIdentityWithWalletButtonLabel` **[NEW]** — `.verifyAge`, `.verifyIdentity`, etc.

**PassKit — Multi-Merchant Payments** **[NEW]**
- `PKPaymentTokenContext` **[NEW]** — sub-merchant context (identifier, name, domain, amount)
- `PKPaymentRequest.multiTokenContexts` **[NEW]** — array of `PKPaymentTokenContext`

**PassKit — Automatic Payments** **[NEW]**
- `PKRecurringPaymentRequest` **[NEW]** — recurring payment descriptor
  - `regularBilling`, `trialBilling`, `billingAgreement`, `managementURL`, `tokenNotificationURL`
- `PKRecurringPaymentSummaryItem` **[NEW]** — summary item with `startDate`/`endDate`
- `PKAutomaticReloadPaymentRequest` **[NEW]** — automatic reload descriptor
  - `automaticReloadBilling`, `billingAgreement`, `managementURL`, `tokenNotificationURL`
- `PKAutomaticReloadPaymentSummaryItem` **[NEW]** — summary item with `thresholdAmount`
- `PKPaymentRequest.recurringPaymentRequest` **[NEW]**
- `PKPaymentRequest.automaticReloadPaymentRequest` **[NEW]**
- Apple Pay Merchant Token Management API (server-side) **[NEW]**

**PassKit — Order Tracking** **[NEW]**
- `PKPaymentOrderDetails` **[NEW]** — order details returned after Apple Pay authorization
  - `orderTypeIdentifier`, `orderIdentifier`, `webServiceURL`, `authenticationToken`
- `PKPaymentAuthorizationResult.orderDetails` **[NEW]**
- Wallet Orders framework / Order Package format **[NEW]**

**PassKit — Identity Verification** **[NEW]**
- `PKIdentityRequest` **[NEW]** — identity verification request
  - `descriptor`, `merchantIdentifier`, `nonce`
- `PKIdentityDriversLicenseDescriptor` **[NEW]** — specifies requested data elements
  - `addElements(_:intentToStore:)` **[NEW]**
  - `PKIdentityElement.age(atLeast:)` **[NEW]** — privacy-preserving age check
  - `PKIdentityElement.givenName`, `.familyName`, `.portrait`, `.dateOfBirth`, `.address`, etc.
  - `PKIdentityIntentToStore.willNotStore` / `.mayStore(days:)` **[NEW]**
- `PKIdentityDocument` **[NEW]** — encrypted response document
- `PKIdentityAuthorizationController` **[NEW]** — UIKit equivalent
- `PKIdentityButton` **[NEW]** — UIKit identity button
- `PKIdentityError` **[NEW]** — `.cancelled`, etc.

## Code Highlights

Multi-merchant payment request:
```swift
let multiTokenContexts = [
    PKPaymentTokenContext(
        merchantIdentifier: "com.example.hotel",
        externalIdentifier: "com.example.hotel",
        merchantName: "Hotel",
        merchantDomain: "hotel.example.com",
        amount: 300)
]
paymentRequest.multiTokenContexts = multiTokenContexts
```

Recurring subscription setup:
```swift
let recurringPaymentRequest = PKRecurringPaymentRequest(
    paymentDescription: "Book Club Membership",
    regularBilling: regularBilling,
    managementURL: URL(string: "https://example.com/manage")!)
recurringPaymentRequest.trialBilling = trialBilling
paymentRequest.recurringPaymentRequest = recurringPaymentRequest
```

Identity verification button with age check:
```swift
let descriptor = PKIdentityDriversLicenseDescriptor()
descriptor.addElements([.age(atLeast: 18)], intentToStore: .willNotStore)
let request = PKIdentityRequest()
request.descriptor = descriptor
request.merchantIdentifier = "..."
request.nonce = serverNonce

VerifyIdentityWithWalletButton(.verifyIdentity, request: request) { result in
    if case .success(let document) = result {
        // send document.encryptedData to server
    }
} fallback: { /* alternate UI */ }
```

## Takeaways
- `AddPassToWalletButton`, `PayWithApplePayButton`, and `VerifyIdentityWithWalletButton` make SwiftUI integration with Wallet first-class with just a few lines of code.
- Multi-merchant payments (`PKPaymentTokenContext`) allow marketplaces to request separate payment tokens for each sub-merchant, improving user privacy and security.
- `PKRecurringPaymentRequest` and `PKAutomaticReloadPaymentRequest` power managed subscriptions and store-card top-ups, with Apple Pay Merchant Tokens (tied to Apple ID) replacing device-specific tokens for better reliability.
- The new Order Tracking API lets merchants push live order status, shipping info, and pickup barcodes into the Wallet app after an Apple Pay transaction — with end-to-end encrypted iCloud sync.

---
_Source: WWDC22 Session 10041 page (abstract, chapter summaries, code samples, and resource links)._
