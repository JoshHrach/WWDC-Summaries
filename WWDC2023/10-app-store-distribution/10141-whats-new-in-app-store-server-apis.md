# What's New in App Store Server APIs
**WWDC23 · Session 10141** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10141/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
This session covers the latest updates to the App Store Server API and App Store Server Notifications V2, the two primary server-side APIs for managing in-app purchases. Three new capabilities are introduced: a dedicated endpoint for fetching a single transaction by ID, a new `status` field in notification payloads that immediately indicates the current subscription state, and an `onlyFailures` filter for the notification history endpoint to efficiently recover missed notifications.

The session also announces the deprecation of `verifyReceipt` and App Store Server Notifications V1, with migration guidance and timelines. A new open-source App Store Server Library is introduced to simplify server integration and migration.

## Key Topics

### New: Get Transaction Info Endpoint
- `GET /v1/transactions/{transactionId}` **[NEW]** — fetches signed transaction info for a single purchase by any `transactionId`.
- Supports all product types including finished consumables (which do not appear in Get Transaction History).
- Eliminates the need to page through Get Transaction History to find a specific transaction.
- Returns a `signedTransactionInfo` JWS string identical in format to other API responses.

### Transacting with Any transactionId
- Core endpoints (`Get Transaction History`, `Get All Subscription Statuses`, `Send Consumption Information`, etc.) now accept **any** `transactionId` as a path parameter — not just `originalTransactionId`.
- Existing calls using `originalTransactionId` continue to work unchanged.
- Removes the extra step of first looking up the `originalTransactionId` before calling core endpoints.

### New: `status` Field in App Store Server Notifications V2
- A new integer `status` field **[NEW]** is added to the `data` object of every auto-renewable subscription notification.
- Values correspond to the five subscription states: Active (1), Expired (2), Billing Retry (3), Billing Grace Period (4), Revoked (5).
- Eliminates the need to call Get All Subscription Statuses after receiving ambiguous notification types (e.g., `REFUND` for a subscription).
- Present in all V2 notifications for auto-renewable subscriptions.

### New: `onlyFailures` in Get Notification History
- `onlyFailures` boolean field **[NEW]** added to the `GET /v1/notifications/history` request body.
- When `true`, limits results to notifications that failed to reach the developer's server (including notifications currently in the retry process).
- Enables efficient recovery from both planned outages (known time window) and transient network failures (unknown time window).
- After a retry ultimately succeeds, the notification no longer appears in subsequent `onlyFailures` queries.
- The `sendAttempts` array in each notification entry shows individual send attempt results (up to 1 initial + 5 retries = 6 entries).

### Deprecations
- **`verifyReceipt` API** **[DEPRECATED]** — no longer receives feature updates; will be retired; migrate to App Store Server API.
- **App Store Server Notifications V1** **[DEPRECATED]** — migrate to V2; V1 notifications in retry may continue for up to 3 days after switching.
- **Migration steps from `verifyReceipt`**:
  1. Sign a JWT for your app (documented process).
  2. Save a `transactionId` for each user (can extract from existing receipts).
  3. Call App Store Server API endpoints with the `transactionId`.
- **Migration steps from Notifications V1 to V2**:
  1. Prepare server to parse JWS-signed V2 format.
  2. Switch preference in App Store Connect (can enable V2 in sandbox first).

### App Store Server Library (NEW)
- New open-source library for calling the App Store Server API and parsing V2 notifications.
- Simplifies JWT generation, signed data verification, and extracting `transactionId` from receipts.
- Available in sandbox and production; see companion session "Meet the App Store Server Library" (10143).

## APIs & Frameworks

- `App Store Server API` — server-side REST API for in-app purchase data
- `GET /v1/transactions/{transactionId}` **[NEW]** — Get Transaction Info endpoint; returns `signedTransactionInfo` for any transaction type
- `GET /v1/history/{transactionId}` — Get Transaction History; now accepts any `transactionId` **[updated]**
- `GET /v1/subscriptions/{transactionId}` — Get All Subscription Statuses; now accepts any `transactionId` **[updated]**
- `PUT /v1/transactions/consumption/{transactionId}` — Send Consumption Information; now accepts any `transactionId` **[updated]**
- `GET /v1/refund/lookup/{transactionId}` — Get Refund History; now accepts any `transactionId` **[updated]**
- `GET /v1/notifications/history` — Get Notification History; new `onlyFailures` request field **[updated]**
- `onlyFailures` request field **[NEW]** — filter Get Notification History to failed/retrying notifications only
- `App Store Server Notifications V2` — proactive event notifications for in-app purchases
- `status` field in `data` object **[NEW]** — integer indicating current subscription state (1–5) in every auto-renewable subscription notification
- `sendAttempts` array — per-notification send attempt results (up to 6 entries)
- `signedTransactionInfo` — JWS-signed transaction payload (existing format)
- `signedPayload` — JWS-signed notification payload (existing format)
- `verifyReceipt` API **[DEPRECATED]** — legacy receipt verification API
- `App Store Server Notifications V1` **[DEPRECATED]** — legacy notification format
- `App Store Server Library` **[NEW]** — open-source library for server integration (see session 10143)
- JWT (JSON Web Token) — authentication mechanism for App Store Server API requests
- `StoreKit` / `StoreKit 2` — both supported by App Store Server API and V2 Notifications

## Code Highlights
No client Swift code in this session. Server-side workflow:

```
// Get Transaction Info — single endpoint call to retrieve any transaction
GET https://api.storekit.itunes.apple.com/inApps/v1/transactions/{transactionId}

Authorization: Bearer <signed-jwt>

// Response
{
  "signedTransactionInfo": "<JWS-string>"
}

// Get Notification History with onlyFailures
POST https://api.storekit.itunes.apple.com/inApps/v1/notifications/history
{
  "onlyFailures": true,
  "startDate": 1680000000000,
  "endDate": 1682000000000
}
```

## Takeaways
- The new `GET /v1/transactions/{transactionId}` endpoint solves the long-standing problem of efficiently retrieving a specific transaction (especially finished consumables) without paging through history.
- The `status` field in V2 notifications makes subscription state immediately readable from the notification alone, eliminating a follow-up API call in ambiguous cases like `REFUND`.
- `onlyFailures` in notification history turns post-outage recovery from a manual scan into a targeted, efficient operation.
- `verifyReceipt` and App Store Server Notifications V1 are now deprecated — teams should begin migration to the App Store Server API and V2 Notifications; the App Store Server Library can accelerate this migration.

---
_Source: WWDC23 Session 10141 page (abstract, chapter summaries, transcript, and resource links)._
