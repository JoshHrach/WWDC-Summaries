# What's New in the Apple Push Notification Service
**WWDC16 · Session 724** · [Watch](https://developer.apple.com/videos/play/wwdc2016/724/)

_Platforms:_ iOS 10, macOS Sierra 10.12, tvOS 10, watchOS 3

## Overview
This session by Apple Push Notification Service (APNs) engineer Mayur Mahajan covers two major topics: a review of the HTTP/2-based provider API introduced in 2015, and the new **Token-Based Authentication** for APNs announced at WWDC 2016. Token authentication replaces the need for client TLS certificates when sending push notifications from a provider server to APNs, eliminating the certificate provisioning, renewal, and revocation overhead that has historically been a pain point for push notification infrastructure.

The session is server-side focused — it covers how provider servers authenticate and communicate with APNs over the HTTP/2 API. No iOS/macOS client-side push APIs are changed; this is purely about the provider-to-APNs connection.

## Key Topics

### HTTP/2 Provider API Review (Introduced 2015)
The HTTP/2 provider API replaced the legacy binary protocol:
- Binary protocol over a persistent HTTP/2 connection; supports multiple concurrent streams over one connection.
- Payload size increased to **4 KB** (up from 256 bytes in the legacy protocol).
- **Immediate response**: HTTP 200 OK on success; HTTP 400 with JSON error body (e.g., `{"reason": "BadDeviceToken"}`) on failure.
- **Instant token feedback**: HTTP 410 response (with timestamp JSON payload) when a device token has been unregistered, replacing the legacy Feedback Service.
- **Simplified certificates**: a single APNs client certificate now covers app, Watch complication, and VoIP pushes in both development and production environments (previously required separate certificates per environment/type).

### Token-Based Authentication (New in 2016)
Token authentication is an alternative to TLS client certificates for authenticating provider servers to APNs:

**Key differences from certificate authentication:**
- Certificate auth: provider presents a client TLS certificate during the TLS handshake; all pushes on that connection are implicitly scoped to the app identified by the certificate.
- Token auth: provider establishes a TLS connection *without* a client certificate; instead, every push request includes a signed JSON Web Token (JWT) in the `Authorization` HTTP header.

**Setup:**
1. Provision a **token sign-in key** (APNs Auth Key) from Certificates, Identifiers & Profiles in the developer account.
2. Apple generates a public/private key pair; the private key (`.p8` file) is used to sign tokens. The public key is retained by Apple for validation.
3. The sign-in key does **not** expire; it can be revoked and replaced if compromised.

**Token structure (JSON Web Token / JWT):**
- Three URL-safe Base64-encoded sections separated by periods: `<header>.<claims>.<signature>`.
- **Header**: `{"alg": "ES256", "kid": "<key_identifier>"}` — algorithm (ES256 = ECDSA with P-256) and key ID.
- **Claims**: `{"iss": "<team_id>", "iat": <unix_timestamp>}` — issuer (10-character developer Team ID) and issued-at time (seconds since epoch).
- **Signature**: ECDSA-P256 signature of `<header>.<claims>` using the private key; encoded as Base64.

**Usage:**
- Each push request includes an `Authorization: bearer <token>` HTTP header.
- Also requires the `apns-topic` header (the app's bundle ID).
- APNs validates the token's signature, team ID, and age.
- Token must have been issued within the **last 1 hour**; if older, APNs returns HTTP 403 with `{"reason": "ExpiredProviderToken"}`.
- Recommendation: reuse the token for its full valid lifetime; do not generate a new token per request.
- If a token is invalid (wrong signature, bad format): HTTP 403 `{"reason": "InvalidProviderToken"}`.
- APNs does not close the connection on token errors.

**Advantages over certificate auth:**
- No certificate provisioning, renewal tracking, or revocation workflow.
- Tokens are generated programmatically; many JWT libraries exist for all major languages.
- One sign-in key can be used to push to any app in the team (the app is identified by `apns-topic`, not by the connection credential).
- Shipping later in 2016 (post-WWDC).

## APIs & Frameworks

This session covers the APNs provider-to-server HTTP/2 API and JWT token format. No iOS/macOS client SDK APIs are introduced.

- **APNs HTTP/2 Provider API** — binary HTTP/2 POST requests to `api.push.apple.com` (production) or `api.development.push.apple.com` (sandbox)
  - `POST /3/device/<device_token>` — send a push notification
  - Request headers: `apns-topic` (bundle ID), `apns-id` (UUID for deduplication), `apns-expiration`, `apns-priority`, `apns-collapse-id`, `Authorization: bearer <token>` [NEW for token auth]
  - Response: HTTP 200 (success); HTTP 400 (bad request, JSON error body); HTTP 403 (auth failure) [NEW]; HTTP 410 (token removed, with timestamp)
- **Token-Based Authentication** [NEW] — JWT-based APNs provider authentication
  - APNs Auth Key (`.p8` file) — ECDSA P-256 private key provisioned from developer account; does not expire
  - JWT format: `<base64url(header)>.<base64url(claims)>.<base64url(signature)>`
  - JWT header fields: `alg` (`"ES256"`), `kid` (10-char key identifier from developer account)
  - JWT claims fields: `iss` (10-char Team ID), `iat` (Unix timestamp, seconds since epoch)
  - Signing algorithm: **ES256** (ECDSA with P-256 curve and SHA-256)
  - Token validity window: issued-at must be within the last **3600 seconds** (1 hour)
  - `Authorization` HTTP header value: `bearer <signed_jwt>`
  - HTTP 403 error reasons: `"InvalidProviderToken"`, `"ExpiredProviderToken"`, `"MissingProviderToken"`
- **APNs client certificate** (existing, continued) — TLS mutual auth; single certificate covers app + complication + VoIP pushes in dev and production
- Push notification payload — JSON body, max **4 KB**; `aps` dictionary with `alert`, `badge`, `sound`, `content-available`, `mutable-content`, `category`, `thread-id`
- Device token — unique per app per device; obtained via `UIApplication.registerForRemoteNotifications()` on client; passed to provider via app code

## Code Highlights

JWT token construction (conceptual, server-side — language-agnostic):
```
Header (JSON):
{
  "alg": "ES256",
  "kid": "ABC1234DEF"   // 10-char key ID from developer account
}

Claims (JSON):
{
  "iss": "DEF123GHIJ",  // 10-char Team ID from developer account
  "iat": 1466980784     // current Unix timestamp (seconds since epoch)
}

Token = base64url(header) + "." + base64url(claims) + "." + base64url(ES256_sign(key, header + "." + claims))
```

HTTP/2 push request with token auth:
```
POST /3/device/<device_token> HTTP/2
Host: api.push.apple.com
Authorization: bearer <signed_jwt>
apns-topic: com.example.MyApp
Content-Type: application/json
Content-Length: 39

{"aps":{"alert":"Hello","badge":1}}
```

HTTP/2 response examples:
```
HTTP/2 200                           // Success
HTTP/2 403                           // Token auth failure
{"reason": "ExpiredProviderToken"}   //  → regenerate token (>1 hour old)
HTTP/2 410                           // Device token no longer active
{"timestamp": 1466980000}            //  → remove from your database
```

## Takeaways
- Token-based APNs authentication (new in 2016) eliminates the certificate lifecycle management burden: provision one non-expiring `.p8` key, generate short-lived JWTs programmatically, include as `Authorization: bearer` header on every push request.
- A single APNs Auth Key covers all apps in your development team; the target app is identified by the `apns-topic` header (bundle ID), not by the connection credential.
- Tokens are valid for up to one hour; reuse them for the full window — generating a new token per request is wasteful and unnecessary.
- The HTTP/2 provider API provides immediate, per-request success/failure feedback and instant 410 responses for stale device tokens, replacing the legacy binary protocol and Feedback Service. If you are still using the legacy binary protocol, migrate now.

---
_Source: WWDC16 Session 724 page (abstract, transcript, and resource links)._
