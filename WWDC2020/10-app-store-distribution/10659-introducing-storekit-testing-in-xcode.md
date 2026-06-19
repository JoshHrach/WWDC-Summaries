# Introducing StoreKit Testing in Xcode
**WWDC20 · Session 10659** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10659/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
Xcode 12 introduces a fully local StoreKit testing environment that lets developers build and validate in-app purchase (IAP) flows without connecting to App Store servers or setting up anything in App Store Connect first. A new StoreKit configuration file format defines products directly in the Xcode project, and a built-in Transaction Manager provides manual control over purchases, refunds, and state resets during development.

Complementing the interactive local environment is the new `StoreKitTest` framework, which exposes all testing controls programmatically. This enables XCTest-based unit and UI tests to cover purchase flows, subscription renewals, deferred/interrupted purchases, and more—fully automated without any user interaction. Sandbox testing remains the required step before App Store submission and has also received significant iOS 14 improvements, including an in-device Manage Subscriptions page for testers and an introductory offer eligibility reset.

## Key Topics

### StoreKit Configuration File
- New file type added to Xcode 12 for defining in-app products locally
- Supports consumables, non-consumables, and auto-renewable subscriptions
- Fields include product ID, price (any decimal—not limited to price tiers), localized name/description, and Family Sharing flag
- Subscription groups can be modeled; introductory offers can be attached to subscription products
- Active configuration selected in the scheme editor under Run Options

### Interactive Local Test Environment
- Replaces the Sandbox connection for local development; no network required
- Payment sheet appears in Simulator/device without requiring Apple ID sign-in or authentication
- Device receipt generated and available for client-side validation (uses a different signing key than Sandbox/production)

### Transaction Manager
- Built into Xcode, shows all transactions made in the local environment
- Allows deletion of transactions (resets purchase state) and simulation of refunds
- Refunded transactions remain in the receipt with a `cancellationDate`
- App's `SKPaymentTransactionObserver` is notified immediately on delete/refund

### Simulated Purchase Scenarios
- **Ask to Buy**: Enable via Editor menu; triggers parental approval flow; transactions enter `.deferred` state; approve/decline from Transaction Manager
- **Interrupted Purchases**: Simulates account issues (e.g., billing update required); initial purchase fails; a new transaction is queued once the issue is "resolved"
- **Subscription Time Rate**: Speed up time so subscriptions renew in seconds (e.g., 1 second = 1 day) for testing renewal behavior

### StoreKitTest Framework (New)
- `SKTestSession` — the core class; initialized with a StoreKit configuration file name
- `SKTestSession.disableDialogs` — suppresses all payment sheets/dialogs for headless test runs
- `SKTestSession.clearTransactions()` — resets all transaction state before each test
- `SKTestSession.buyProduct(identifier:)` — programmatically triggers a purchase
- `SKTestSession.expireSubscription(identifier:)` — forces immediate subscription renewal/expiry for testing
- Works with `XCTest` for unit and UI test coverage

### Receipt Validation in Local Environment
- Local receipts signed with a StoreKit Test Certificate (not the Apple Inc. Root Certificate)
- Export the StoreKit Test Certificate from the configuration file's Editor menu
- Client-side validation code should use `#if DEBUG` to switch between the test certificate and the Apple root certificate
- When using OpenSSL: pass `PKCS7_NOVERIFY` flag only in debug builds

### Sandbox Improvements (iOS 14)
- New Manage Subscriptions page accessible from iOS Settings > App Store > Sandbox Account
- Test upgrades, downgrades, and cancellations from the device without needing a Mac
- **Reset Introductory Offer Eligibility** — reuse the same Sandbox Apple ID to retest free trials
- Interrupted purchases: Enable per Sandbox Apple ID in App Store Connect (coming soon at time of WWDC20)
- Developers with the Developer role in App Store Connect can create and manage Sandbox tester accounts

## APIs & Frameworks

- **StoreKit**
  - `SKProductsRequest` — fetches product metadata (unchanged)
  - `SKProduct` — carries price, `isFamilyShareable` **[NEW]**, localized name/description
  - `SKProduct.isFamilyShareable` **[NEW]** — Boolean indicating if the product supports Family Sharing
  - `SKPayment` — represents a payment request
  - `SKPaymentQueue` — manages the transaction queue
  - `SKPaymentTransactionObserver` — protocol for transaction state updates
  - `SKPaymentTransactionObserver.paymentQueue(_:didRevokeEntitlementsForProductIdentifiers:)` **[NEW]** — called when entitlements are revoked (e.g., via refund or Family Sharing removal)
  - `SKPaymentTransaction.transactionState` — `.purchasing`, `.purchased`, `.deferred`, `.failed`, `.restored`
- **StoreKitTest** **[NEW]**
  - `SKTestSession` **[NEW]** — manages the local StoreKit test environment; initialized with a configuration file
  - `SKTestSession.disableDialogs` **[NEW]** — property to suppress payment UI for automated tests
  - `SKTestSession.clearTransactions()` **[NEW]** — resets all transactions to a clean state
  - `SKTestSession.buyProduct(identifier:)` **[NEW]** — programmatically initiates a purchase
  - `SKTestSession.expireSubscription(productIdentifier:)` **[NEW]** — forces subscription expiry/renewal
  - `SKTestSession.timeRate` **[NEW]** — controls subscription renewal speed (e.g., `.oneSecondIsOneDay`)
- **Xcode 12 tooling**
  - StoreKit Configuration File (`.storekit`) — new Xcode file type **[NEW]**
  - StoreKit Transaction Manager — Xcode debug tool **[NEW]**
  - StoreKit Test Certificate export — via Editor menu of configuration file **[NEW]**

## Code Highlights

Creating an `SKTestSession` in a unit test:
```swift
func testBuyRecipe() throws {
    let session = try SKTestSession(configurationFileNamed: "NonConsumables")
    session.disableDialogs = true
    session.clearTransactions()
    // Purchase the product and assert the app unlocks it
    try session.buyProduct(productIdentifier: "com.example.fruta.recipe.berryblue")
    XCTAssertTrue(store.unlockedRecipes.contains("berryblue"))
}
```

Switching receipt validation certificate in debug builds:
```swift
#if DEBUG
let certificate = StoreKitTestCertificate
#else
let certificate = AppleRootCertificate
#endif
```

## Takeaways
- StoreKit Testing in Xcode 12 lets you build and iterate on IAP flows entirely locally, before any App Store Connect setup — dramatically shortening the early development feedback loop.
- The `StoreKitTest` framework enables fully automated, deterministic IAP testing in XCTest, including subscription renewals, deferred/interrupted purchases, and refunds.
- Receipt signing in the local environment uses a different certificate; use `#if DEBUG` to branch between the StoreKit Test Certificate and the Apple root certificate.
- Sandbox testing remains mandatory before App Store submission; iOS 14 adds an in-device Manage Subscriptions UI and introductory offer eligibility reset that make Sandbox testing significantly less painful.

---
_Source: WWDC20 Session 10659 page (abstract, transcript, and resource links)._
