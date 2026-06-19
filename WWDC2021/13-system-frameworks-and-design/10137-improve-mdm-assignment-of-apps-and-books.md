# Improve MDM Assignment of Apps and Books
**WWDC21 · Session 10137** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10137/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
The new version of the Apps and Books Management API (used by MDM vendors to assign apps and books purchased through Apple School Manager or Apple Business Manager) introduces two major enhancements: real-time server-to-MDM push notifications for assignment, asset count, and user state changes; and asynchronous processing that reduces a 25,000-request assignment operation to just 10 requests. The API is available today; the legacy API continues to be supported but mixing old and new endpoints for the same token is unsupported.

## Key Topics

**Real-Time Notifications**
MDM servers opt in to notifications per sToken via the `clientConfig` endpoint by providing a `notificationUrl` and a `notificationAuthToken` (sent as bearer token in the `Authorization` header of each POST). Three notification types:

1. `ASSET_MANAGEMENT` — fired when `associateAssets`, `disassociateAssets`, or `revokeAssets` events complete (async). Payload contains an `assignments` array with `adamId`, `pricingParam`, and `serialNumber` (or `clientUserId`), plus `eventId`, `result` (SUCCESS/FAILURE), and `type` (ASSOCIATE/DISASSOCIATE/REVOKE). Subscribe via `assetManagement` notification type in client config.

2. `ASSET_COUNT` — fired when asset counts change due to buys, transfers, or refunds. Payload contains `adamId`, `pricingParam`, and a `countDelta` (positive for buys/transfers received, negative for refunds/transfers sent). Subscribe via `assetCount`. Eliminates the need to poll the `getAssets` endpoint for count updates.

3. `USER_MANAGEMENT` / `USER_ASSOCIATED` — fired when registered users are created, updated, associated with a managed Apple ID, or retired. `USER_ASSOCIATED` specifically fires when a user accepts an invitation link to associate their personal Apple ID. Payloads contain `users` / `associatedUsers` arrays with `clientUserId`, `idHash`, `inviteCode`, and `status`.

Apple makes a best-effort delivery; MDM must return HTTP 2xx. On non-2xx or timeout, Apple retries a few times, then discards — MDM must fall back to polling when notifications are missed.

**Asynchronous Processing**
The new management endpoints (e.g., `POST /assets/associate`) accept:
- Up to 25 `adamId`/`pricingParam` pairs per request (dynamic limit in `serviceConfig`)
- Up to 1,000 `serialNumbers` or `clientUserIds` per request

The synchronous response returns only an `eventId` (UUID) and `tokenExpirationDate`. Actual assignment results arrive as `ASSET_MANAGEMENT` notifications. The `eventId` ties notifications back to the originating request. An `eventStatus` endpoint is available as a polling fallback. This reduces 250,000 assignments (25 apps × 10,000 devices) from a minimum of 25,000 legacy requests to just 10 new API requests.

**Sync API Changes**
All read (sync) endpoints use proper HTTP GET with versioned URIs. Bearer token auth (`Authorization: Bearer <sToken>`) is required on all new API requests. Sync responses include pagination fields (`currentPageIndex`, `size`, `totalPages`) and a `versionId` that changes on any write, useful for detecting stale pages during large traversals.

## APIs & Frameworks

- **Apps and Books Management API** (MDM server-to-Apple REST API) **[UPDATED]**
- `POST /clientConfig` — subscribe to notifications; set `notificationUrl`, `notificationAuthToken`, notification type list **[NEW]**
- `GET /assets` (asset sync endpoint) **[NEW]** — replaces legacy `getVPPAssetsSrv`; query params: `adamId`, `pricingParam`, `assignedOnly`, `facilitatorMemberId`; returns `assets[]` with `availableCount`, `assignedCount`, `totalCount`
- `POST /assets/associate` **[NEW]** — async assign assets to devices/users; body: `assets[]`, `serialNumbers[]` or `clientUserIds[]`; response: `eventId`
- `POST /assets/disassociate` **[NEW]** — async revoke assets from devices/users
- `GET /assets/events/{eventId}` — event status polling fallback **[NEW]**
- `GET /users` (user sync endpoint) **[NEW]** — query params: `clientUserId`, `facilitatorMemberId`; returns `users[]` with `status`, `inviteCode`, `idHash`
- `POST /users` — create registered users **[NEW]**
- `PATCH /users` — update/associate managed Apple IDs to registered users **[NEW]**
- Notification types (in `clientConfig.notificationTypes[]`):
  - `assetManagement` — assignment results **[NEW]**
  - `assetCount` — inventory changes **[NEW]**
  - `userManagement` — user create/update/retire **[NEW]**
  - `userAssociated` — Apple ID invitation accepted **[NEW]**
- `serviceConfig` endpoint — exposes dynamic limits (`maxAssetsPerRequest`, `maxDevicesPerRequest`) **[NEW]**
- **Apple School Manager** / **Apple Business Manager** — purchase/transfer origin (no API change)

## Code Highlights

Associate assets request body (up to 25 apps, 1,000 devices):
```json
{
    "assets": [
      { "adamId": "361309726", "pricingParam": "STDQ" }
    ],
    "serialNumbers": ["serial-1", "...", "serial-1000"]
}
```

Synchronous response (only an event ID):
```json
{
    "eventId": "92467a8e-8a50-4df9-9b30-f7ff4a99dea7",
    "tokenExpirationDate": "2021-07-06T14:12:10+0000",
    "uId": "2049025000431439"
}
```

Asset Management notification (arrives asynchronously):
```json
{
    "notification": {
        "assignments": [{ "adamId": "361309726", "pricingParam": "STDQ", "serialNumber": "serial-1" }],
        "eventId": "92467a8e-8a50-4df9-9b30-f7ff4a99dea7",
        "result": "SUCCESS",
        "type": "ASSOCIATE"
    },
    "notificationType": "ASSET_MANAGEMENT"
}
```

Asset Count notification (delta-based, no re-poll needed):
```json
{
    "notification": { "adamId": "408709785", "countDelta": 50, "pricingParam": "STDQ" },
    "notificationType": "ASSET_COUNT"
}
```

## Takeaways

- Adopt the new Apps and Books Management API now; it reduces large-scale assignment operations by orders of magnitude (e.g., 25,000 requests → 10 requests for 250,000 assignments) via async processing and batched endpoints.
- Subscribe to `ASSET_MANAGEMENT` notifications to drive MDM install commands in real time — send `InstallApplication` MDM commands only after receiving a successful assignment notification for each device.
- Use `ASSET_COUNT` notifications to keep asset inventory in sync without polling; use `USER_ASSOCIATED` to track when users have completed Apple ID association.
- When notifications are missed (non-2xx or Apple retries exhausted), fall back to polling only the endpoints known to be stale rather than a full sync.

---
_Source: WWDC21 Session 10137 page (abstract, full transcript, and code samples)._
