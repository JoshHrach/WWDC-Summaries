# What's New in Apple In-App Purchase
**WWDC26 · Session 210** · [Watch](https://developer.apple.com/videos/play/wwdc2026/210/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, visionOS, watchOS

## Overview
The headline feature this year is **monthly subscriptions with a 12-month commitment** — a new pricing option that lets customers pay for an annual subscription on a month-by-month basis, offering a more accessible entry point to premium tiers while still locking in a full-year revenue commitment. This is additive to existing upfront annual pricing: developers can offer both billing plans on the same subscription product simultaneously, and StoreKit APIs expose the additional metadata needed to merchandise the choice clearly.

The session covers the complete lifecycle of this new option: setup in App Store Connect, merchandising with `SubscriptionStoreView` and raw `Product` APIs, server-side monitoring via updated `JWSTransaction` and `JWSRenewalInfo` fields, and testing in StoreKit Testing in Xcode 27. Additional improvements include **subscription Bundles and Suites**, an updated **offer code redemption API** that now returns a `VerificationResult`, and an **enhanced App Review submission experience** for In-App Purchases.

## Key Topics

### Overview of Monthly Subscriptions with a 12-Month Commitment (0:51)
- A new billing plan type alongside the existing upfront annual option.
- Customers pay the monthly rate each month; the full 12-month commitment is contractually enforced.
- Can be added to any new or existing one-year subscription in App Store Connect.
- Both billing plans (upfront annual and monthly commitment) can be offered at the same time, giving customers a choice.

### Set Up in App Store Connect (1:42)
- New "Billing plan" section on a subscription product's configuration page.
- Set separate pricing for the monthly commitment plan.
- Configure introductory and promotional offers per billing plan.
- Toggle availability by storefront.

### Merchandise with StoreKit (2:28–6:55)
- `Product.subscription.pricingTerms` returns an array of `SubscriptionPricingTerms`; filter by `.billingPlanType == .monthly` to surface the commitment plan's terms.
- `SubscriptionPricingTerms` exposes `billingDisplayPrice` (per-billing-period price) and `commitmentInfo.price` (total commitment price) — both should be shown to customers.
- `SubscriptionStoreView` gains `.preferredSubscriptionPricingTerms` modifier to default the UI to a specific billing plan.
- Purchase is initiated with `product.purchase(options: [.billingPlanType(.monthly)])`.
- `manageSubscriptionsSheet(isPresented:subscriptionGroupID:)` modifier (existing) is used to let customers switch billing plans or cancel.
- StoreKit Testing in Xcode 27 supports simulating monthly commitment billing cycles.

### Monitor Subscriptions with App Store Server APIs (6:55–8:50)
- **`JWSTransaction`** gains new fields: **`NEW`** `billingPlanType` (`"MONTHLY"` or `"BILLED_UPFRONT"`), and a **`NEW`** `commitmentInfo` object containing:
  - `billingPeriodNumber` — current period (1–12)
  - `totalBillingPeriods` — always 12
  - `commitmentExpiresDate` — Unix ms timestamp when the 12-month commitment ends
  - `commitmentPrice` — total price in currency's smallest unit
- **`JWSRenewalInfo`** gains: **`NEW`** `renewalBillingPlanType`, and a **`NEW`** `commitmentInfo` object containing:
  - `commitmentAutoRenewProductId` — product the subscription will renew to after commitment ends
  - `commitmentAutoRenewStatus` — renewal intent flag
  - `commitmentRenewalDate`, `commitmentRenewalPrice`, `commitmentRenewalBillingPlanType`
- App Store Server Notifications V2 delivers these fields in `SUBSCRIBED`, `DID_RENEW`, and related notification types.

### Bundles and Suites (8:50)
- **Subscription Bundles** — package multiple subscription products from the same app at a combined price.
- **Subscription Suites** — cross-app subscription offering, letting a single purchase cover subscriptions across multiple apps from the same developer.
- Both configured in App Store Connect; no new client-side StoreKit API surface described beyond existing purchase flows.

### Offer Code Redemption (9:26)
- The `offerCodeRedemption(options:isPresented:onCompletion:)` view modifier is updated: **`NEW`** it now accepts an `options` parameter (a `Set<RedeemOption>`) and the completion handler receives a `Result<VerificationResult<Transaction>, Error>` instead of a raw transaction.
- This aligns offer code redemption with the standard StoreKit 2 `VerificationResult` pattern, making it easier to handle signature verification in a single code path.

### Enhanced Submission Experience (10:35)
- Revamped In-App Purchase submission flow in App Store Connect.
- Inline review notes, structured metadata fields, and attachment support per IAP product.
- Supports submitting multiple IAP products in a single review request alongside an app version.

## APIs & Frameworks

### StoreKit
- **`Product.subscription.pricingTerms`** — `[SubscriptionPricingTerms]`; array now includes commitment plan entry **[NEW behavior]**
- **`SubscriptionPricingTerms`** — **[NEW fields]** `billingPlanType` (`.monthly` / `.billedUpfront`), `billingDisplayPrice`, `commitmentInfo`
- **`SubscriptionPricingTerms.CommitmentInfo`** **[NEW]** — `price`, `billingPeriodNumber`, `totalBillingPeriods`, `commitmentExpiresDate`
- **`Product.PurchaseOption.billingPlanType(_:)`** **[NEW]** — pass `.monthly` to select the commitment billing plan at purchase time
- **`SubscriptionStoreView`** — existing SwiftUI view; gains `.preferredSubscriptionPricingTerms` modifier **[NEW]**
- **`.preferredSubscriptionPricingTerms(_:)`** view modifier **[NEW]** — closure receives `(Product, SubscriptionInfo)`, returns preferred `SubscriptionPricingTerms?`
- **`.manageSubscriptionsSheet(isPresented:subscriptionGroupID:)`** — existing modifier; unchanged API, now surfaces billing plan switching
- **`.offerCodeRedemption(options:isPresented:onCompletion:)`** **[NEW signature]** — `options: Set<RedeemOption>`, completion: `Result<VerificationResult<Transaction>, Error>`
- **`RedeemOption`** **[NEW]** — option set for offer code redemption behaviour
- **`Transaction.currentEntitlements`** — existing async sequence; unchanged

### App Store Server APIs
- **`JWSTransaction`** — new fields: `billingPlanType` **[NEW]**, `commitmentInfo` object **[NEW]**
- **`JWSRenewalInfo`** — new fields: `renewalBillingPlanType` **[NEW]**, `commitmentInfo` object **[NEW]**
- **App Store Server Notifications V2** — existing notification system; carries new `billingPlanType` and `commitmentInfo` fields

### App Store Connect
- **Monthly billing plan configuration** **[NEW]** — per-subscription-product billing plan setup
- **Subscription Bundles** **[NEW]** — multi-product bundle configuration
- **Subscription Suites** **[NEW]** — cross-app subscription configuration
- **Enhanced IAP submission experience** **[NEW]** — structured review notes and multi-product submissions

### Testing
- **StoreKit Testing in Xcode 27** — supports simulating monthly commitment billing cycles and commitment expiry

## Code Highlights

Merchandise both billing plans and initiate a monthly commitment purchase:
```swift
let pricingTerms = product?.subscription?.pricingTerms
    .first(where: { $0.billingPlanType == .monthly })
let monthlyPrice = pricingTerms?.billingDisplayPrice
let totalPrice   = pricingTerms?.commitmentInfo.price

let result = try? await product?.purchase(options: [.billingPlanType(.monthly)])
```

SwiftUI subscription store defaulting to the monthly plan:
```swift
SubscriptionStoreView(groupID: "3F19ED53") { /* marketing content */ }
    .preferredSubscriptionPricingTerms { _, subscriptionInfo in
        subscriptionInfo.pricingTerms.first { $0.billingPlanType == .monthly }
    }
```

Updated offer code redemption returning `VerificationResult`:
```swift
.offerCodeRedemption(options: [], isPresented: $presentingOfferCodeSheet) { result in
    switch result {
    case .success(let verificationResult): // verify & finish
    case .failure(let error): // handle
    }
}
```

## Takeaways
- **Monthly subscriptions with a 12-month commitment** lower the upfront barrier for customers while guaranteeing a full year of revenue; both billing plans can coexist on the same product.
- `Product.subscription.pricingTerms` and the new `billingPlanType` / `commitmentInfo` fields provide all the data needed to merchandise the price difference clearly (monthly price vs. total commitment price).
- `JWSTransaction` and `JWSRenewalInfo` carry new `billingPlanType` and `commitmentInfo` fields — server-side subscription management must be updated to track commitment period progress.
- The offer code redemption modifier now returns a full `VerificationResult<Transaction>`, aligning it with the standard StoreKit 2 verification pattern.

---
_Source: WWDC26 Session 210 page (abstract, chapter summaries, code samples, and resource links)._
