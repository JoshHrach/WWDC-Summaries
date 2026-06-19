# Dive into App Store Server APIs for In-App Purchase
**WWDC25 · Session 249** · [Watch](https://developer.apple.com/videos/play/wwdc2025/249/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, visionOS, watchOS (server-side APIs)

## Overview
This session covers the latest updates to the App Store Server API, App Store Server Notifications, and the App Store Server Library, organized around three server responsibilities: managing In-App Purchases (associating transaction data with customer accounts), signing requests (generating promotional offer and introductory offer signatures), and participating in refund decisions (consuming the new Send Consumption Information V2 endpoint).

New identifiers (`appTransactionId`, `offerPeriod`), a new endpoint to set `appAccountToken` server-side, a new unified JWS signature format for all signing use cases, and a greatly simplified Send Consumption Information V2 endpoint with prorated refund support are the major additions.

## Key Topics

### Transaction Identifiers
- **`transactionId`** — unique per purchase event (renewal, restore, purchase).
- **`originalTransactionId`** — stable across subscription renewals; key identifier for subscription lifecycle.
- **`appAccountToken`** — UUID set by the developer (via StoreKit or the new server endpoint) to associate App Store transactions with an internal account.
  - **[NEW]** `appAccountToken` is now also included in `JWSRenewalInfo`, making it easier to link renewals to customer accounts.
  - **[NEW] Set App Account Token endpoint** — allows servers to set or update `appAccountToken` for all product types (consumables, non-consumables, non-renewing subscriptions, and auto-renewable subscriptions), including for purchases made outside the app (offer code redemptions, App Store promoted purchases). Carries over to future renewals/upgrades.
- **`appTransactionId`** — **[NEW]** a globally unique, static identifier per Apple Account per app (present across all App Store transaction objects: `AppTransaction`, `JWSTransaction`, `JWSRenewalInfo`). Works with Family Sharing. Stable across redownloads, refunds, repurchases, storefront changes.
  - Recommended as the primary one-stop identifier for associating customer accounts with all App Store transactions.
  - **[NEW] Get App Transaction Info endpoint** — fetch `AppTransaction` data server-side without relying on a device. Accepts `originalTransactionId`, `transactionId`, or `appTransactionId` in path. (Available later in 2025.)
- **[NEW]** `offerPeriod` field in `JWSTransaction` and `JWSRenewalInfo` — ISO 8601 duration of the redeemed offer.

### Unified JWS Signature Format
- All signing use cases now use the **JWS (JSON Web Signature)** format. **[NEW]**
- **Promotional offer signatures** — now generated with `PromotionalOfferV2SignatureCreator` (App Store Server Library); simpler than previous format (fewer inputs; accepts any `transactionId` including `appTransactionId`). **[NEW]**
- **JWS introductory offer signature** — **[NEW]** enables custom per-transaction, per-user introductory offer eligibility.
- Advanced Commerce API also accepts signed JWS in-app requests.

### Send Consumption Information V2
- **[NEW]** Replaces the deprecated V1 endpoint (V1 still accepts requests but is deprecated).
- Supports **all product types** — consumables, non-consumables, non-renewing subscriptions, auto-renewable subscriptions. (V1 only supported consumables and auto-renewables.)
- Reduced from 12 to **5 input fields** (3 mandatory, 2 optional):
  - `customerConsented` (mandatory) — must be `true`; if `false`, request is rejected.
  - `sampleContentProvided` (mandatory).
  - `deliveryStatus` (mandatory) — `DELIVERED` or an `UNDELIVERED` variant.
  - `refundPreference` (optional) — `GRANT_FULL`, `GRANT_PRORATED` **[NEW]**, or `DECLINE`.
  - `consumptionPercentage` (optional, required when using `GRANT_PRORATED`) — in millipercent (e.g., 25000 = 25%).
- **Prorated refund preference** — new ability to request a partial refund proportional to consumption. Apple calculates subscription prorations automatically; developer-provided `consumptionPercentage` is required for consumables, non-consumables, and non-renewing subscriptions.

### REFUND Notification Updates
- **[NEW]** `refundPercentage` field — indicates how much of a refund was granted (e.g., 75 = 75% refund).
- **[NEW]** `revocationType` field — `REFUND_FULL`, `REFUND_PRORATED`, or `FAMILY_REVOKE`.
- For `REFUND_PRORATED` on consumables: revoke proportional content (e.g., reduce virtual currency balance by the refunded percentage).

## APIs & Frameworks

### App Store Server API (server-side REST)
- `Set App Account Token` endpoint — **[NEW]** `PATCH` to set/update `appAccountToken` for a transaction.
- `Get App Transaction Info` endpoint — **[NEW]** `GET` by any transaction identifier; returns `JWS signedAppTransactionInfo`.
- `Send Consumption Information V2` endpoint — **[NEW]** simplified refund participation.
- Existing: `Get Transaction History`, `Get All Subscription Statuses`, `Get Refund History`.

### App Store Server Library (open source)
- `PromotionalOfferV2SignatureCreator` — **[NEW]** generates JWS promotional offer signatures.
- `AppTransaction` decoding, `JWSTransaction` / `JWSRenewalInfo` decoding.
- Available in Java, Swift, Python, Node.js, PHP on GitHub.

### App Store Server Notifications
- `CONSUMPTION_REQUEST` — triggers server-side refund participation flow.
- `REFUND` / `REFUND_DECLINED` — outcome notifications; `REFUND` now includes `refundPercentage` and `revocationType`.

### StoreKit (client-side)
- `appAccountToken` — set during purchase.
- `AppTransaction` — includes `appTransactionId` for linking app download to purchases.

## Code Highlights
No code samples were shown in this session. Java example for `PromotionalOfferV2SignatureCreator` was demonstrated live; refer to the App Store Server Library GitHub repository.

## Takeaways
- Adopt `appTransactionId` as your primary cross-session customer identifier — it is stable, works with Family Sharing, and is present in every App Store transaction object.
- Use the new Set App Account Token endpoint to retrofit `appAccountToken` for purchases that happened outside of StoreKit (offer code redemptions, promoted purchases).
- Migrate promotional offer signing to the new JWS format — fewer inputs, unified approach, and the old format remains available during transition.
- Implement Send Consumption Information V2 for all product types to participate in refund decisions, and consider providing `GRANT_PRORATED` preference where partial consumption is meaningful.

---
_Source: WWDC25 Session 249 page (abstract, chapter summaries, and full transcript)._
