# Meet StoreKit for SwiftUI
**WWDC23 · Session 10013** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10013/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10, tvOS 17, visionOS 1

## Overview
StoreKit 2 gains a comprehensive suite of SwiftUI views in 2023 that abstract the entire in-app purchase merchandising flow — product data loading, caching, purchase initiation, and entitlement checking — behind declarative SwiftUI components. `StoreView`, `ProductView`, and `SubscriptionStoreView` handle all common functionality out of the box while exposing rich customization hooks for branding and layout.

The views integrate natively with Xcode Previews and StoreKit configuration files so developers can iterate on purchase UI without hitting the App Store. New view modifier APIs (`subscriptionStatusTask`, `currentEntitlementTask`, `onInAppPurchaseCompletion`) provide reactive, event-driven patterns for tracking subscription state and handling purchase outcomes, enabling apps to keep their UI synchronized with entitlement changes automatically.

All three primary views are cross-platform and automatically adapt to each platform's conventions and form factors.

## Key Topics

### StoreView
- Quickest path to a functional, well-designed store UI
- Initialized with a collection of product IDs; loads display names, descriptions, and prices from App Store / StoreKit configuration file automatically
- Trailing closure provides per-product decorative icons via `(Product) -> View`
- Handles Screen Time purchase restrictions, data caching, and expiration automatically
- Style controlled with `.productViewStyle(_:)` modifier (same as `ProductView`)
- Cross-platform: adapts layout for iPhone, iPad, Mac, Apple Watch, Apple TV

### ProductView
- Used for custom store layouts; `StoreView` is composed of `ProductView` instances
- Initialize with product ID: `ProductView(id:)` or with a pre-loaded `Product` value to skip async loading
- Trailing icon closure for decorative artwork; second closure for placeholder icon during loading
- `prefersPromotionalIcon: true` parameter uses App Store promotional icon (set up in App Store Connect)
- `.inAppPurchaseOptions()` modifier applies the icon border treatment to custom SwiftUI icons
- Three standard styles: `.compact`, `.regular` (default for lists/shelves), `.large` (hero presentation)
- Custom styles: conform to `ProductViewStyle` protocol, implement `makeBody(configuration:)`
  - `ProductViewStyleConfiguration` provides: `state` enum (loading, success, unavailable, failure), `Product` value, decorative icon, `purchase()` method (must use this — not `Product.purchase()` — to trigger `onInAppPurchaseCompletion`)

### SubscriptionStoreView
- Purpose-built for subscription groups
- Initialize with `groupID` from App Store Connect / StoreKit configuration
- Automatically checks subscriber status and introductory offer eligibility
- Marketing content header: replace with any SwiftUI view via trailing closure
- Container background: `.containerBackground(_:for: .subscriptionStore)` modifier (or `.subscriptionStoreHeaderBackground`, `.subscriptionStoreFullHeightBackground` placements)
- Background style: `.backgroundStyle(.clear)` to make subscription controls area transparent
- `visibleRelationships` parameter — pass `.upgrade` to show only upgrade options for existing subscribers

### Subscription Controls Styling
- `.subscriptionStoreControlStyle(_:)` – choose control layout: `.automatic`, `.picker`, `.prominentPicker`, `.buttons`
- `.subscriptionStoreButtonLabel(_:)` – `.singleLine`, `.multiline`; configure label content: `.displayName`, `.price`, or composed combinations
- `.subscriptionStoreControlIcon` modifier – add per-plan decorative icon using `(Product, SubscriptionInfo) -> View`
- `.subscriptionStorePickerItemBackground(_:)` – shape style for subscription plan option backgrounds

### Auxiliary Buttons (`storeButton` modifier)
- Applies to `StoreView` and `SubscriptionStoreView`
- Visibility: `.automatic`, `.visible`, `.hidden`
- Button kinds: `.cancellation` (dismiss button), `.restorePurchases`, `.redeemCode`, `.signIn`, `.policies`
- `.subscriptionStoreSignInAction` modifier – declares sign-in action; enables `.signIn` button automatically
- `.subscriptionStorePolicyForegroundStyle(_:)` – style for policy buttons (terms/privacy links)

### Reactive Entitlement APIs
- `.subscriptionStatusTask(groupID:)` modifier – background task loads subscription status on view appear, calls handler on every status change; provides `EntitlementTaskState<[Product.SubscriptionInfo.Status]>`
- `.currentEntitlementTask(for: productID)` modifier – for non-consumable / non-renewing subscriptions; same `EntitlementTaskState` pattern
- `.onInAppPurchaseCompletion` modifier – called after any descendant StoreKit view finishes a purchase; receives `Product` and `Result<Product.PurchaseResult, Error>`
- `.onInAppPurchaseStart` modifier – called before purchase begins; receives `Product` about to be purchased
- `.storeProductsTask(ids:)` modifier – pre-loads `Product` values for a collection of IDs; enables parent view to manage loading state and pass `Product` values directly to `ProductView`

