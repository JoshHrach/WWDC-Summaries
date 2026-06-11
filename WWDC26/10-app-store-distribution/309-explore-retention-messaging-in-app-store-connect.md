# Explore Retention Messaging in App Store Connect
**WWDC26 · Session 309** · [Watch](https://developer.apple.com/videos/play/wwdc2026/309/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, visionOS (any platform with Auto-Renewable Subscriptions)

## Overview
Retention Messaging is a new App Store feature that intercepts subscribers at the moment they initiate cancellation and presents a custom message — optionally paired with a retention offer — giving developers one last opportunity to preserve the relationship before a subscriber leaves for good. This session introduces both a configuration-driven path through App Store Connect and a real-time, server-driven path via the new Retention Messaging API.

The App Store Connect path requires no server infrastructure: you compose a message, attach optional offer media from the Asset Library, link a retention offer (a new offer type, `offerType: 5`), and the App Store delivers it automatically to every eligible subscriber who taps "Cancel." The real-time path adds a server-to-server callback, giving developers the power to personalise the message — or swap it for an alternate product or a promotional offer — on a per-subscriber basis at the moment of cancellation.

The session closes with a side-by-side comparison of both approaches so developers can select the right tier of investment for their app or game.

## Key Topics

### Introduction
Retention Messaging targets the critical cancellation moment. Subscribers who see a well-timed message or offer are more likely to stay, making this an important complement to existing win-back campaigns.

### Retention Messaging in App Store Connect (0:00–6:38)
- Configure retention messages directly in App Store Connect — no code required.
- Messages are composed with title, body, and optional image assets sourced from the **Asset Library** (the centralised media store introduced this year).
- An optional **retention offer** (a new subscription offer type) can be attached; eligible offer types include free trial, pay-as-you-go discount, and pay-up-front discount.
- Messages are scoped to a specific subscription product and locale.
- The system delivers the message automatically whenever a subscriber for that product initiates cancellation.

### Real-time Retention Messaging (6:38–11:46)
- Developers register a **webhook URL** with the Retention Messaging API (`PUT /realtime/url`).
- When a subscriber attempts to cancel, the App Store makes a **signed server-to-server request** to that URL containing `originalTransactionId`, `productId`, `userLocale`, `requestIdentifier`, `environment`, and `signedDate`.
- The server responds with one of three options:
  1. A pre-configured **message** (by `messageIdentifier`).
  2. An **alternate product** (a different subscription product ID + message).
  3. A **promotional offer** (using `promotionalOfferSignatureV2` + message).
- A **Sandbox performance-testing endpoint** (`POST /performanceTest`, `GET /performanceTest/result/{requestId}`) lets developers validate latency and response correctness before going live.
- Real-time Retention Messaging is an **interest-form feature**; developers apply via the provided link.

### Retention Messaging Comparison (11:46–end)
- App Store Connect path: zero server requirement, static per-product messages, suitable for most apps.
- Real-time path: full per-subscriber personalisation, supports alternate products and promotional offers, requires server infrastructure and API approval.

## APIs & Frameworks

### Retention Messaging API (NEW)
- Base URL: `https://api.storekit.apple.com/inApps/v1/messaging`
- **URL configuration**
  - `PUT /realtime/url` — register webhook endpoint
  - `GET /realtime/url` — retrieve current webhook URL
  - `DELETE /realtime/url` — remove webhook URL
- **Message configuration**
  - `PUT /message/{messageIdentifier}` — create or update a message
  - `DELETE /message/{messageIdentifier}`
  - `GET /message/list`
  - `PUT /default/{productId}/{locale}` — set default message for product + locale
  - `DELETE /default/{productId}/{locale}`
  - `GET /default/{productId}/{locale}`
- **Image configuration**
  - `PUT /image/{imageIdentifier}`
  - `DELETE /image/{imageIdentifier}`
  - `GET /image/list`
- **Sandbox performance testing** (Sandbox only)
  - `POST /performanceTest` — initiate a test
  - `GET /performanceTest/result/{requestId}` — retrieve results

### App Store Server Notifications / JWSTransaction
- **`offerType: 5`** **[NEW]** — identifies a retention offer in the signed transaction payload
- `offerIdentifier` — retention offer identifier string
- `offerDiscountType` — e.g. `"FREE_TRIAL"`
- `offerPeriod` — ISO 8601 duration, e.g. `"P3M"`

### App Store Connect Features
- **Retention Messaging configuration** **[NEW]** — per-product, per-locale message composer
- **Asset Library** **[NEW]** — centralised media store; images used in retention messages are sourced here
- **Retention Offers** **[NEW]** — new subscription offer type (offer type 5); supports free trial, pay-as-you-go, pay-up-front discount modes

### Real-time Request / Response Schema
- Request fields: `originalTransactionId`, `appAppleId`, `productId`, `userLocale`, `requestIdentifier`, `environment`, `signedDate`
- Response variants: `{ "message": { "messageIdentifier": "…" } }`, `{ "alternateProduct": { "messageIdentifier": "…", "productId": "…" } }`, `{ "promotionalOffer": { "messageIdentifier": "…", "promotionalOfferSignatureV2": "…" } }`

## Code Highlights

Signed transaction payload showing retention offer fields:
```json
{
  "offerType": 5,
  "offerIdentifier": "Yoga_2026_cancel_free_3m",
  "offerDiscountType": "FREE_TRIAL",
  "offerPeriod": "P3M"
}
```

Real-time webhook response — promotional offer variant:
```json
{
  "promotionalOffer": {
    "messageIdentifier": "80135e2b-ae15-4ec4-8c5c-9ecc8045c0dc",
    "promotionalOfferSignatureV2": "eyJhbGciOiJFUzI…"
  }
}
```

## Takeaways
- **Retention Messaging** lets developers present a custom message (and optionally a discount offer) at the exact moment a subscriber taps "Cancel" — no client-side code required for the App Store Connect path.
- A new **offer type 5** (retention offer) surfaces in `JWSTransaction` and can be configured in App Store Connect alongside existing introductory and promotional offers.
- The **Retention Messaging API** enables real-time personalisation including alternate product offers and promotional offer signatures; it requires server infrastructure and an interest-form registration.
- Start with the App Store Connect path for zero-overhead coverage, then graduate to real-time if subscriber-level personalisation justifies the server investment.

---
_Source: WWDC26 Session 309 page (abstract, chapter summaries, code samples, and resource links)._
