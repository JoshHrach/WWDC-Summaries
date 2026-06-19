# Meet StoreKit 2
**WWDC21 · Session 10114** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10114/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
StoreKit 2 is a complete rewrite of the StoreKit in-app purchase APIs using Swift-first design: async/await concurrency, value types, and native JSON Web Signature (JWS) transaction objects. It covers five major areas: products, purchases, transaction verification, transaction history, and subscription status. StoreKit 2 APIs live inside the existing StoreKit framework and complement (rather than replace) the original receipt-based APIs.

Key advances over original StoreKit: automatic JWS transaction verification, transaction history available on first app launch with no restore required, per-transaction signed JSON objects, and richer subscription status including renewal info and expiration reason — all with async/await replacing delegate callbacks.

## Key Topics

**Products**
`Product.products(for:)` is a single async call replacing `SKProductsRequest`. Returned `Product` values include `productType` (consumable, nonConsumable, autoRenewable, nonRenewing), extended subscription info (subscription period, trial eligibility, offer details), and a `BackingValue` subscript for forward-compatible access to future fields.

**Purchases**
`product.purchase(options:)` is an instance async method that returns a `Product.PurchaseResult` inline. Purchase options (`Product.PurchaseOption`) are composable: `quantity`, `promotionalOffer`, and the new `appAccountToken` (a UUID linking the purchase to an app-managed account, persisted in the transaction forever).

**Transaction Verification (Automatic)**
Every transaction returned by StoreKit 2 is a `VerificationResult<Transaction>`. StoreKit verifies the JWS signature automatically — the result is either `.verified(Transaction)` or `.unverified(Transaction, VerificationResult.VerificationError)`. Apps choose how to respond; calling `checkVerified()` is the common pattern.

**JWS Format**
Transactions and renewal info are signed as JSON Web Signature objects (ECDSA / x5c header — full certificate chain embedded, no network required for validation). Payload fields include `productID`, `transactionID`, `purchaseDate`, `expirationDate`, `appAccountToken`, `revocationDate`, `isUpgraded`, `deviceVerificationNonce`, `deviceVerificationID`.

**Transaction History**
- `Transaction.currentEntitlements` — finite `AsyncSequence` of currently active non-consumable and subscription transactions; revoked transactions excluded.
- `Transaction.latest(for:)` — most recent transaction for a given product ID.
- `Transaction.all` — all historical transactions.
- `AppStore.sync()` — explicit resync (replaces `restoreCompletedTransactions`); requires user authentication; only expose in response to user action.
- `Transaction.updates` — infinite `AsyncSequence` of incoming transaction updates (pending approvals, cross-device purchases); must be started at app launch.

**Subscription Status**
`product.subscription?.status` returns `[Product.SubscriptionInfo.Status]` (array because users can have multiple active statuses, e.g., personal + Family Sharing). Each `Status` has:
- `state: Product.SubscriptionInfo.RenewalState` — `.subscribed`, `.expired`, `.inGracePeriod`, `.inBillingRetryPeriod`, `.revoked`
- `transaction: VerificationResult<Transaction>` — most recent transaction
- `renewalInfo: VerificationResult<Product.SubscriptionInfo.RenewalInfo>` — auto-renew status, next product ID, expiration reason, offer details (signed JWS)

## APIs & Frameworks

### StoreKit 2 — Product **[NEW]**
- `Product.products(for: Set<String>) async throws -> [Product]` — fetch product metadata **[NEW]**
- `Product.productType` — `.consumable`, `.nonConsumable`, `.autoRenewable`, `.nonRenewing` **[NEW]**
- `Product.subscription: Product.SubscriptionInfo?` — extended subscription metadata **[NEW]**
- `Product.SubscriptionInfo.isEligibleForIntroOffer async -> Bool` **[NEW]**
- `Product.BackingValue` subscript — forward-compatible raw value access **[NEW]**

### StoreKit 2 — Purchase **[NEW]**
- `product.purchase(options: Set<Product.PurchaseOption> = []) async throws -> Product.PurchaseResult` **[NEW]**
- `Product.PurchaseResult` — `.success(VerificationResult<Transaction>)`, `.pending`, `.userCancelled` **[NEW]**
- `Product.PurchaseOption.appAccountToken(UUID)` — link purchase to app account **[NEW]**
- `Product.PurchaseOption.quantity(Int)` — consumable quantity **[NEW]**
- `Product.PurchaseOption.promotionalOffer(offerID:keyID:nonce:signature:timestamp:)` **[NEW]**

