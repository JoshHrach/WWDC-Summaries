# Support Customers and Handle Refunds
**WWDC21 · Session 10175** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10175/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session is the third part of a three-part series on in-app purchases, focusing on customer support tooling and refund workflows. It introduces several new StoreKit 2 and App Store Server APIs that allow developers to provide contextual, in-app support for purchase-related issues without requiring customers to leave the app or contact Apple directly.

The session covers the full support lifecycle: identifying customer purchases via invoice lookup, compensating subscribers for outages, enabling in-app subscription management, and allowing customers to initiate refund requests from within the app. These APIs reduce friction, improve retention, and increase customer satisfaction.

A second speaker covers the refund decisioning side: how developers can now respond to Apple's new `CONSUMPTION-REQUEST` server notification to share consumption data that informs Apple's refund decision for consumable in-app purchases.

## Key Topics

### Invoice Lookup API
Developers can ask customers for the order ID from their App Store invoice email and use the new App Store Server API invoice lookup endpoint to validate the purchase and identify refund status. The response returns signed JWS transactions and a `status` field (0 = valid, 1 = invalid order ID, 2 = valid but no matching transactions).

### Lookup Refunded Transactions API
A new server-to-server API allows developers to query all refunded transactions for a customer using any `originalTransactionId` from their app. This is useful for recovering from server outages when refund notifications may have been missed.

### Subscription Offer Codes (Compensation)
Using `presentCodeRedemptionSheet` (StoreKit), developers can surface offer code redemption directly in the app as a compensation mechanism for service issues. Codes can be distributed via email, chat, or any channel.

### Extend Renewal Date API
A new server-to-server API allows developers to extend a paid subscriber's renewal date by up to 90 days, up to twice per calendar year. This is used to appease customers for outages or canceled events. The extension period does not count toward the 85% proceeds threshold.

### `showManageSubscriptions()` (StoreKit 2)
A single-line API call that presents the standard manage subscriptions sheet inside the app, enabling customers to upgrade, downgrade, or cancel without visiting the App Store.

### `beginRefundRequest` (StoreKit 2)
A new API enabling customers to request a refund for an in-app purchase directly from within the app. Approved refunds trigger a `REFUND` server notification; denied requests trigger a new `REFUND_DECLINED` notification. Both sandbox and production are supported.

### Consumption API and `CONSUMPTION-REQUEST` Notification
When a refund is requested for a consumable IAP, Apple sends a `CONSUMPTION-REQUEST` server notification. Developers must respond within 12 hours with consumption data (consumption status, delivery status, account age, playtime, spend history, etc.) to inform Apple's refund decisioning system.

## APIs & Frameworks

**StoreKit 2 (new)**
- `showManageSubscriptions()` **[NEW]** — displays the standard manage subscriptions UI in-app
- `beginRefundRequest(for:in:)` **[NEW]** — initiates a refund request for a specific transaction ID from within the app
- `presentCodeRedemptionSheet` — presents the offer code redemption UI

**App Store Server API (new endpoints)**
- Invoice Lookup API **[NEW]** — `GET /inApps/v1/lookup/{orderId}` — returns signed JWS transactions for an invoice order ID; response includes `status` field
- Refund Lookup API **[NEW]** — lookup refunded transactions by `originalTransactionId`
- Extend Renewal Date API **[NEW]** — extends subscription renewal date by up to 90 days, up to twice per calendar year; request includes `originalTransactionId`, extension duration (days), and reason code

**App Store Server Notifications**
- `REFUND` — sent when a refund is issued (all content types)
- `REFUND_DECLINED` **[NEW]** — sent when a refund request (initiated via StoreKit API) is denied
- `CONSUMPTION-REQUEST` **[NEW]** — sent when a customer requests a refund for a consumable IAP; triggers developer to submit consumption data

**Consumption API payload fields:**
- `customerConsented` — whether the user consented to sharing data
- `consumptionStatus` — not consumed / partially consumed / fully consumed
- `platform` — cross-platform consumption location
- `sampleContentProvided` — whether a free sample/trial was offered
- `deliveryStatus` — whether the IAP was successfully delivered
- `appAccountToken` — UUID from StoreKit 2 associated with the user's account
- Account lifetime, playtime, total spend, account status fields

**Transaction payload fields (JWS)**
- `originalTransactionId`
- `productId`
- `purchaseDate`
- `webOrderLineItemId`

## Code Highlights

Presenting the manage subscriptions sheet (one line):
```swift
try await AppStore.showManageSubscriptions(in: scene)
```

Initiating a refund request:
```swift
do {
    let status = try await transaction.beginRefundRequest(in: windowScene)
} catch StoreKitError.userCancelled {
    // User cancelled
} catch {
    // Handle other errors
}
```

Invoice lookup (server-side, pseudocode):
```
GET /inApps/v1/lookup/{orderId}?appAppleId={appAppleId}
// Response: { status: 0, signedTransactions: [...] }
```

## Takeaways
- New StoreKit 2 APIs (`showManageSubscriptions`, `beginRefundRequest`) allow fully in-app support flows for subscriptions and refunds — no App Store redirect needed.
- Three new App Store Server API endpoints cover invoice lookup, refund history lookup, and subscription renewal date extension.
- The new `CONSUMPTION-REQUEST` notification gives developers an active role in refund decisioning for consumable IAPs by supplying consumption data within 12 hours.
- New `REFUND_DECLINED` notification closes the loop on refund request outcomes for all content types.

---
_Source: WWDC21 Session 10175 page (abstract, chapter summaries, code samples, and resource links)._
