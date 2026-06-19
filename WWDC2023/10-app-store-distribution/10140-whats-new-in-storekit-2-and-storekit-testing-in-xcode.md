# What's new in StoreKit 2 and StoreKit Testing in Xcode
**WWDC23 · Session 10140** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10140/)

_Platforms:_ iOS 16.4+, iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10

## Overview
StoreKit 2 receives a broad set of enhancements in 2023, spanning promoted in-app purchases, richer Transaction and RenewalInfo data models, a new billing issue message type, SHA-256 receipt signing, and a full suite of SwiftUI merchandising views. These improvements make it significantly easier to build, merchandise, and test in-app purchases without boilerplate.

StoreKit Testing in Xcode 15 adds a Transaction Inspector that works outside active debug sessions, supports multiple simultaneous devices, allows off-device purchase creation with custom parameters, and introduces configurable error simulation for exhaustive coverage of failure scenarios.

## Key Topics

**Promoted In-App Purchases (iOS 16.4+)**
- New Swift async sequence `PurchaseIntent.intents` replaces the old delegate-based promoted purchase listener
- New APIs to read and write the local promotion order and visibility per device (`Product.PromotionInfo.currentOrder`, `updateProductOrder`, `updateProductVisibility`)
- Visibility states: `.visible`, `.hidden`, `.default` (follows App Store Connect settings)

**Transaction and RenewalInfo Model Updates**
- `Transaction` gains `storefront` and `storefrontCountryCode` **[NEW]** and `reason` **[NEW]** (`.purchase` vs `.renewal`)
- `Product.SubscriptionInfo.RenewalInfo` gains `nextRenewalDate` **[NEW]**
- All new fields work retroactively with prior iOS versions when using StoreKit 2

**StoreKit Messages — billingIssue**
- New `Message.Reason.billingIssue` **[NEW]** (iOS 16.4+) — App Store sends this when a subscription fails to renew due to a billing problem; displays an in-app Billing Problem sheet

**Receipt Security: SHA-256 Migration**
- Receipt signing certificate migrating from SHA-1 to SHA-256 for original StoreKit receipts
- Sandbox transition: June 20, 2023 (iOS 16.6 / macOS 13.5+); production for new/updated apps: August 14, 2023
- StoreKit 2 (Signed Transaction, RenewalInfo, App Transaction) and App Store Server APIs already use SHA-256 — no changes needed

**SwiftUI StoreKit Views (NEW)**
- `ProductView` — single in-app purchase with title, description, optional promotional image
- `StoreView` — collection of products as a store listing
- `SubscriptionStoreView` — subscription group merchandising page, customizable
- `.manageSubscriptionsSheet(groupID:)` — jump directly to a specific subscription group's management sheet **[NEW parameter]**

**StoreKit Testing in Xcode 15**
- Transaction Inspector now shows all StoreKit apps on all connected devices/simulators, even outside debug sessions
- Create off-device purchases directly from Xcode with configurable parameters (purchase date, quantity, offer codes, renewal behavior)
- New Configuration Settings section in StoreKit configuration file (replaces Editor menu options)
- New subscription renewal rates (per-renewal expiration for faster testing, iOS 16.4+)
- Simulated error configuration: choose an error type per StoreKit 2 API (product loading, purchase, verification, refund, etc.)

## APIs & Frameworks

**StoreKit**
- `PurchaseIntent.intents` **[NEW]** — async sequence for promoted purchase intents (iOS 16.4+)
- `PurchaseIntent` **[NEW]** — struct with `.product: Product`
- `Product.PromotionInfo` **[NEW]**
  - `currentOrder` async — returns `[PromotionInfo]` with current local order
  - `updateProductOrder(byID:)` async throws **[NEW]**
  - `updateProductVisibility(_:for:)` async throws **[NEW]**
  - `PromotionInfo.visibility` — `.visible`, `.hidden`, `.default`
  - `PromotionInfo.update()` async throws **[NEW]** — save visibility change on instance
- `Transaction.storefront` **[NEW]** (`Storefront`)
- `Transaction.storefrontCountryCode` **[NEW]** (`String`)
- `Transaction.reason` **[NEW]** — `.purchase` or `.renewal`
- `Product.SubscriptionInfo.RenewalInfo.nextRenewalDate` **[NEW]** (`Date?`)
- `Message.Reason.billingIssue` **[NEW]** (iOS 16.4+)
- `ProductView(id:)` **[NEW]** SwiftUI view — single product
- `StoreView(ids:)` **[NEW]** SwiftUI view — product collection
- `SubscriptionStoreView(groupID:)` **[NEW]** SwiftUI view — subscription group
- `.productViewStyle(.large)` **[NEW]** modifier
- `.manageSubscriptionsSheet(groupID:)` **[NEW]** — new `groupID` parameter

**StoreKitTest**
- `SKTestSession.buyProduct(identifier:options:)` async throws **[NEW]** — off-device purchase
- `SKTestSession.PurchaseOption.purchaseDate(_:)` **[NEW]** — test-only purchase option
- `SKTestSession.setSimulatedError(_:forAPI:)` **[NEW]** — simulate errors per API
- `SKTestSession.timeRate` — updated with new faster renewal rates (iOS 16.4+)
  - `.oneRenewalEveryMinute` (new rate example)

## Code Highlights

Listening for promoted purchase intents:
```swift
let promotedPurchasesListener = Task {
    for await promotion in PurchaseIntent.intents {
        let result = try await promotion.product.purchase()
        // Process result
    }
}
```

Setting promotion order:
```swift
try await Product.PromotionInfo.updateProductOrder(byID: ["acorns.individual", "nectar.cup"])
```

SubscriptionStoreView with custom background:
```swift
SubscriptionStoreView(groupID: groupID)
    .backgroundStyle(.thinMaterial)
```

Simulating a network error in tests:
```swift
session.setSimulatedError(.generic(.networkError), forAPI: .loadProducts)
// Any call to Product.products(for:) now throws networkError
session.setSimulatedError(nil, forAPI: .loadProducts) // disable
```

Off-device purchase with back-dated date:
```swift
let transaction = try await session.buyProduct(
    identifier: "birdpass.individual",
    options: [.purchaseDate(Date.now - 365 * 24 * 60 * 60)]
)
```

## Takeaways
- Adopt `PurchaseIntent.intents` to handle promoted in-app purchases with modern Swift concurrency instead of the old delegate pattern.
- Use `ProductView`, `StoreView`, and `SubscriptionStoreView` to ship in-app purchase merchandising UI with minimal code.
- If doing on-device receipt validation with original StoreKit, verify SHA-256 support in your crypto library before the June/August 2023 deadlines.
- Use the new simulated error APIs in `SKTestSession` to systematically cover all StoreKit failure paths in unit tests.

---
_Source: WWDC23 Session 10140 page (abstract, chapter summaries, code samples, and resource links)._
