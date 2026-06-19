# Architecting for Subscriptions
**WWDC20 · Session 10671** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10671/)

_Platforms:_ iOS, macOS, tvOS, watchOS (server-side architecture)

## Overview
Subscription entitlement is far more complex than a simple active/inactive binary. This session introduces a structured approach to building a server-side entitlement engine that accurately determines what service a subscriber should receive at any moment in their lifecycle — including nuanced billing states like Billing Retry, Grace Period, voluntary cancellation, upgrades, and win-back scenarios.

The session models the subscriber journey as a set of states and substates derived from App Store receipt fields, then shows how to synthesize that data into a simplified entitlement payload (expressed as a numeric code) that drives client-side experience decisions. Sample code in Node.js accompanies the session. The architecture progressively scales from a stateless receipt-processing baseline up to a fully event-driven server using App Store Server Notifications with persistent storage.

## Key Topics
- **Subscriber journey complexity** — Subscriptions produce many more states than active vs. expired: voluntary cancel, involuntary churn (Billing Retry), Grace Period recovery, upgrade/downgrade/crossgrade, win-back, and more. Each state carries messaging and access implications.
- **State and substate model** — States are derived by combining receipt fields (`expires_date`, `auto_renew_status`, `expiration_intent`, `is_in_billing_retry_period`, `is_in_grace_period`, etc.); substates identify the specific product context (free trial, promotional offer, etc.).
- **Entitlement engine** — A server-side component that: (1) validates the receipt via `verifyReceipt`; (2) synthesizes fields from `latest_receipt_info` and `pending_renewal_info`; (3) computes an entitlement code (positive = access granted, negative = no access); (4) attaches subscriber-specific messaging and offers; (5) stores the result and/or returns it to the client.
- **Entitlement code** — A numeric value combining state (integer) and substate (decimal) that uniquely identifies a subscriber cohort; drives conditional logic for retention offers, billing update prompts, win-back campaigns, and upgrade suggestions.
- **App Store Server Notifications** — Real-time server-to-server notifications for events (`DID_RENEW`, `CANCEL`, `DID_FAIL_TO_RENEW`, `DID_RECOVER`, initial buy, etc.) that trigger the entitlement engine without requiring the client to send a receipt.
- **Progressive architecture** — Three phases: (1) stateless (device sends receipt per request, no storage); (2) add persistent storage (enables web/cross-platform); (3) add server notification endpoint (event-driven, near-real-time accuracy). Each phase degrades gracefully to the prior one as a fallback.
- **Grace Period vs. Billing Retry** — Grace Period (up to 16 days): Apple continues attempting renewal AND the subscriber retains access; billing date continuity preserved upon recovery. Billing Retry: no access granted; subscriber must update payment to recover.
- **Subscription offers and eligibility** — The App Store determines free trial eligibility; developers control promotional/win-back offer eligibility based on their own business logic applied inside the entitlement engine.
- **Security** — Entitlement payload should be signed (e.g., JSON Web Signature) before delivery to the client to prove server provenance.

## APIs & Frameworks

This session is primarily a server-side architecture guide. No new client-side Swift/Objective-C APIs are introduced. Relevant server-side and StoreKit constructs:

### App Store Receipts / StoreKit (server-side)
- **`verifyReceipt` endpoint** — App Store server endpoint; validates receipt and returns `latest_receipt_info` array and `pending_renewal_info` array
- **`latest_receipt_info`** — Array of in-app purchase transactions; key fields: `product_id`, `expires_date`, `transaction_id`, `original_transaction_id`, `is_trial_period`, `is_in_intro_offer_period`
- **`pending_renewal_info`** — Array of upcoming renewal records; key fields: `auto_renew_status`, `is_in_billing_retry_period`, `expiration_intent`, `grace_period_expires_date`, `auto_renew_product_id`
- **App Store Server Notifications** — Real-time POST to developer's server; `notification_type` values: `INITIAL_BUY`, `DID_RENEW`, `CANCEL`, `DID_FAIL_TO_RENEW`, `DID_RECOVER`, `DID_CHANGE_RENEWAL_STATUS`, `DID_CHANGE_RENEWAL_PREF`, `PRICE_INCREASE_CONSENT`, `REFUND`
- **Billing Retry** — `is_in_billing_retry_period = "1"` in `pending_renewal_info`; Apple retrying payment; no service access
- **Grace Period** — `grace_period_expires_date` in `pending_renewal_info`; access continues during recovery window (up to 16 days)

## Code Highlights

Entitlement code assignment (conceptual Node.js pseudocode from the session):
```js
// State values (positive = access, negative = no access)
const STATE = { ACTIVE: 1, GRACE: 2, BILLING_RETRY: -1, EXPIRED: -2, REFUNDED: -3 };
// Substate decimals distinguish free trial, offer, standard
const SUBSTATE = { STANDARD: 0.0, FREE_TRIAL: 0.1, PROMO_OFFER: 0.2 };

const entitlementCode = STATE.ACTIVE + SUBSTATE.FREE_TRIAL; // e.g., 1.1

// Retention logic: active subscriber who disabled auto-renew during trial
if (entitlementCode === 1.1 && autoRenewStatus === 0) {
    response.offer = buildRetentionOffer(productId);
}
// Grace Period: unlock access + prompt billing update
if (entitlementCode === 2.0) {
    response.accessGranted = true;
    response.message = "Update your payment method to keep your subscription.";
}
```

Progressive server architecture phases:
```
Phase 1: Device → [send receipt] → Entitlement Engine → response
Phase 2: + Persistent Storage → Engine reads/writes DB → cross-platform support
Phase 3: + App Store Server Notifications endpoint → real-time event-driven updates
```

## Takeaways
- Accurate entitlement requires inspecting multiple receipt fields together — expiration date alone is insufficient; `auto_renew_status`, `is_in_billing_retry_period`, `grace_period_expires_date`, and `expiration_intent` all change what a subscriber should experience.
- Model subscribers as cohorts via a numeric entitlement code (state + substate) so the same engine logic handles Grace Period, Billing Retry, voluntary cancel, and win-back as distinct cases with distinct messaging and offers.
- Build the entitlement engine in three progressive phases — stateless, then with storage, then with App Store Server Notifications — so each layer fails gracefully to the previous one.
- Fix entitlement bugs server-side without requiring an app update: the engine processes receipts on the server, so logic corrections deploy immediately to all inbound requests.
- Sign entitlement payloads (e.g., JSON Web Signature) before sending to the client to prevent tampering.

---
_Source: WWDC20 Session 10671 page (abstract, transcript, and resource links)._
