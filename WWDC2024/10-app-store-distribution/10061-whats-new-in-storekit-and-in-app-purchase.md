# What's New in StoreKit and In-App Purchase
**WWDC24 · Session 10061** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10061/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, tvOS 18, watchOS 11, visionOS 2

## Overview
StoreKit 2024 brings four interconnected improvements: richer transaction data (finished consumables in history, new `price`/`currency`/`renewalPrice` fields), a powerful new subscription merchandising API (subscription option groups, six control styles including three new ones, custom control styles, and platform-specific placements), win-back offers as a new offer type for recovering churned subscribers, and expanded StoreKit Testing in Xcode (privacy policy testing, localization testing, win-back simulation, billing issue message testing, and purchase intent testing). The Original API for In-App Purchase (StoreKit 1) is deprecated in iOS 18 and aligned releases.

## Key Topics

### Core API Enhancements
**Finished consumable transactions** are now included in the transaction history APIs (opt-in via `SKIncludeConsumableInAppPurchaseHistory = true` in `Info.plist`). Previously, finished consumables disappeared from history after completion; now all consumables appear regardless of finished state. Finished consumables are also accessible via the App Store Server API.

**New Transaction fields**: `currency` (the currency at time of purchase) and `price` (the configured price from App Store Connect). **New RenewalInfo fields**: `currency` and `renewalPrice` (the amount charged at next renewal). These fields are available in Xcode 16 builds but back-deployed to iOS 15.

**Win-back offers**: A new fourth offer type for auto-renewable subscriptions (joining introductory, promotional, and referral). Designed to re-engage churned subscribers. Eligibility rules are configured in App Store Connect. The offer can surface automatically via the StoreKit Message API with no additional code, and may be promoted by the App Store editorial team on the Today, Games, and Apps tabs.

### Subscription Store View Enhancements
**Subscription option groups** let developers declare a hierarchy of subscription plans within `SubscriptionStoreView`. Each group is defined by a condition (which products belong), an optional label, and optional custom marketing content. Multiple groups trigger an automatic tab view or links presentation style.

`SubscriptionOptionGroupSet` is the convenience API: provide a closure mapping each product to a group identifier value; the system creates one group per unique value. `SubscriptionPeriodGroupSet` is a further convenience for grouping by subscription period.

**Control styles** — three new standard styles added (total now six):
- `compactPicker` — horizontal shelf, ideal for 2–3 plans with the same service level
- `pagedPicker` — horizontal paging, shows more detail per plan
- `pagedProminentPicker` — like paged picker but with prominent border and scale on selected plan

**Control placement** (`subscriptionStoreControlStyle(_:placement:)`) — new API to position controls at `.bottomBar` or other placements. Some placements are platform-specific; the API statically indicates which placements are compatible with each style. On tvOS 18, the `buttons` style gains new horizontal layouts with `.leading`, `.trailing` (new default), and `.bottom` placements.

**Custom control styles**: Conform to `SubscriptionStoreControlStyle` and implement `makeBody(configuration:)`. Use the new `SubscriptionPicker` API to build a picker view, and `SubscribeButton` for the purchase confirmation button. Access per-option data including `displayName`, `isFamilyShareable`, `description`, `isSelected`, and localized price display.

### StoreKit Testing in Xcode
- App policies section in StoreKit configuration: test privacy policy and license agreement text locally
- Localization testing for subscription group display names
- Win-back offer configuration in StoreKit configuration file
- In-app purchase image preview (test `prefersPromotionalIcon`)
- Dialogs setting: disable system dialogs for automation/UI testing (auto-chooses defaults)
- Purchase intents: send a purchase intent to the simulator directly from the Transaction Manager (since Xcode 15.2)
- Billing issue messages: test billing retry and billing issue resolution sheet on iOS 18

### Original API Deprecation
The Original API for In-App Purchase (StoreKit 1: `SKPaymentQueue`, `SKProduct`, `SKPaymentTransaction`, unified receipt) is deprecated in iOS 18 and aligned releases. Existing apps continue to work; no new features will be added. Apple strongly recommends migrating to StoreKit 2.

## APIs & Frameworks

**StoreKit 2**
- `Transaction.currency: Currency?` **[NEW]** — currency at time of purchase
- `Transaction.price: Decimal?` **[NEW]** — configured price from App Store Connect
- `Transaction.all` / `Transaction.updates` — now includes finished consumables (opt-in) **[NEW behavior]**
- `SKIncludeConsumableInAppPurchaseHistory` — `Info.plist` key to opt in **[NEW]**
- `Product.SubscriptionInfo.RenewalInfo.currency: Currency?` **[NEW]**
- `Product.SubscriptionInfo.RenewalInfo.renewalPrice: Decimal?` **[NEW]**
- `Product.SubscriptionOffer.OfferType.winBack` **[NEW]** — win-back offer type
- Win-back offer eligibility — configured via App Store Connect, surfaced via `Message` API **[NEW]**

