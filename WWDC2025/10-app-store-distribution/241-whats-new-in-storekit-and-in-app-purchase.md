# What's new in StoreKit and In-App Purchase
**WWDC25 · Session 241** · [Watch](https://developer.apple.com/videos/play/wwdc2025/241/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26, watchOS 26

## Overview
This session covers four areas of StoreKit improvement in 2025: new fields on core types (`AppTransaction`, `Transaction`, `RenewalInfo`), expanded offer code support for non-subscription product types, JWS-signed purchase requests via the App Store Server Library, and a new `SubscriptionOfferView` for merchandising subscriptions.

Many new fields (especially `appTransactionID` and `originalPlatform`) are back-deployed to iOS 15 and aligned OS versions, making them immediately usable without raising deployment targets. The emphasis throughout is on giving developers richer transactional data to support evolving business models and more nuanced subscription management.

## Key Topics

### New fields on AppTransaction, Transaction, RenewalInfo
**AppTransaction** gains `appTransactionID` (globally unique per Apple Account per app, unique per Family Sharing member, back-deployed to iOS 15) and `originalPlatform` (an `AppStore.Platform` enum: iOS, macOS, tvOS, visionOS — apps downloaded on watchOS report `.iOS`). Useful for entitlement logic across business-model changes (paid → freemium).

**Transaction** gains `appTransactionID`, `offer.offerPeriod` (the subscription period associated with a redeemed offer), and `advancedCommerceInfo` (nil unless using the Advanced Commerce API). The deprecated `Transaction.currentEntitlement(for: productID)` is replaced by **`Transaction.currentEntitlements(for: productID)`** which returns an async sequence for cases where multiple entitlements apply (e.g., direct subscription + Family Sharing).

**RenewalInfo** gains `appTransactionID`, `offer.offerPeriod`, `advancedCommerceInfo`, and `appAccountToken` (mirrors the value set via `appAccountToken` purchase option). **`SubscriptionStatus(transactionID:)`** is new: query subscription status directly from any Transaction ID.

### Offer codes for non-subscription products
Offer codes are now available for consumables, non-consumable, and non-renewing subscriptions (previously auto-renewable subscriptions only). Redemption via `offerCodeRedemption` API (SwiftUI) or `presentOfferCodeRedeemSheet` (UIKit) back to iOS 16.3. A new **`Transaction.Offer.PaymentMode.oneTime`** case (available since iOS 17.2; use `offerPaymentModeStringRepresentation` for iOS 15+).

### JWS-signed purchase requests and App Store Server Library
Two new purchase options require a compact JWS string:
- **`introductoryOfferEligibility`** — set customer eligibility server-side
- **`promotionalOffer`** (new JWS-based version) — replaces the older `promotionalOffer` purchase option

The App Store Server Library (Swift/Java/Python/Node.js) creates these signatures via `PromotionOfferV2SignatureCreator.createSignature(productID:offerID:transactionID:)`. New SwiftUI view modifiers `subscriptionPromotionalOffer` and `preferredSubscriptionOffer` accompany these options.

### SubscriptionOfferView
A new SwiftUI view for merchandising a single auto-renewable subscription plan. Can be initialized by product or product ID (auto-loads metadata). Decoratable with `prefersPromotionalIcon: true` or a custom icon `ViewBuilder`. The **`subscriptionOfferViewDetailAction`** modifier adds a detail link button that fires a closure (e.g., to navigate to the full subscription store). The **`visibleRelationship`** parameter controls which plan is shown relative to the customer's current subscription: `.upgrade`, `.downgrade`, `.crossgrade`, `.current`, `.all`. Use `subscriptionStatusTask` (iOS 17+) to determine the correct relationship to pass.

## APIs & Frameworks

### StoreKit
- `AppTransaction.appTransactionID` **[NEW]** — back-deployed iOS 15
- `AppTransaction.originalPlatform` **[NEW]** — `AppStore.Platform` enum (iOS 18.4)
- `AppStore.Platform` **[NEW]** — `.iOS`, `.macOS`, `.tvOS`, `.visionOS`
- **`Transaction.currentEntitlements(for: productID)`** **[NEW]** — replaces deprecated single-entitlement API (iOS 18.4)
- `Transaction.currentEntitlement(for: productID)` — **deprecated** (iOS 18.4)
- `Transaction.appTransactionID` **[NEW]** — back-deployed iOS 15
- `Transaction.offer.offerPeriod` **[NEW]** — iOS 18.4
- `Transaction.advancedCommerceInfo` **[NEW]** — iOS 18.4
- `Transaction.Offer.PaymentMode.oneTime` **[NEW]** — iOS 17.2 (back-deployed string representation: iOS 15)
- `RenewalInfo.appTransactionID` **[NEW]** — back-deployed iOS 15
- `RenewalInfo.offer.offerPeriod` **[NEW]** — iOS 18.4
- `RenewalInfo.advancedCommerceInfo` **[NEW]** — iOS 18.4
- `RenewalInfo.appAccountToken` **[NEW]** — iOS 18.4
- **`SubscriptionStatus(transactionID:)`** **[NEW]** — iOS 18.4
- **`introductoryOfferEligibility`** purchase option **[NEW]** — JWS-based; back-deployed iOS 15
- **`promotionalOffer`** purchase option (JWS version) **[NEW]** — back-deployed iOS 15
- `offerCodeRedemption` (SwiftUI, existing) — now supports consumables, non-consumables, non-renewing subscriptions
- `presentOfferCodeRedeemSheet` (UIKit, existing) — same expansion
- **`SubscriptionOfferView`** **[NEW]** — SwiftUI view
- `SubscriptionOfferView(product:prefersPromotionalIcon:)` **[NEW]**
- `SubscriptionOfferView(id:prefersPromotionalIcon:)` **[NEW]**
- **`.subscriptionOfferViewDetailAction(_:)`** modifier **[NEW]**
- **`.subscriptionPromotionalOffer(_:signature:)`** modifier **[NEW]**
- **`.preferredSubscriptionOffer(_:)`** modifier **[NEW]**
- `SubscriptionOfferView.VisibleRelationship` **[NEW]** — `.upgrade`, `.downgrade`, `.crossgrade`, `.current`, `.all`
- `subscriptionStatusTask` modifier — iOS 17 (existing)
- `AdvancedCommerceProduct` **[NEW]** — iOS 18.4
- `PurchaseAction` / `@Environment(\.purchase)` (existing, for context-aware purchases from SwiftUI)

### App Store Server Library
- `PromotionOfferV2SignatureCreator` **[NEW]** — Swift (and Java/Python/Node.js)
- `PromotionOfferV2SignatureCreator.createSignature(productID:offerID:transactionID:)` **[NEW]**

## Code Highlights

```swift
// New currentEntitlements sequence
for await entitlement in Transaction.currentEntitlements(for: "com.example.pro") {
    // may include direct + family sharing entitlements
}

// SubscriptionOfferView with detail link
SubscriptionOfferView(groupID: subscriptionGroupID,
                      visibleRelationship: customerStatus.isSubscribed ? .upgrade : .all) {
    Image("subscription-icon")
}
.subscriptionOfferViewDetailAction {
    showSubscriptionStore = true
}

// Conditional upgrade recommendation
.subscriptionPromotionalOffer({ subscription in
    subscription.bestPromoOffer
}, signature: { productID, offerID in
    await networkLayer.fetchPromoSignature(productID: productID, offerID: offerID)
})
```

## Takeaways
- Adopt `Transaction.currentEntitlements(for:)` (replaces deprecated `currentEntitlement`) to correctly handle Family Sharing overlapping entitlements.
- Use `appTransactionID` as a stable per-user-per-app identifier that survives app reinstalls and doesn't require a server round-trip.
- Integrate the App Store Server Library for JWS signing of promotional and introductory offers — it's open source and available in four languages.
- Deploy `SubscriptionOfferView` with `subscriptionOfferViewDetailAction` as a lightweight upsell card that can route to your full subscription store.

---
_Source: WWDC25 Session 241 page (abstract, chapter summaries, code samples, and resource links)._