### StoreKit 2 — Transaction **[NEW]**
- `VerificationResult<Transaction>` — `.verified(Transaction)` / `.unverified(Transaction, error)` **[NEW]**
- `Transaction.updates: AsyncStream<VerificationResult<Transaction>>` — infinite async update stream **[NEW]**
- `Transaction.currentEntitlements: AsyncStream<VerificationResult<Transaction>>` — finite active entitlements **[NEW]**
- `Transaction.latest(for: String) async -> VerificationResult<Transaction>?` **[NEW]**
- `Transaction.all: AsyncStream<VerificationResult<Transaction>>` — full history **[NEW]**
- `Transaction.finish() async` — mark transaction complete **[NEW]**
- `Transaction.appAccountToken: UUID?` — persisted app account link **[NEW]**
- `Transaction.revocationDate: Date?` / `Transaction.revocationReason` **[NEW]**
- `Transaction.isUpgraded: Bool` — true when superseded by a higher-tier subscription **[NEW]**

### StoreKit 2 — Subscription Status **[NEW]**
- `Product.SubscriptionInfo.status async throws -> [Product.SubscriptionInfo.Status]` **[NEW]**
- `Product.SubscriptionInfo.RenewalState` — `.subscribed`, `.expired`, `.inGracePeriod`, `.inBillingRetryPeriod`, `.revoked` **[NEW]**
- `Product.SubscriptionInfo.RenewalInfo` — `willAutoRenew`, `autoRenewPreference`, `expirationReason`, `offerID`, `offerType` (JWS signed) **[NEW]**

### StoreKit 2 — App Store Sync **[NEW]**
- `AppStore.sync() async throws` — explicit transaction resync; requires user authentication **[NEW]**
- `AppStore.deviceVerificationID: UUID` — for manual JWS device verification **[NEW]**

## Code Highlights

Request products and purchase:
```swift
let products = try await Product.products(for: productIDs)
let result = try await products.first!.purchase(options: [
    .appAccountToken(myAccountUUID)
])
switch result {
case .success(let verification):
    let transaction = try checkVerified(verification)
    await deliverContent(for: transaction)
    await transaction.finish()
case .pending:
    break // await Transaction.updates
case .userCancelled:
    break
}
```

Listen for transaction updates at app launch:
```swift
let updateTask = Task.detached {
    for await result in Transaction.updates {
        guard let transaction = try? checkVerified(result) else { continue }
        await deliverContent(for: transaction)
        await transaction.finish()
    }
}
```

Check current entitlements on launch:
```swift
for await result in Transaction.currentEntitlements {
    guard let transaction = try? checkVerified(result) else { continue }
    switch transaction.productType {
    case .nonConsumable: unlock(transaction.productID)
    case .autoRenewable: unlockSubscription(transaction.productID)
    default: break
    }
}
```

Read subscription status:
```swift
let statuses = try await subscriptionProduct.subscription?.status ?? []
for status in statuses {
    guard status.state != .expired, status.state != .revoked else { continue }
    let renewalInfo = try checkVerified(status.renewalInfo)
    // Use renewalInfo.willAutoRenew, renewalInfo.expirationReason, etc.
}
```

## Takeaways
- StoreKit 2 replaces delegate-based callbacks with async/await throughout — requesting products, purchasing, and reading transaction history all fit in sequential code with no delegates or notifications.
- Automatic JWS verification via `VerificationResult` means apps no longer parse raw receipts; manual validation remains possible using CryptoKit + the embedded x5c certificate chain.
- `Transaction.currentEntitlements` is a finite async sequence that delivers all active entitlements on first launch — making manual "restore purchases" flows largely unnecessary.
- `Product.SubscriptionInfo.Status` returns an array to handle personal + Family Sharing subscriptions; always take the highest level of service from the array.

---
_Source: WWDC21 Session 10114 page (abstract, chapter summaries, code samples, and resource links)._
