# What's New with In-App Purchase
**WWDC20 · Session 10661** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10661/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 6.2+

## Overview
In-App Purchase in 2020 expanded along three axes: server infrastructure, client APIs, and ecosystem reach. On the server side, a new Refund App Store Server Notification extends the existing subscription notification system to consumables, non-consumables, and non-renewing subscriptions, while a new `DID_RENEW` notification eliminates the need to poll `/verifyReceipt` for successful auto-renewals. On the client side, Family Sharing for in-app purchases lets up to five family members access a single subscription or non-consumable, `SKOverlay` provides a floating app-promotion view that works inside App Clips, and `SKAdNetwork 2.0` adds conversion values, re-download tracking, and source app attribution. StoreKit also shipped in-app purchases directly on watchOS, and an improved subscription price increase consent flow reduces churn.

## Key Topics

### Refund App Store Server Notification (New for Non-Subscription IAP)
A brand-new `REFUND` notification type fires whenever Apple issues a refund for a consumable, non-consumable, or non-renewing subscription. (Subscriptions continue to use the existing `CANCEL` notification.)
- Delivered as a JSON POST to the configured App Store Connect endpoint; retried up to 3 times on no HTTP 200
- Payload: `original_transaction_id`, `cancellation_date`, `cancellation_reason` (0 = other, 1 = app issue), `bid`, `product_id` — all in `unified_receipt.latest_receipt_info`
- Live immediately; no code changes required to receive it if server notifications are already enabled
- Enables: in-app messaging, access revocation, balance deduction (consumables), cross-platform restriction

### DID_RENEW Notification (New Subscription Notification Type)
Fires after every successful auto-renewal, completing the full subscription lifecycle coverage:
- Contains `unified_receipt` with `original_transaction_id`, new `transaction_id`, new `expiration_date`, and `auto_renew_product_id`
- Eliminates the need to schedule `/verifyReceipt` polls for every subscriber; use `/verifyReceipt` only as a recovery mechanism during server outages or to verify status on first launch

### Pending Renewal Info: Promotional Offer ID
`pending_renewal_info` in both `/verifyReceipt` responses and App Store Server Notification payloads now includes `promotional_offer_id` — the offer that will be applied at the next renewal. Available today.

### Family Sharing for In-App Purchases
- Up to 5 additional family members can share auto-renewable subscriptions and non-consumable IAPs
- Enabled per-product in App Store Connect; once enabled it cannot be disabled
- New purchases: shared by default (subscriptions); shared if purchase-sharing is on (non-consumables)
- Existing purchases: must opt in (subscriptions) or restore purchases (non-consumables)
- Family member devices receive transactions that look like restored purchases — existing restore logic handles them without new code
- When a purchaser disables sharing or leaves a family group, the new `paymentQueue(_:didRevokeEntitlementsForProductIdentifiers:)` delegate method is called; receipt is auto-updated; products no longer appear in receipt
- `SKProduct.isFamilySharable` — new boolean property; use it to display sharing status to customers without hard-coding product IDs

### SKOverlay (New)
Floating app-promotion UI element presented at the bottom of a scene:
- `SKOverlay.AppClipConfiguration` — transitions users from an App Clip to the full app
- `SKOverlay.AppConfiguration` — promotes any app by iTunes ID; `userDismissible` controls swipe-down dismissal
- Both configurations have `position` (`.bottom` or `.bottomRaised` for tab bar apps), `campaignToken`, `providerToken`, and arbitrary key-value pairs for analytics integration
- `SKOverlay.present(in:)` / `SKOverlay.dismiss(in:)` — class-level dismiss operates on the scene, not the object
- `SKOverlayDelegate` — methods for presentation start/end and dismissal start/end, each receiving `SKOverlayTransitionContext` to coordinate app UI animations alongside the overlay

### SKAdNetwork 2.0
Expanded postback data for ad attribution while preserving privacy:
- **Version 2.0** — required new field in ad impression payload; also requires `source_app_id`
- **`updateConversionValue(_:)`** — new API called by the advertised app to record a 6-bit action value (0–63); only higher values accepted on subsequent calls (monotonically increasing)
- **Postback additions**: `redownload` (bool — first install vs. reinstall), `source-app-id` (source app), `conversion-value` (set by advertising app); latter two are privacy-gated by App Store server calculations and may be omitted

### In-App Purchase on watchOS (Shipped in watchOS 6.2)
Full StoreKit API available on watchOS:
- Observe payment queue, request products, add to payment queue — same as iOS
- Receipt local validation: use `WKInterfaceDevice.current().identifierForVendor` instead of `UIDevice`; hash construction is otherwise identical

