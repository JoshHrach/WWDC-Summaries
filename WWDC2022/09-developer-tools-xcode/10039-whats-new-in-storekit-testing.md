# What's New in StoreKit Testing
**WWDC22 · Session 10039** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10039/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
Xcode 14 and the App Store sandbox environment receive a significant set of improvements for testing in-app purchases and subscriptions. The session is organized in two halves: Xcode-side testing enhancements (synced StoreKit configuration files from App Store Connect, SwiftUI Previews support, an improved Transaction Manager) and advanced subscription test scenarios (refund requests, offer codes, price increase consent, billing retry and grace period). The second half covers new sandbox testing capabilities: streamlined Apple ID creation, App Store Connect API additions for sandbox management, and billing failure simulation in sandbox.

## Key Topics

### Synced StoreKit Configuration Files (Xcode 14) [NEW]
A new "Synced" StoreKit Configuration file type pulls product and subscription metadata directly from App Store Connect. This means developers configure IAP products once in App Store Connect and use the same data locally in Xcode, in unit tests, in sandbox, and on the App Store. Workflow:
1. File > New > StoreKit Configuration File → check "Keep this file in sync with an app in App Store Connect"
2. Select team and app; file syncs product data automatically
3. Press the sync button to pull future App Store Connect changes
4. Convert to local (editable) file via Editor menu — one-way, irreversible operation

### SwiftUI Previews with StoreKit (Xcode 14) [NEW]
Products from StoreKit configuration files (both local and synced) now load in SwiftUI Previews, enabling live UI testing of subscription store views with real IAP metadata without running the app.

### Transaction Manager Improvements (Xcode 14) [NEW]
- **Transaction Inspector** — right panel showing decoded transaction details: expiration dates, renewal info, offer types, revocation reason/date
- **Jump to configuration** — click from any transaction property to jump directly to the product in the configuration file
- **Transaction filtering** — filter by product ID, purchase date, and other attributes using an autocomplete-driven filter bar

### Refund Request Testing [NEW]
`StoreView.refundRequestSheet(for:isPresented:onDismiss:)` view modifier opens the system refund request sheet. In Xcode/sandbox, the issue selected maps directly to `Transaction.RevocationReason` and the refund is processed immediately. Available on iOS 15.2+, macOS 12.1+.

### Offer Code Testing (Xcode 13.3+)
`View.offerCodeRedeemSheet(isPresented:)` presents the offer code redemption sheet. In Xcode, the sheet shows all configured offer codes grouped by subscription, allowing selection without typing a code. Test `transaction.offerType` (`.code`) and `renewalInfo.offerID` to verify code redemption handling.

### Price Increase Consent Testing [NEW]
In Xcode 14, the Transaction Manager toolbar exposes "Request Price Increase Consent" for any active subscription transaction. A system sheet appears above the app; use `Message.messages` async sequence + `DisplayMessageAction` to control when the sheet appears. Programmatically test with `SKTestSession.requestPriceIncreaseConsentForTransaction(identifier:)`, `consentToPriceIncreaseForTransaction(identifier:)`, and `declinePriceIncreaseForTransaction(identifier:)`.

### Billing Retry and Grace Period Testing (Xcode 13.3+)
Enable via Xcode Editor menu: "Billing Retry on Renewal" + "Billing Grace Period". Use `SKTestSession.billingGracePeriodIsEnabled` and `shouldEnterBillingRetryOnRenewal` in unit tests. Resolve billing issues with `resolveIssueForTransaction(identifier:)`.

### Sandbox Improvements
- **Streamlined Apple ID creation** — fewer required fields, email plus-addressing support, inline password strength hints
- **App Store Connect API for sandbox** — new endpoints for listing sandbox Apple IDs, clearing purchase history, setting interrupted purchase state (coming later in 2022)
- **Billing failure simulation in sandbox** — new Sandbox Account Settings toggle simulates payment decline, triggering billing retry state, V2 Notifications (`DID_FAIL_TO_RENEW`, `GRACE_PERIOD_EXPIRED`, `BILLING_RECOVERY`), and receipt fields (`is_in_billing_retry_period`, grace period expiration dates)

## APIs & Frameworks

