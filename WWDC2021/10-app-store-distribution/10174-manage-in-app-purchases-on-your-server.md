# Manage In-App Purchases on Your Server
**WWDC21 · Session 10174** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10174/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
This session introduces the new server-side in-app purchase infrastructure that accompanies StoreKit 2. The centerpiece is a move from opaque unified app receipts to JWS (JSON Web Signature) signed transactions, plus two new App Store Server APIs (Subscription Status API and In-App Purchase History API) with JWT authentication. App Store Server Notifications v2 brings a new JWS-signed payload, new notification types, and substate fields. Sandbox testing gains several new capabilities: a separate sandbox notification URL, subscription renewal rate control, region changing, and purchase history clearing per tester account.

## Key Topics

**Signed Transactions (JWS Format)**
Transactions are now represented as three base64-encoded strings joined by `.` — header (algorithm + x5c certificate chain), payload (transaction JSON), signature. To read a transaction: base64-decode the payload. To verify authenticity: use the `alg` from the header and the certificate chain in `x5c` with any cryptographic library. No App Store server call is required for verification.

Transaction payload changes from the old receipt format:
- `type` field — content type: `Auto-Renewable Subscription`, `Non-Consumable`, `Consumable`, `Non-Renewing Subscription`
- `appAccountToken` — UUID supplied by the app at purchase time (StoreKit 2 `Product.purchase(options:)`), persisted by Apple and returned in every subsequent notification/API response
- `revocationDate` / `revocationReason` — renamed from `cancellation_date`/`cancellation_reason` to clarify that service should be revoked
- `offerType` + `offerIdentifier` — consolidate `isTrialPeriod`, `isIntroOfferPeriod`, `promotionalOfferIdentifier`, `offerCodeRefName`; `offerType` values: `1` = intro offer, `2` = subscription offer, `3` = offer code
- Date fields normalized to milliseconds since epoch only

**App Store Server APIs (NEW)**
All new APIs use JWT authentication (ES256, key from App Store Connect → Users and Access → Keys → In-App Purchase), key only the `originalTransactionId` (not a receipt), and return signed transactions.

1. **Subscription Status API** (`GET /inApps/v1/subscriptions/{originalTransactionId}`) — returns current status for all subscriptions in the same app, grouped by `subscriptionGroupIdentifier`. Per entry: `status` integer (1=active, 2=expired, 3=billing retry, 4=grace period, 5=revoked), `signedTransactionInfo`, `signedRenewalInfo`.

2. **In-App Purchase History API** (`GET /inApps/v1/history/{originalTransactionId}`) — paginated (20 per page); returns `signedTransactionInfo` for every transaction in the app. Paginate via `hasMore` + `revision` query parameter.

**JWT Authentication**
Generate a JWT signed with ES256 using a private key downloaded from App Store Connect. JWT payload: `issuer` (issuer ID from App Store Connect), `issuedAt`, `expiresAt` (max 1 hour gap), `audience: "appstoreconnect-v1"`, `nonce`, `bundleId`. Include `kid` (key ID) in the header.

**App Store Server Notifications v2 (NEW)**
The entire notification payload is JWS-signed. Payload always includes: `notificationType`, `subtype`, `version: "2"`, `environment`, `bundleId`, `appAppleId`, `bundleVersion`, `signedTransactionInfo`, `signedRenewalInfo`.

New notification types: `SUBSCRIBED`, `OFFER_REDEEMED`, `EXPIRED`, `GRACE_PERIOD_EXPIRED`, `PRICE_INCREASE`.
Deprecated from v1: `INITIAL_BUY`, `INTERACTIVE_RENEWAL`, `CANCEL`, `PRICE_INCREASE_CONSENT`.
New `subtype` field narrows notification to specific user actions — e.g., `SUBSCRIBED` subtypes: `INITIAL_BUY`, `RESUBSCRIBE`; `EXPIRED` subtypes: `VOLUNTARY`, `BILLING_RETRY`, `PRICE_INCREASE`.

Opt in to v2 per-URL (production and sandbox independently) via App Store Connect → App page → App Store Server Notifications section.

**Family Sharing Notifications Expansion**
Additional notification types now supported for family members in both v1 and v2 (e.g., `DID_RENEW`, `DID_CHANGE_RENEWAL_STATUS`, `SUBSCRIBED`, `EXPIRED`, `OFFER_REDEEMED`). `inAppOwnershipType` field (`PURCHASED` vs. `FAMILY_SHARED`) available in all signed transactions.

