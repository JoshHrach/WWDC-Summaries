# Explore in-app purchase integration and migration
**WWDC22 · Session 10040** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10040/)

_Platforms:_ iOS 15+, iPadOS 15+, macOS Monterey 12+, tvOS 15+, watchOS 8+

## Overview
This session covers migrating server-side in-app purchase infrastructure from the legacy `verifyReceipt` endpoint to the modern App Store Server API and from App Store Server Notifications Version 1 to Version 2. The App Store Server API returns signed JSON Web Signature (JWS) transactions that can be cryptographically verified, eliminating the need to trust a decoded receipt. Version 2 notifications use the same JWS format and cover the complete subscription lifecycle with a new `subtype` field that provides granular context for every state transition.

The session is split into two parts: Gabriel Ting explains how to authenticate API requests with JWTs, verify JWS certificate chains, and map `verifyReceipt` use cases to App Store Server API equivalents; Alex Baker explains how to configure, receive, validate, and recover from missed Version 2 notifications.

## Key Topics

**App Store Server API overview** — Five endpoints use `originalTransactionId` as a path parameter: Get Transaction History, Get All Subscription Statuses, Extend Subscription Renewal Date, Send Consumption Information, and Look Up Order ID (uses `orderId` for customer-support flows). Usable with both Original StoreKit and StoreKit 2; no migration to StoreKit 2 is required.

**Signing JWTs for API requests** — Every request requires an `Authorization: Bearer <JWT>` header. The JWT header contains `alg` (ES256), `kid` (private key ID from App Store Connect), and `x5c`. The payload contains `iss` (issuer ID), `iat`, `exp`, `aud` (`appstoreconnect-v1`), and `bid` (bundle ID). Sign with the PKCS#8 private key downloaded from App Store Connect.

**Verifying JWS signed transactions** — Transactions arrive as JWS strings with a base64-encoded header, payload, and ES256 signature. The `x5c` header claim contains an Apple-issued certificate chain (root → WWDR intermediate → leaf). Verify the chain against the Apple root CA obtained from `apple.com/certificateauthority`; then use the leaf certificate's public key to verify the JWS signature.

**Migrating from verifyReceipt** — Three common patterns: (1) Get latest subscription status via Get All Subscription Statuses instead of parsing `expiration_intent` et al. from a decoded receipt; (2) Get full transaction history via Get Transaction History (paginated, filterable, sortable) instead of `in_app` / `latest_receipt_info` arrays; (3) Use `applicationUsername` in Original StoreKit to populate `appAccountToken` in signed transactions and notifications.

**App Store Server Notifications V2 setup** — Configure endpoint URL per environment (production / sandbox) in App Store Connect. Validate HTTPS certificate and Apple IP allowlist before testing. Use the new "Request a Test Notification" endpoint to trigger a test; check delivery status with the new "Get Test Notification Status" endpoint (returns `firstSendAttemptResult` values like `SSL_ISSUE`).

**Notification validation** — Extract `signedPayload` from the JSON body, then apply the same JWS verification used for signed transactions. Verify `appAppleId`, `bundleId`, and `environment` fields to prevent replay attacks. Respond with HTTP 200 promptly; do heavy processing asynchronously to avoid retry storms.

**Subscription lifecycle coverage** — Version 2 notification types include `SUBSCRIBED`, `DID_RENEW`, `EXPIRED`, `DID_CHANGE_RENEWAL_STATUS`, `OFFER_REDEEMED`, `REFUND`, `REVOKE`, and more. The new `subtype` field disambiguates within a type (e.g., `INITIAL_BUY`, `BILLING_RECOVERY`, `AUTO_RENEW_DISABLED`, `VOLUNTARY`, `PRICE_INCREASE`).

**Recovering from missed notifications** — New Get Notification History endpoint provides up to 6 months of sent notifications, filterable by time range, notification type, and `originalTransactionId`. Use `signedDate` in notifications to detect retries vs. new deliveries; use `notificationUUID` for deduplication. Retry schedule: 1 hour → 12 hours → 24 hours → 48 hours → 72 hours.