**StoreKit (Swift)**
- `View.refundRequestSheet(for:isPresented:onDismiss:)` **[NEW]** — present in-app refund request sheet
- `Transaction.revocationDate`, `Transaction.revocationReason` — detect refunded transactions from `Transaction.updates`
- `View.offerCodeRedeemSheet(isPresented:)` — offer code redemption sheet
- `Transaction.offerType` — `.introductory`, `.promotional`, `.code`
- `Product.SubscriptionInfo.RenewalInfo.offerType`, `.offerID` — pending offer type and reference name
- `Product.SubscriptionInfo.RenewalInfo.priceIncreaseStatus` — `.agreed` or pending
- `Product.SubscriptionInfo.RenewalInfo.expirationReason` — `.didNotConsentToPriceIncrease`
- `Message.messages` **[NEW]** — async sequence of `StoreKit.Message` values (e.g., `.priceIncrease`)
- `DisplayMessageAction` **[NEW]** — environment action to display a StoreKit message
- `Product.SubscriptionInfo.RenewalInfo.gracePeriodExpirationDate` **[NEW]** — billing grace period end
- `Product.SubscriptionInfo.RenewalInfo.isInBillingRetry` — billing retry state
- `Product.SubscriptionInfo.Status.state` — `.inBillingRetryPeriod`, `.inGracePeriod`
- `Transaction.currentEntitlements` — includes grace period subscriptions automatically

**StoreKitTest (XCTest)**
- `SKTestSession(configurationFileNamed:)` — test session for unit testing
- `SKTestSession.disableDialogs` — suppress UI sheets during tests
- `SKTestSession.requestPriceIncreaseConsentForTransaction(identifier:)` **[NEW]**
- `SKTestSession.consentToPriceIncreaseForTransaction(identifier:)` **[NEW]**
- `SKTestSession.declinePriceIncreaseForTransaction(identifier:)` **[NEW]**
- `SKTestTransaction.isPendingPriceIncreaseConsent` **[NEW]**
- `SKTestSession.billingGracePeriodIsEnabled` **[NEW]**
- `SKTestSession.shouldEnterBillingRetryOnRenewal` **[NEW]**
- `SKTestTransaction.hasPurchaseIssue` **[NEW]**
- `SKTestSession.resolveIssueForTransaction(identifier:)` **[NEW]**

## Code Highlights

```swift
// Detect refunded transactions
for await update in Transaction.updates {
    let transaction = try update.payloadValue
    if let revocationDate = transaction.revocationDate,
       let revocationReason = transaction.revocationReason {
        switch revocationReason {
        case .developerIssue: break // handle developer issue
        case .other: break          // handle other issue
        default: break
        }
    }
}

// Offer code redemption sheet
.offerCodeRedeemSheet(isPresented: $redeemSheetIsPresented)

// StoreKit Messages (price increase) with deferral logic
for await message in Message.messages {
    if !sensitiveViewIsPresented,
       let display: DisplayMessageAction = displayAction {
        try? display(message)
    } else {
        pendingMessages.append(message)
    }
}
```

```swift
// Unit test: billing retry and grace period
let session = try SKTestSession(configurationFileNamed: "FoodTruck")
session.billingGracePeriodIsEnabled = true
session.shouldEnterBillingRetryOnRenewal = true
// ... purchase subscription, wait for renewal ...
let tx = session.allTransactions().first!
XCTAssertTrue(tx.hasPurchaseIssue)
// ... assert grace period access still granted ...
session.resolveIssueForTransaction(identifier: tx.identifier)
// ... assert subscription access restored ...
```

## Takeaways

- Sync your StoreKit Configuration File with App Store Connect in Xcode 14 to use real product data in local tests, SwiftUI Previews, and sandbox without duplicating configuration.
- Add `refundRequestSheet` and `offerCodeRedeemSheet` to your UI and write `XCTestCase` tests using `SKTestSession` methods to systematically verify all subscription lifecycle corner cases before App Store submission.
- Use the `Message.messages` async sequence to intercept price increase messages and defer display until the UI is in an appropriate state — test deferral logic with `SKTestSession.requestPriceIncreaseConsentForTransaction`.
- Enable "Billing Retry on Renewal" in Xcode and set `shouldEnterBillingRetryOnRenewal = true` in unit tests to verify that your app correctly grants access during grace period and prompts the user to fix billing in retry state.

---
_Source: WWDC22 Session 10039 page (abstract, chapter summaries, code samples, and resource links)._