**Sandbox Enhancements**
- Separate sandbox notification URL in App Store Connect (keep production/sandbox notifications independent) **[NEW]**
- Choose sandbox notification version (v1 or v2) **[NEW]**
- Clear purchase history per sandbox tester account (enables re-purchasing without a new account) **[NEW]**
- Change App Store region per tester (test across 175 storefronts) **[NEW]**
- Adjust subscription renewal rate per tester (slow down to test upgrade/downgrade/cancel, speed up to simulate long-term subscribers) **[NEW]**
- `verifyReceipt` returns error for TestFlight receipts when user is no longer a TestFlight user (security enhancement) **[NEW]**

## APIs & Frameworks

- **App Store Server API** **[NEW]**
  - `GET /inApps/v1/subscriptions/{originalTransactionId}` — Subscription Status API
  - `GET /inApps/v1/history/{originalTransactionId}` — In-App Purchase History API
  - JWT authentication (ES256); private key from App Store Connect Keys tab
  - `signedTransactionInfo` (JWS) — decoded payload contains `type`, `appAccountToken`, `revocationDate`, `revocationReason`, `offerType`, `offerIdentifier`, ms-since-epoch dates
  - `signedRenewalInfo` (JWS) — renewal preferences, `offerType`, `offerIdentifier` for next renewal
- **App Store Server Notifications v2** **[NEW]**
  - New types: `SUBSCRIBED`, `OFFER_REDEEMED`, `EXPIRED`, `GRACE_PERIOD_EXPIRED`, `PRICE_INCREASE`
  - New `subtype` field on `SUBSCRIBED`, `DID_CHANGE_RENEWAL_STATUS`, `DID_CHANGE_RENEWAL_PREF`, `OFFER_REDEEMED`, `EXPIRED`, `PRICE_INCREASE`
  - Whole payload signed as JWS
  - `appAccountToken` in signed transaction payload
- **StoreKit** — `Product.purchase(options:)` with `appAccountToken` UUID **[NEW]**
- **verifyReceipt** — legacy server-to-server endpoint; still supported; migration path to JWS via `originalTransactionId`
- **App Store Connect** — in-app purchase key management, sandbox tester management, notification URL/version config
- **JSON Web Token (JWT)** / **JSON Web Signature (JWS)** — ES256 algorithm; CryptoKit or any crypto library for verification

## Code Highlights

Decode a signed transaction (server-side, any language):
```
# signedTransactionInfo is a JWS: header.payload.signature
parts = signedTransactionInfo.split('.')
payload_json = base64url_decode(parts[1])   # no App Store call needed
transaction = JSON.parse(payload_json)
# transaction.type, transaction.appAccountToken, transaction.offerType, etc.
```

JWT authentication header + payload for App Store Server API:
```json
// Header
{ "alg": "ES256", "kid": "<KEY_ID>", "typ": "JWT" }
// Payload
{
  "iss": "<ISSUER_ID>",
  "iat": 1625000000,
  "exp": 1625003600,
  "aud": "appstoreconnect-v1",
  "nonce": "<UUID>",
  "bid": "com.example.myapp"
}
```

Subscription Status API response shape:
```json
{
  "environment": "Production",
  "bundleId": "com.example.myapp",
  "data": [{
    "subscriptionGroupIdentifier": "12345678",
    "lastTransactions": [{
      "status": 1,
      "originalTransactionId": "...",
      "signedTransactionInfo": "...",
      "signedRenewalInfo": "..."
    }]
  }]
}
```

## Takeaways

- Migrate from `verifyReceipt` + unified receipts to the new App Store Server APIs: no receipt needed on the server — only the `originalTransactionId` — and signed transactions can be decoded client-side with a simple base64 decode.
- Store only the fields you need from a signed transaction and discard the JWS blob; call the Subscription Status or History API on demand using `originalTransactionId`.
- Enroll in App Store Server Notifications v2 to gain `SUBSCRIBED`, `OFFER_REDEEMED`, `EXPIRED`, substates, and `appAccountToken` — allowing immediate server-side user linkage without a separate verification call.
- Use the new sandbox controls (clear purchase history, region change, renewal rate) to comprehensively test subscription lifecycle scenarios without managing multiple test accounts.

---
_Source: WWDC21 Session 10174 page (abstract, full transcript, and resource links)._