## APIs & Frameworks

### App Store Server API (REST)
- `GET /inApps/v1/transactions/{originalTransactionId}` — Get Transaction History
- `GET /inApps/v1/subscriptions/{originalTransactionId}` — Get All Subscription Statuses
- `PUT /inApps/v1/subscriptions/extend/{originalTransactionId}` — Extend Subscription Renewal Date
- `POST /inApps/v1/notifications/test` **[NEW]** — Request a Test Notification
- `GET /inApps/v1/notifications/test/{testNotificationToken}` **[NEW]** — Get Test Notification Status
- `POST /inApps/v2/history/{originalTransactionId}` — Get Notification History **[NEW]**
- `GET /lookup/v1/orders/{orderId}` — Look Up Order ID
- `firstSendAttemptResult` field values: `SUCCESS`, `SSL_ISSUE`, `INVALID_RESPONSE`, `OTHER`, etc. **[NEW]**

### App Store Server Notifications V2
- `signedPayload` — top-level JWS field in notification POST body
- `notificationType` — `SUBSCRIBED`, `DID_RENEW`, `EXPIRED`, `DID_CHANGE_RENEWAL_STATUS`, `OFFER_REDEEMED`, `REFUND`, `REVOKE`, `CONSUMPTION_REQUEST`, `PRICE_INCREASE`
- `subtype` **[NEW]** — `INITIAL_BUY`, `RESUBSCRIBE`, `DOWNGRADE`, `UPGRADE`, `AUTO_RENEW_ENABLED`, `AUTO_RENEW_DISABLED`, `VOLUNTARY`, `BILLING_RETRY`, `PRICE_INCREASE`, `GRACE_PERIOD`, `BILLING_RECOVERY`, `PENDING`, `ACCEPTED`
- `notificationUUID` — unique per notification; same UUID on retries (use for deduplication)
- `signedDate` — timestamp when notification was created (use to detect retries)
- `signedTransactionInfo` — JWS-encoded signed transaction
- `signedRenewalInfo` — JWS-encoded signed renewal info

### JWS / JWT
- `x5c` — certificate chain header claim in both API JWTs and signed transactions
- `originalTransactionId` — top-level field in decoded JWS transaction payload
- `appAccountToken` — UUID linking transaction to developer's user account (original StoreKit uses `applicationUsername`)

### StoreKit 2 (client-side reference)
- `Transaction.originalID` — maps to `originalTransactionId` for App Store Server API calls

## Code Highlights

No inline code samples were included in this session. The key server-side pseudopattern for JWT signing:
```
private_key = load_pem_key(key_id)
jwt = sign(header={alg:"ES256", kid:key_id}, 
           payload={iss:issuer_id, iat:now, exp:now+1800, aud:"appstoreconnect-v1", bid:bundle_id},
           key=private_key)
curl -H "Authorization: Bearer ${jwt}" \
  https://api.storekit.itunes.apple.com/inApps/v1/subscriptions/${originalTransactionId}
```

OpenSSL certificate chain verification:
```
openssl verify -trusted AppleRootCA.cer -untrusted WWDR.cer leaf.cer
```

## Takeaways
- The App Store Server API and Version 2 Notifications work independently of StoreKit version — you can migrate your server without requiring client-side StoreKit 2 adoption.
- Always verify JWS certificate chains against the Apple root CA and check `appAppleId`, `bundleId`, and `environment` on every notification to prevent replay attacks.
- Respond to notifications with HTTP 200 immediately and process asynchronously; slow responses cause retries, creating duplicate delivery.
- The new Get Notification History endpoint (6-month window, filterable by time, type, and `originalTransactionId`) is the primary recovery tool after server outages.

---
_Source: WWDC22 Session 10040 page (abstract, chapter summaries, and resource links)._