## APIs & Frameworks

- **StoreKit** / **StoreKit 2** – in-app purchase framework
- `StoreView(ids:)` **[NEW]** – list-style merchandising view for multiple products
- `ProductView(id:)` / `ProductView(_:)` **[NEW]** – single-product merchandising view
- `SubscriptionStoreView(groupID:)` **[NEW]** – subscription-group merchandising view
- `.productViewStyle(_:)` modifier **[NEW]** – `.compact`, `.regular`, `.large`, custom `ProductViewStyle`
- `ProductViewStyle` protocol **[NEW]** – custom style; `makeBody(configuration:)` requirement
- `ProductViewStyleConfiguration` **[NEW]** – provides `state`, `Product`, icon, `purchase()` method
- `ProductViewStyleConfiguration.State` **[NEW]** – `.loading`, `.success(Product)`, `.unavailable`, `.failure`
- `.subscriptionStoreControlStyle(_:)` modifier **[NEW]**
- `SubscriptionStoreControlStyle` **[NEW]** – `.automatic`, `.picker`, `.prominentPicker`, `.buttons`
- `.subscriptionStoreButtonLabel(_:)` modifier **[NEW]**
- `SubscriptionStoreButtonLabel` **[NEW]** – `.singleLine`, `.multiline`, `.displayName`, `.price`
- `.subscriptionStoreControlIcon(_:)` modifier **[NEW]**
- `.subscriptionStorePickerItemBackground(_:)` modifier **[NEW]**
- `.subscriptionStorePolicyForegroundStyle(_:)` modifier **[NEW]**
- `.subscriptionStoreSignInAction(_:)` modifier **[NEW]**
- `.storeButton(_:for:)` modifier **[NEW]** – controls auxiliary button visibility
- `StoreButtonKind` **[NEW]** – `.cancellation`, `.restorePurchases`, `.redeemCode`, `.signIn`, `.policies`
- `.subscriptionStatusTask(groupID:priority:action:)` modifier **[NEW]**
- `.currentEntitlementTask(for:priority:action:)` modifier **[NEW]**
- `EntitlementTaskState` **[NEW]** – `.loading`, `.failure(Error)`, `.success(…)`
- `.onInAppPurchaseCompletion(perform:)` modifier **[NEW]**
- `.onInAppPurchaseStart(perform:)` modifier **[NEW]**
- `.storeProductsTask(ids:priority:action:)` modifier **[NEW]**
- `.containerBackground(_:for:)` modifier (SwiftUI) – positions backgrounds in SubscriptionStoreView
- `Product` – StoreKit 2 type for in-app purchase product metadata
- `Product.SubscriptionInfo.Status` – subscriber status type
- `Transaction.updates` – async sequence of transaction updates (default path without `onInAppPurchaseCompletion`)

## Code Highlights

Minimal store with custom icons:
```swift
import SwiftUI
import StoreKit

struct BirdFoodShop: View {
    @Query var birdFood: [BirdFood]

    var body: some View {
        StoreView(ids: birdFood.productIDs) { product in
            BirdFoodProductIcon(productID: product.id)
        }
    }
}
```

Hero product with large style + purchase completion handler:
```swift
ProductView(id: bestValueProduct.id) {
    BirdFoodProductIcon(birdFood: birdFood, quantity: product.quantity)
}
.productViewStyle(.large)
.onInAppPurchaseCompletion { product, result in
    if case .success(let purchaseResult) = result {
        await BirdBrain.shared.process(purchaseResult)
    }
}
```

Subscription store with branding and upgrade path:
```swift
SubscriptionStoreView(groupID: passGroupID, visibleRelationships: .upgrade) {
    MarketingContentView()
        .containerBackground(.blue.gradient, for: .subscriptionStore)
}
.backgroundStyle(.clear)
.subscriptionStoreButtonLabel(.multiline)
.subscriptionStorePickerItemBackground(.ultraThinMaterial)
.storeButton(.visible, for: .redeemCode, .cancellation)
```

Reactive subscription status:
```swift
BackyardGrid()
    .subscriptionStatusTask(groupID: passGroupID) { taskState in
        self.passStatus = await BirdBrain.shared.processPassStatus(taskState.value ?? [])
    }
```

## Takeaways
- `StoreView`, `ProductView`, and `SubscriptionStoreView` replace dozens of lines of data-loading, state-management, and purchase-handling code with a handful of declarative SwiftUI declarations.
- Custom `ProductViewStyle` conformances give full control over layout while retaining all App Store data flow, caching, and purchase infrastructure.
- Use `subscriptionStatusTask` and `currentEntitlementTask` to keep UI in sync with entitlement changes without polling — the system calls your handler whenever status changes.
- Always call `configuration.purchase()` (not `product.purchase()`) inside custom styles to ensure `onInAppPurchaseCompletion` fires and the payment sheet anchors correctly.

---
_Source: WWDC23 Session 10013 page (abstract, chapter summaries, code samples, and resource links)._