### Subscription Price Increase Consent Flow (Improved, Shipped in iOS 13.3)
- System automatically presents a consent sheet when a user opens an app with a pending price increase for their subscription
- `SKPaymentQueueDelegate.paymentQueueShouldShowPriceConsent(_:) -> Bool` — return `false` to defer the sheet
- `SKPaymentQueue.showPriceConsentIfNeeded()` — call later to display the sheet at a better moment (only shows if a pending increase exists)

## APIs & Frameworks

### App Store Server Notifications
- `REFUND` notification type **[NEW]** — consumable, non-consumable, non-renewing subscription refunds
- `DID_RENEW` notification type **[NEW]** — fires on every successful auto-renewal
- `promotional_offer_id` in `pending_renewal_info` **[NEW]**

### StoreKit (Client)
- `SKProduct.isFamilySharable: Bool` **[NEW]** — indicates if a product supports Family Sharing
- `SKPaymentTransactionObserver.paymentQueue(_:didRevokeEntitlementsForProductIdentifiers:)` **[NEW]** — called when Family Sharing is disabled for a product or a family member leaves the group; also called on non-consumable/subscription refunds
- `SKOverlay` **[NEW]** — floating app-promotion overlay
  - `SKOverlay.AppClipConfiguration(position:)` **[NEW]**
  - `SKOverlay.AppConfiguration(appIdentifier:position:)` **[NEW]**
  - `SKOverlay.present(in: UIWindowScene)` **[NEW]**
  - `SKOverlay.dismiss(in: UIWindowScene)` **[NEW]** (class method)
  - `SKOverlayDelegate` **[NEW]** — `willStartPresentation`, `didFinishPresentation`, `willStartDismissal`, `didFinishDismissal` (each with `SKOverlayTransitionContext`)
- `SKAdNetwork.registerAppForAdNetworkAttribution()` — existing; initializes postback on first advertised app launch
- `SKAdNetwork.updateConversionValue(_ conversionValue: Int)` **[NEW]** — sets or updates the 6-bit conversion value (monotonically increasing; 0–63)
- `SKPaymentQueueDelegate.paymentQueueShouldShowPriceConsent(_:) -> Bool` **[NEW — iOS 13.3]**
- `SKPaymentQueue.showPriceConsentIfNeeded()` **[NEW — iOS 13.3]**
- `WKInterfaceDevice.current().identifierForVendor` — use on watchOS for receipt local validation (replaces `UIDevice`)

## Code Highlights

Checking Family Sharing eligibility and responding to revocation:
```swift
// Check if a product supports Family Sharing
if product.isFamilySharable {
    // Show "Family Sharing included" badge in your UI
}

// SKPaymentTransactionObserver
func paymentQueue(_ queue: SKPaymentQueue,
                  didRevokeEntitlementsForProductIdentifiers productIdentifiers: [String]) {
    // Refresh receipt (local or server-side) and revoke access for listed products
    // Note: user may still be entitled via another overlapping purchase — check receipt, not just the array
    refreshReceiptAndUpdateEntitlements()
}
```

Presenting an SKOverlay from an App Clip:
```swift
guard let scene = view.window?.windowScene else { return }
let config = SKOverlay.AppClipConfiguration(position: .bottom)
let overlay = SKOverlay(configuration: config)
overlay.delegate = self
SKOverlay.present(overlay, in: scene)
```

Controlling subscription price increase consent timing:
```swift
// SKPaymentQueueDelegate
func paymentQueueShouldShowPriceConsent(_ paymentQueue: SKPaymentQueue) -> Bool {
    // Return false to delay; call showPriceConsentIfNeeded() later
    return shouldShowNow
}

// Later, when the user has been educated about new value:
SKPaymentQueue.default().showPriceConsentIfNeeded()
```

watchOS receipt validation device identifier:
```swift
#if os(watchOS)
let deviceID = WKInterfaceDevice.current().identifierForVendor
#else
let deviceID = UIDevice.current.identifierForVendor
#endif
// Use deviceID with your PAKE value to hash and validate the receipt
```

## Takeaways

- The `REFUND` notification is the first App Store Server Notification for non-subscription content — enable it immediately if you sell consumables or non-consumables, as it fires today with no additional configuration if notifications are already set up.
- `DID_RENEW` completes the subscription lifecycle notification coverage; combined with the other notification types, you no longer need to poll `/verifyReceipt` for subscription status updates — use it only as a recovery path.
- Family Sharing for IAP is enabled per-product in App Store Connect and requires `SKPaymentTransactionObserver.paymentQueue(_:didRevokeEntitlementsForProductIdentifiers:)` to handle access revocation; existing restore logic handles share grants automatically.
- `SKOverlay` is the recommended way to drive full-app downloads from App Clips and to cross-promote apps without leaving the current experience.
- `SKAdNetwork 2.0` adds conversion value tracking and source app attribution, giving ad networks significantly more signal while maintaining Apple's privacy protections.

---
_Source: WWDC20 Session 10661 page (abstract, transcript, and resource links)._
