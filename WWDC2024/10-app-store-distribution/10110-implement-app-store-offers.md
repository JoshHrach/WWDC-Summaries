# Implement App Store Offers
**WWDC24 · Session 10110** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10110/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2

## Overview
This session covers all updates to App Store subscription offers for 2024, with a major focus on the new **win-back offers** feature. Win-back offers allow developers to re-engage former subscribers through App Store-managed eligibility criteria, App Store promotion, and a streamlined purchasing flow — without requiring custom server-side eligibility logic.

The session is split between App Store Connect configuration (from the backend engineer perspective) and StoreKit 2 implementation (from the client app perspective). It also covers several smaller but important API additions: a new `offer` member in `Transaction` and `RenewalInfo`, offer codes on macOS, and the `preferredSubscriptionOffer` view modifier for `SubscriptionStoreView`.

## Key Topics

**Updates to Existing Offers**
- New `Transaction.offer` property — `SubscriptionOffer?` containing `id`, `type`, `paymentMode` (freeTrial, payAsYouGo, payUpFront); available from iOS 17.2
- `RenewalInfo.offer` — indicates the offer applied to the subscription's next renewal period; available from iOS 18.0
- App Store server API: new `offerDiscountType` field in `JWSTransaction` and `JWSRenewalInfo` (available for historical transactions)
- `subscriptionPromotionalOffer` view modifier for `SubscriptionStoreView` — display a promotional offer and generate its signature inline
- Offer code redemption sheet now supported on **macOS 15** via `.offerCodeRedemption(isPresented:)` SwiftUI modifier

**Win-Back Offers**
- New offer type targeting churned subscribers (subscription expired, auto-renew disabled)
- Eligibility criteria managed entirely by App Store Connect: paid subscription duration, time since last subscribed, wait between offers
- Configured per country/region with localized pricing; offer priority controls App Store promotion ordering
- Promoted automatically in App Store (Today, Games, Apps tabs; product page; customer's Subscriptions page) when a subscription image is approved
- App Store Connect API 3.6 adds full CRUD endpoints: `WinBackOffers` resource

**Supporting Win-Back Offers in Your App**
- **StoreKit Message API** (easiest): `Message.Reason.winBackOffer` — StoreKit automatically presents the offer sheet when eligible; listen for transaction updates
- **StoreKit Views** (no-code option): `SubscriptionStoreView` automatically surfaces win-back offers; use `preferredSubscriptionOffer` modifier when custom selection logic is needed
- **Core StoreKit 2**: use `RenewalInfo.eligibleWinBackOfferIDs` to get personalized list of eligible offer IDs; find corresponding `Product.SubscriptionOffer` in `subscription.winBackOffers`; purchase with `.winBackOffer(offer)` option
- Test in Sandbox by enabling "Display Win-back Offers" toggle in iOS sandbox account settings
- Test in Xcode using StoreKit Testing — configure win-back offers and toggle eligibility per offer

**Streamlined Purchasing**
- Enabled by default for win-back offers promoted on App Store — the App Store handles the payment sheet before the app opens
- Opt out in App Store Connect (Billing Grace Period settings) to require purchase to flow through your app
- When opted out, receive a `PurchaseIntent` via `PurchaseIntent.intents` with the new `PurchaseIntent.offer` property containing the win-back `SubscriptionOffer`

## APIs & Frameworks

**StoreKit**
- `Transaction.offer` **[NEW]** — `SubscriptionOffer?` (`iOS 17.2+`)
- `RenewalInfo.offer` **[NEW]** — offer applied to next renewal period (`iOS 18.0+`)
- `Product.SubscriptionInfo.OfferType.winBack` **[NEW]** — new offer type case (`iOS 18.0+`)
- `Product.SubscriptionInfo.winBackOffers` **[NEW]** — array of available win-back offers on subscription info
- `Product.SubscriptionInfo.Status.RenewalInfo.eligibleWinBackOfferIDs` **[NEW]** — personalized list of redeemable offer IDs
- `Product.PurchaseOption.winBackOffer(_:)` **[NEW]** — add win-back offer to a purchase (`iOS 18.0+`)
- `Message.Reason.winBackOffer` **[NEW]** — StoreKit Message reason for displaying a win-back offer
- `preferredSubscriptionOffer` **[NEW]** — `SubscriptionStoreView` modifier; closure receives `(Product, SubscriptionInfo, [SubscriptionOffer])` and returns `SubscriptionOffer?`
- `subscriptionPromotionalOffer` **[NEW]** — view modifier to display a promotional offer in `SubscriptionStoreView`
- `SubscriptionStoreView` — automatically shows win-back offer pricing when eligible
- `PurchaseIntent` — carries purchase information initiated outside the app (e.g., App Store promotion)
- `PurchaseIntent.offer` **[NEW]** — `SubscriptionOffer?` for win-back offers from App Store promotion (`iOS 18.0`, `macOS 15.0`)
- `.offerCodeRedemption(isPresented:)` **[NEW on macOS]** — present offer code sheet on macOS 15
- `Product.SubscriptionInfo.isEligibleForIntroOffer(for:)` — check introductory offer eligibility

**App Store Connect API 3.6**
- `WinBackOffers` resource — create, read, update, delete win-back offers
- `inAppPurchaseImages` / `subscriptionImages` resources **[NEW]** — manage promotional images (replaces deprecated `PromotedPurchaseImages`)

**App Store Connect (UI)**
- Win-Back Offers tab on subscription pricing page
- Offer priority configuration; country/region selection; image upload for App Store promotion
- Streamlined Purchasing toggle (in Billing Grace Period settings)
- Eligibility criteria: paid subscription duration (months), time since last subscribed (min/max months), wait between offers (months)

## Code Highlights

Choose the best offer in a `SubscriptionStoreView`:
```swift
SubscriptionStoreView(groupID: groupID)
    .preferredSubscriptionOffer { product, subscription, eligibleOffers in
        let freeTrialOffer = eligibleOffers
            .filter { $0.paymentMode == .freeTrial }
            .max { $0.period.value < $1.period.value }
        return freeTrialOffer ?? eligibleOffers.first
    }
```

Purchase with a win-back offer:
```swift
var purchaseOptions: Set<Product.PurchaseOption> = []
if let offer, offer.type == .winBack {
    purchaseOptions.insert(.winBackOffer(offer))
}
try await product.purchase(options: purchaseOptions)
```

## Takeaways
- Adopt `Message.Reason.winBackOffer` for the fastest path to surfacing win-back offers — StoreKit handles presentation automatically.
- Use `RenewalInfo.eligibleWinBackOfferIDs` alongside existing subscription status checks for custom merchandising UI.
- Configure win-back offers in App Store Connect before writing any app code — eligibility, pricing, and promotion are all managed there.
- Enable Streamlined Purchasing (on by default) unless your app requires custom onboarding before purchase completes.

---
_Source: WWDC24 Session 10110 page (abstract, chapter summaries, code samples, and resource links)._