**StoreKit Views (SwiftUI)**
- `SubscriptionStoreView` — subscription merchandising view (existing)
- `SubscriptionOptionGroup` **[NEW]** — declare a group of subscription options within `SubscriptionStoreView`
- `SubscriptionOptionGroupSet` **[NEW]** — convenience API to create groups by mapped value
- `SubscriptionPeriodGroupSet` **[NEW]** — convenience grouping by subscription period
- `.subscriptionStoreControlStyle(_:placement:)` — control style + placement modifier **[NEW placement param]**
- `SubscriptionStoreControlStyle.compactPicker` **[NEW]** — horizontal compact shelf
- `SubscriptionStoreControlStyle.pagedPicker` **[NEW]** — horizontal paging detail
- `SubscriptionStoreControlStyle.pagedProminentPicker` **[NEW]** — paged with prominence border
- `.subscriptionStoreOptionGroupStyle(.tabs)` / `.links` **[NEW]** — group presentation style
- `SubscriptionStoreControlStyle` protocol **[NEW]** — conform to create custom control styles
- `SubscriptionPicker` **[NEW]** — building block for custom control styles
- `SubscribeButton` **[NEW]** — purchase confirmation button primitive for custom styles
- `SelectionIndicator` **[NEW]** — selection state primitive for custom styles

**StoreKit Testing (Xcode)**
- App policies section (privacy policy + license agreement in config) **[NEW]**
- Subscription group display name localization in config **[NEW]**
- Win-back offer configuration **[NEW]**
- In-app purchase image configuration **[NEW]**
- Dialogs enable/disable setting **[NEW]**
- Purchase intent simulation in Transaction Manager **[NEW]**
- Billing issue message simulation (iOS 18) **[NEW]**

**Deprecated (StoreKit 1 — iOS 18+)**
- `SKPaymentQueue` — deprecated
- `SKProduct` — deprecated
- `SKPaymentTransaction` — deprecated
- Unified receipt — deprecated

## Code Highlights

SubscriptionStoreView with option group set and compact picker in bottom bar:
```swift
SubscriptionStoreView(groupID: Self.subscriptionGroupID) {
    SubscriptionOptionGroupSet { product in
        StreamingPassLevel(product)
    } label: { streamingPassLevel in
        Text(streamingPassLevel.localizedTitle)
    } marketingContent: { streamingPassLevel in
        StreamingPassMarketingContent(level: streamingPassLevel)
        StreamingPassFeatures(level: streamingPassLevel)
    }
}
.subscriptionStoreControlStyle(.compactPicker, placement: .bottomBar)
```

Custom control style with family-sharing badge:
```swift
struct BadgedPickerControlStyle: SubscriptionStoreControlStyle {
    func makeBody(configuration: Configuration) -> some View {
        SubscriptionPicker(configuration) { pickerOption in
            HStack(alignment: .top) {
                VStack(alignment: .leading) {
                    Text(pickerOption.displayName).font(.title2.bold())
                    Text(priceDisplay(for: pickerOption))
                    if pickerOption.isFamilyShareable { FamilyShareableBadge() }
                    Text(pickerOption.description)
                }
                Spacer()
                SelectionIndicator(pickerOption.isSelected)
            }
        } confirmation: { option in
            SubscribeButton(option)
        }
    }
}
```

Reading new transaction fields:
```swift
for await result in Transaction.updates {
    if case .verified(let transaction) = result {
        let price = transaction.price
        let currency = transaction.currency
    }
}
```

## Takeaways
- Opt in to finished consumable transaction history (`SKIncludeConsumableInAppPurchaseHistory`) to simplify consumable tracking — the framework now manages history instead of your app.
- Use `SubscriptionOptionGroupSet` with `compactPicker` placement at `.bottomBar` to build polished, space-efficient subscription merchandising; use `pagedProminentPicker` when you want to highlight a single recommended plan.
- Create a custom `SubscriptionStoreControlStyle` when standard styles don't match your brand — `SubscriptionPicker`, `SubscribeButton`, and `SelectionIndicator` give you full creative control while StoreKit handles all purchase logic.
- Migrate off the Original StoreKit API (deprecated iOS 18+): StoreKit 2 provides back-deployment to iOS 15 for all new fields, making migration safe for any supported deployment target.

---
_Source: WWDC24 Session 10061 page (abstract, chapter summaries, code samples, and resource links)._
