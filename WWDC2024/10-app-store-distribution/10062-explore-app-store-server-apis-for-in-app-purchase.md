# Explore App Store Server APIs for In-App Purchase
**WWDC24 · Session 10062** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10062/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, tvOS 18, watchOS 11, visionOS 2

## Overview
The App Store Server API and App Store Server Notifications receive several targeted additions in 2024. Highlights include a new `ONE_TIME_CHARGE` notification type for non-consumable and non-renewing subscription purchases, an extension of the `CONSUMPTION_REQUEST` workflow to auto-renewable subscriptions, a new `consumptionRequestReason` field that explains why Apple is asking for consumption data, a new `refundPreference` field in the consumption submission, a revamped Get Transaction History v2 endpoint that returns all transactions regardless of finish/refund status, and richer transaction fields including price, currency, and offer discount type.

The session also introduces new fields in renewal information and highlights the open-source App Store Server Library (Swift, Java, Python, Node.js).

## Key Topics

### ONE_TIME_CHARGE Notification
Previously, `App Store Server Notifications` only sent real-time events for auto-renewable subscriptions. The new `ONE_TIME_CHARGE` notification type fires immediately when a user purchases a non-consumable IAP or non-renewing subscription. Apps can now handle refunds, revocations, and purchase confirmations server-side for all product types.

### CONSUMPTION_REQUEST for Subscriptions
The `CONSUMPTION_REQUEST` notification (previously only for consumables) now fires for auto-renewable subscriptions when a subscriber requests a refund. This gives server-side apps the opportunity to submit consumption data before Apple makes a refund decision.

### consumptionRequestReason
The `CONSUMPTION_REQUEST` notification payload now includes `consumptionRequestReason`, a string enum that tells the server *why* Apple is asking:
- `UNINTENDED_PURCHASE` — user says they bought by mistake
- `FULFILLMENT_ISSUE` — user says the content wasn't delivered
- `UNSATISFIED_WITH_PURCHASE` — user didn't find value
- `LEGAL` — legal/regulatory reason
- `OTHER`

Servers can use this to decide whether to proactively refund (e.g., fulfillment issue) or submit consumption data defending the charge.

### refundPreference in SendConsumptionInformation
`SendConsumptionInformation` gains a `refundPreference` field: `GRANT`, `DECLINE`, or `NO_PREFERENCE`. This lets servers signal their preference directly to Apple rather than relying solely on consumption data heuristics.

### Get Transaction History v2
The v2 endpoint (`GET /inApps/v2/history/{transactionId}`) returns all transactions for the app—including finished and refunded transactions—not just active/unfinished ones. This enables accurate lifetime purchase auditing. New sortable fields and filter parameters include `productType`, `inAppOwnershipType`, `revoked`, and `excludeRevoked`.

### New Transaction Fields
- `price: Int` and `currency: String` — the exact price paid at transaction time (in micro-units)
- `offerDiscountType` — `FREE_TRIAL`, `PAY_AS_YOU_GO`, `PAY_UP_FRONT`, or `OTHER`
- `renewalPrice: Int` in renewal info — the scheduled renewal price, useful for detecting price increase consent

### App Store Server Library
Open-source libraries (Swift, Java, Python, Node.js) handle JWT verification, payload decoding, and retry logic. Available on GitHub under `apple/app-store-server-library-*`.

## APIs & Frameworks

**App Store Server Notifications v2**
- `ONE_TIME_CHARGE` notification type **[NEW]** — fires for non-consumable and non-renewing subscription purchases
- `CONSUMPTION_REQUEST` — now also fires for auto-renewable subscription refund requests **[updated]**
- `consumptionRequestReason` field in `CONSUMPTION_REQUEST` payload **[NEW]**
  - Values: `UNINTENDED_PURCHASE`, `FULFILLMENT_ISSUE`, `UNSATISFIED_WITH_PURCHASE`, `LEGAL`, `OTHER`

**App Store Server API**
- `GET /inApps/v2/history/{transactionId}` **[NEW]** — Get Transaction History v2
  - Returns all transactions (including finished/refunded)
  - New filters: `productType`, `inAppOwnershipType`, `revoked`, `excludeRevoked`
- `PUT /inApps/v1/transactions/consumption/{transactionId}` — SendConsumptionInformation
  - `refundPreference` field **[NEW]** — `GRANT`, `DECLINE`, `NO_PREFERENCE`
- Transaction fields:
  - `price: Int` **[NEW]** — price in micro-units (e.g., 990000 = $0.99)
  - `currency: String` **[NEW]** — ISO 4217 currency code
  - `offerDiscountType: String` **[NEW]** — `FREE_TRIAL`, `PAY_AS_YOU_GO`, `PAY_UP_FRONT`, `OTHER`
- Renewal Info fields:
  - `renewalPrice: Int` **[NEW]** — upcoming renewal price
  - `currency: String` **[NEW]** (in renewal context)

**App Store Server Library (open source)**
- Swift: `apple/app-store-server-library-swift` **[NEW]**
- Java: `apple/app-store-server-library-java` **[NEW]**
- Python: `apple/app-store-server-library-python` **[NEW]**
- Node.js: `apple/app-store-server-library-node` **[NEW]**

## Code Highlights

```swift
// Handle ONE_TIME_CHARGE notification
func handleNotification(_ payload: ResponseBodyV2DecodedPayload) {
    switch payload.notificationType {
    case .ONE_TIME_CHARGE:
        let txInfo = payload.data?.transactionInfo
        fulfillContent(for: txInfo?.transactionId)
    case .CONSUMPTION_REQUEST:
        let reason = payload.data?.consumptionRequestReason
        if reason == .FULFILLMENT_ISSUE {
            submitConsumptionInfo(refundPreference: .GRANT)
        } else {
            submitConsumptionInfo(refundPreference: .DECLINE)
        }
    default: break
    }
}

// Get full transaction history (v2)
// GET /inApps/v2/history/{transactionId}?excludeRevoked=false
```

## Takeaways
- Adopt `ONE_TIME_CHARGE` notifications to handle non-consumable purchases and refunds server-side in real time — previously this required polling Get Transaction History.
- Read `consumptionRequestReason` before deciding how to respond to `CONSUMPTION_REQUEST` — a `FULFILLMENT_ISSUE` warrants a `GRANT` preference, while `UNSATISFIED_WITH_PURCHASE` may warrant `DECLINE`.
- Switch from Get Transaction History v1 to v2 to get revoked/refunded transactions included automatically.
- Use the open-source App Store Server Library rather than hand-rolling JWT verification — it handles key rotation and payload decoding edge cases.

---
_Source: WWDC24 Session 10062 page (abstract, chapter summaries, code samples, and resource links)._
