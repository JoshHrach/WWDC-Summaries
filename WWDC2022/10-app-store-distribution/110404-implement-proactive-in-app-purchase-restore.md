# Implement Proactive In-App Purchase Restore
**WWDC22 · Session 110404** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110404/)

_Platforms:_ iOS 15+, iPadOS 15+, macOS 12+, tvOS 15+, watchOS 8+

## Overview
Proactive in-app purchase restore is the practice of automatically identifying a customer's purchase state at app launch — without requiring any action from the user such as tapping a "Restore Purchases" button or authenticating. By reading transaction data already available on device via StoreKit, an app can immediately determine whether a user is a new customer, an active subscriber, or an inactive/lapsed subscriber, and tailor the onboarding experience accordingly.

The session covers both the modern StoreKit 2 path (iOS 15+) and the original StoreKit path using the `verifyReceipt` server endpoint (iOS 7+). The recommended three-step implementation pattern — listen for transaction updates, determine current entitlements, and personalize the experience — applies to both frameworks.

## Key Topics
- **Three core customer product states** — New (no purchase history), Active (current entitlement), Inactive (expired, revoked, or billing retry)
- **Step 1: Transaction listener** — `Transaction.updates` async sequence processes unfinished or updated transactions at launch and on every subsequent launch
- **Step 2: Current entitlements** — `Transaction.currentEntitlements` iterates all active transactions for persistent in-app purchase types (non-consumables, non-renewing subscriptions, auto-renewable subscriptions)
- **Step 3: Personalize the experience** — branch UI based on product states; hide buy buttons for owned products, show win-back offers for lapsed subscribers, show billing retry call-to-action when in grace period
- **Auto-renewable subscription renewal states** — `Product.SubscriptionInfo.RenewalState` handles expired, billing retry, revoked, and cancelled states
- **Original StoreKit fallback** — retrieve `Bundle.main.appStoreReceiptURL`, send to server, call `verifyReceipt` endpoint, use Entitlement Engine to determine states
- **App Store Server Notifications v2** — 28 unique event types/subtypes for server-side transaction status maintenance
- **CloudKit for cross-device state** — securely storing and syncing customer product state across devices
- **Testing** — Sandbox, TestFlight, and Xcode StoreKit testing environments; note that app receipt may not be present in Sandbox until first purchase

## APIs & Frameworks
**StoreKit 2**
- `Transaction.updates` — async sequence of `VerificationResult<Transaction>` for unfinished/updated transactions **[NEW in StoreKit 2]**
- `Transaction.currentEntitlements` — async sequence returning all currently active transactions for a customer **[NEW in StoreKit 2]**
- `transaction.finish()` — marks a transaction as delivered; removes it from `updates` and `currentEntitlements` on all devices
- `Transaction.productType` — `.nonConsumable`, `.nonRenewable`, `.autoRenewable` discriminators
- `Transaction.productID` — string product identifier
- `Transaction.purchaseDate` — used to calculate non-renewing subscription expiry
- `Transaction.expirationDate` — expiration for auto-renewable subscriptions
- `Product.SubscriptionInfo.RenewalState` — enum with cases `.subscribed`, `.expired`, `.inBillingRetryPeriod`, `.inGracePeriod`, `.revoked`
- `product.subscription?.status.first?.state` — retrieves `RenewalState` for a subscription group
- `checkVerified(_:)` — helper to unwrap `VerificationResult` and throw on failed JWS verification

**Original StoreKit**
- `Bundle.main.appStoreReceiptURL` — URL of the on-device App Store receipt
- `FileManager.default.fileExists(atPath:)` — checks if receipt is present (may be absent in Sandbox before first purchase)
- `Data(contentsOf:options:)` with `.alwaysMapped` — reads receipt bytes
- `receiptData.base64EncodedString(options:)` — encodes receipt for server transmission
- `verifyReceipt` App Store server endpoint — validates receipt and returns latest transactions

**App Store Server**
- App Store Server Notifications Version 2 — near real-time server-to-server notifications for all transaction events
- Billing Grace Period (App Store Connect setting) — grants continued access during billing retry up to 60 days

**CloudKit**
- `CloudKit` framework — recommended for securely syncing customer product state across devices

## Code Highlights
Transaction listener at app launch (StoreKit 2):
```swift
func listenForTransactions() -> Task<Void, Error> {
    return Task.detached {
        for await result in Transaction.updates {
            do {
                let transaction = try self.checkVerified(result)
                await self.updateCustomerProductStatus()
                await transaction.finish()
            } catch {
                print("Transaction failed verification")
            }
        }
    }
}
```

Determining current entitlements and subscription renewal state:
```swift
for await result in Transaction.currentEntitlements {
    let transaction = try checkVerified(result)
    switch transaction.productType {
    case .nonConsumable:
        if let car = cars.first(where: { $0.id == transaction.productID }) {
            purchasedCars.append(car)
        }
    case .autoRenewable:
        if let sub = subscriptions.first(where: { $0.id == transaction.productID }) {
            purchasedSubscriptions.append(sub)
        }
    default: break
    }
}
subscriptionGroupStatus = try? await subscriptions.first?.subscription?.status.first?.state
```

Handling inactive subscriber states:
```swift
if subscriptionGroupStatus == .expired || subscriptionGroupStatus == .revoked {
    Text("Welcome Back! Head over to the shop to get started!")
} else if subscriptionGroupStatus == .inBillingRetryPeriod {
    Text("Please verify your billing details.")
}
```

Fetching app receipt for original StoreKit path:
```swift
if let appStoreReceiptURL = Bundle.main.appStoreReceiptURL,
   FileManager.default.fileExists(atPath: appStoreReceiptURL.path) {
    let receiptData = try Data(contentsOf: appStoreReceiptURL, options: .alwaysMapped)
    let receiptString = receiptData.base64EncodedString(options: [])
    // send receiptString to server for verifyReceipt
}
```

## Takeaways
- Proactive restore requires no user action — apps should check `Transaction.currentEntitlements` at launch before showing any buy buttons.
- The three product states (new, active, inactive) drive three distinct UI experiences: default merchandising, immediate access, and win-back offers.
- Always maintain a "Restore Purchases" button as a fallback for edge cases (wrong Apple ID, receipt issues).
- App Store Server Notifications v2 is essential for keeping server-side entitlement state accurate across all transaction lifecycle events.

---
_Source: WWDC22 Session 110404 page (abstract, chapter summaries, code samples, and resource links)._
