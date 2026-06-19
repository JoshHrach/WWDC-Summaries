# In-App Purchases and Using Server-to-Server Notifications
**WWDC19 · Session 302** · [Watch](https://developer.apple.com/videos/play/wwdc2019/302/)

_Platforms:_ iOS 13, iPadOS 13, macOS 10.15, tvOS 13, watchOS 6 (StoreKit / App Store Server)

## Overview
This session covers three areas: new StoreKit API additions in iOS 13 (`SKStorefront`), significant improvements to App Store Server-to-Server Notifications including four new notification types and the unified receipt payload, and best practices for managing the full subscription lifecycle including involuntary churn reduction via the new Billing Grace Period feature.

The core message is that server-to-server notifications, combined with the new unified receipt field, now carry enough information to manage a subscription lifecycle without polling `verifyReceipt` after most billing events. The `original_transaction_id` field is the stable key to link all notifications for a subscription across its lifetime.

## Key Topics

**SKStorefront (New in iOS 13)**
- Exposes the App Store storefront currently set on the device — a 3-letter ISO country code
- Accessed via `SKPaymentQueue.default().storefront`; returns an optional `SKStorefront` because it is a cached, device-specific value that may briefly be nil
- Use before making `SKProductsRequest` calls to avoid fetching metadata for products not available in the current storefront
- `SKPaymentTransactionObserver.paymentQueueDidChangeStorefront(_:)` **[NEW]** — called when the storefront changes; reload merchandised content in response
- `SKPaymentQueueDelegate.paymentQueue(_:shouldContinue:in:)` **[NEW]** — called mid-purchase if the storefront switches; return `false` plus `SKErrorStoreProductNotAvailable` error handling to cancel and inform the user

**App Pre-orders on watchOS 6**
- App pre-orders (introduced in iOS 11.2) now extend to watchOS 6
- Receipt soon to contain a flag indicating whether the app was obtained via pre-order, enabling targeted thank-you messages or bonus content for early adopters (available retroactively in receipts for iOS 11+)

**Server-to-Server Notifications — Unified Receipt**
- New `unified_receipt` field in the JSON payload (launching fall 2019)
- Contains: `latest_receipt` (Base64-encoded), `latest_receipt_info` (array of up to 100 purchases), `pending_renewal_info`, `status`, and `environment`
- Mirrors the structure of a `verifyReceipt` response so existing parsing logic reuses unchanged
- Receipt generated at notification time; must be stored server-side, never on-device (not tied to any specific device install)
- In most cases replaces the need to call `verifyReceipt` after receiving a notification

**New Server Notification Types (4 additional, launching fall 2019)**
- `DID_CHANGE_RENEWAL_STATUS` — user toggled auto-renew on or off from Manage Subscriptions; check `auto_renew_status` (true/false) and `auto_renew_status_change_date_ms`
- `DID_FAIL_TO_RENEW` — subscription failed to renew at first billing attempt; check `is_in_billing_retry_period` (1 = App Store actively retrying)
- `DID_RECOVER` — billing recovered during billing retry period (replaces old `RENEWAL` notification; both sent during transition); check new `purchase_date_ms` and `expires_date_ms`
- `PRICE_INCREASE_CONSENT` — subscriber entered a price increase consent flow; check `price_consent_status` (0 = not yet consented) and `price_increase_effective_date`; deploy in-app consent prompts

**Subscription Lifecycle Best Practices**
- Always track `original_transaction_id` as the stable unique identifier for a subscription
- Use `web_order_line_item_id` to link a notification to a specific `verifyReceipt` array entry
- On `INITIAL_BUY`: store `original_transaction_id`, enable service, validate receipt via `verifyReceipt` once
- On `INTERACTIVE_RENEWAL` / upgrade: `CANCEL` for lower tier + `INTERACTIVE_RENEWAL` for higher tier arrive together
- On `DID_CHANGE_RENEWAL_STATUS` with `auto_renew_status=false`: flag customer as at-risk and deploy win-back offers via Subscription Offers

**Billing Grace Period (New Feature, Fall 2019)**
- Opt-in via App Store Connect to offer a grace period during billing retry
- Pre-configured durations: 6 days for weekly subscriptions, 16 days for all other durations
- New `grace_period_expires_date_ms` field in `verifyReceipt` and notifications indicates the end of the grace period
- Keep service active during the grace period to earn revenue and improve recovery rates
- Apple recovers >77% of billing-failed subscriptions within 60 days; >80% of recoveries happen within the first 16 days

## APIs & Frameworks

**StoreKit**
- `SKStorefront` **[NEW]** — object exposing `countryCode` (3-letter ISO) and `identifier` (locale-region combination)
- `SKPaymentQueue.storefront: SKStorefront?` **[NEW]** — read current cached storefront
- `SKPaymentTransactionObserver.paymentQueueDidChangeStorefront(_:)` **[NEW]** — delegate callback when storefront changes
- `SKPaymentQueueDelegate.paymentQueue(_:shouldContinue:in:) -> Bool` **[NEW]** — intercept mid-purchase storefront switch
- `SKErrorCode.storeProductNotAvailable` — returned when `shouldContinue` returns `false` or product unavailable in storefront
- `SKProductsRequest` — unchanged; use storefront to pre-filter product identifiers before requesting
- `SKPaymentQueue.default()` — unchanged entry point

**App Store Server API (Server-Side)**
- `verifyReceipt` endpoint — unchanged; still needed for first validation after `INITIAL_BUY` and for full history beyond 100 transactions
- Server-to-Server Notification JSON fields **[NEW]**:
  - `unified_receipt` object with `latest_receipt`, `latest_receipt_info`, `pending_renewal_info`, `status`, `environment`
  - `notification_type`: `DID_CHANGE_RENEWAL_STATUS`, `DID_FAIL_TO_RENEW`, `DID_RECOVER`, `PRICE_INCREASE_CONSENT` **[NEW notification types]**
  - `auto_renew_status`, `auto_renew_status_change_date_ms`, `is_in_billing_retry_period`, `price_consent_status`, `price_increase_effective_date`, `grace_period_expires_date_ms` **[NEW fields]**

## Code Highlights

Reading the current storefront and filtering products:
```swift
// In SKPaymentTransactionObserverDelegate or AppDelegate
if let storefront = SKPaymentQueue.default().storefront {
    let countryCode = storefront.countryCode // "USA", "GBR", etc.
    let validIDs = allProductIDs.filter { shouldShow($0, inCountry: countryCode) }
    let request = SKProductsRequest(productIdentifiers: Set(validIDs))
    request.start()
}
```

Responding to a storefront change:
```swift
func paymentQueueDidChangeStorefront(_ queue: SKPaymentQueue) {
    guard let newStorefront = queue.storefront else { return }
    reloadMerchandising(for: newStorefront.countryCode)
}
```

Gating a mid-purchase storefront switch:
```swift
func paymentQueue(_ queue: SKPaymentQueue,
                  shouldContinue transaction: SKPaymentTransaction,
                  in newStorefront: SKStorefront) -> Bool {
    return shouldShow(transaction.payment.productIdentifier, inCountry: newStorefront.countryCode)
}
```

## Takeaways
- Add `SKPaymentTransactionObserver.paymentQueueDidChangeStorefront` to all apps using in-app purchase to stay current with the active storefront without polling.
- Adopt `unified_receipt` in server-to-server notifications to eliminate most `verifyReceipt` polling calls; only poll when you need >100 transactions of history.
- Handle all eight notification types by action: `INITIAL_BUY` (activate), `INTERACTIVE_RENEWAL` (update expires), `DID_CHANGE_RENEWAL_STATUS=false` (retain), `DID_FAIL_TO_RENEW` (suspend or grace), `DID_RECOVER` (reactivate), `CANCEL` (deactivate), `PRICE_INCREASE_CONSENT` (in-app prompt).
- Opt in to Billing Grace Period in App Store Connect — the recovery data shows it directly increases revenue for the 77% of billing-failed subscriptions that eventually recover.

---
_Source: WWDC19 Session 302 page (abstract, chapter summaries, code samples, and resource links)._
