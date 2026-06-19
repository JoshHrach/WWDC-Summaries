# What's New in App Store Connect
**WWDC22 · Session 10043** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10043/)

_Platforms:_ iOS 16, iPadOS 16

## Overview
This session reviews the major changes to App Store Connect in 2022: the enhanced App Store submission experience (group submissions, submit without a new binary, dedicated App Review page), its availability on the iOS/iPadOS App Store Connect app, and the App Store Connect API 2.0 release — a 60% expansion of available resources covering in-app purchases, subscriptions, customer reviews, and app hang diagnostics. The session also announces the deprecation of the legacy XML feed in favor of the REST API.

## Key Topics

### Enhanced App Store Submission Experience
Review submissions are now the unit of work submitted to App Review. Key characteristics:
- **Grouped submissions** — multiple review items (app version, in-app events, custom product pages, product page optimization tests) are submitted together in one review submission
- **Submit without a new app version** — after the first version of an app is approved, subsequent in-app events, custom product pages, and page optimization tests can be submitted to App Review without attaching a new binary
- **One in-progress submission per platform** — iOS, macOS, tvOS, and watchOS each have their own submission queue
- **All-or-nothing acceptance** — no items in a submission are approved until all items are accepted; rejected items can be edited and resubmitted, or removed from the submission to unblock the rest

The dedicated **App Review** page in App Store Connect (left nav) provides a single view to create, manage, and track all review submissions, view rejection reasons, and communicate with App Review.

### Enhanced Submission on iOS and iPadOS App
The App Store Connect iOS/iPadOS app (updated WWDC22 week) adds support for:
- Submitting Ready for Review submissions to App Review with one tap
- Tracking review submission status
- Opt-in push notifications for status changes
- Removing items from a submission, viewing rejection reasons, and replying to App Review

### App Store Connect API 2.0
A major milestone release expanding API resources by approximately 60%, shipping summer 2022. Key additions:

**In-App Purchases and Subscriptions**
- Subscriptions are now a separate resource type
- Full CRUD for in-app purchases and auto-renewable subscriptions
- Manage pricing, submit for App Review, create promotional offers and promo codes

**Customer Reviews**
- Fetch customer reviews for an app
- Post developer responses to reviews
- Enables custom workflows around customer feedback

**Power and Performance — App Hang Diagnostics**
- New `hangDiagnostics` diagnostic type added to the existing diagnostic signatures resource
- Access detailed stack traces for hang signatures via the `diagnosticLogs` relationship
- Previously, only hang rate metrics were accessible via the API; now individual hang signatures with stack traces are available

**Legacy XML Feed Deprecation**
The XML feed will be decommissioned in fall 2022. All integrations should migrate to the App Store Connect API.

## APIs & Frameworks

**App Store Connect API 2.0** (REST, no SDK runtime APIs)

_Submission_
- `GET /v1/apps/{id}/appStoreVersions` — list versions (existing)
- `POST /v1/appStoreVersionSubmissions` — submit a version for review (existing)
- `GET /v1/appStoreVersions/{id}/appReviewAttachments` — manage review attachments

_In-App Purchases and Subscriptions (new in 2.0)_
- `POST /v1/inAppPurchases` **[NEW]** — create an in-app purchase
- `PATCH /v1/inAppPurchases/{id}` **[NEW]** — update an in-app purchase
- `DELETE /v1/inAppPurchases/{id}` **[NEW]** — delete an in-app purchase
- `POST /v1/subscriptions` **[NEW]** — create an auto-renewable subscription
- `POST /v1/subscriptionPricePoints` **[NEW]** — manage subscription pricing
- `POST /v1/promotionalOffers` **[NEW]** — create promotional offers
- `POST /v1/subscriptionPromoCodes` **[NEW]** — generate promo codes

_Customer Reviews (new in 2.0)_
- `GET /v1/apps/{id}/customerReviews` **[NEW]** — fetch customer reviews
- `POST /v1/customerReviewResponses` **[NEW]** — post a developer response

_Power and Performance — Hang Diagnostics (new in 2.0)_
- `GET /v1/apps/{id}/diagnosticSignatures?filter[diagnosticType]=HANG` **[NEW]** — list hang diagnostic signatures
- `GET /v1/diagnosticSignatures/{id}/logs` **[NEW]** — fetch stack traces for a hang signature

## Code Highlights

No Swift/Objective-C runtime code in this session. All new functionality is App Store Connect web UI and REST API. Example REST call to fetch hang diagnostic signatures:

```bash
# Fetch hang diagnostic signatures for an app
GET https://api.appstoreconnect.apple.com/v1/apps/{appId}/diagnosticSignatures
  ?filter[diagnosticType]=HANG

# Fetch stack trace logs for a specific signature
GET https://api.appstoreconnect.apple.com/v1/diagnosticSignatures/{signatureId}/logs
```

Example REST call to create an in-app purchase:
```bash
POST https://api.appstoreconnect.apple.com/v1/inAppPurchases
Content-Type: application/json

{
  "data": {
    "type": "inAppPurchases",
    "attributes": {
      "name": "Premium Upgrade",
      "productId": "com.example.app.premium",
      "inAppPurchaseType": "NON_CONSUMABLE"
    },
    "relationships": {
      "app": { "data": { "type": "apps", "id": "1234567890" } }
    }
  }
}
```

## Takeaways
- The enhanced submission experience lets you group in-app events, custom product pages, and optimization tests into a single App Review submission — and submit them without a new app binary once an initial version is approved.
- The App Store Connect iOS/iPadOS app now supports the full review submission workflow, enabling on-the-go submission, tracking, and communication with App Review.
- API 2.0 adds comprehensive in-app purchase/subscription CRUD, customer review fetching and responding, and hang diagnostic signatures with stack traces — a 60% resource expansion.
- The legacy XML feed is being decommissioned fall 2022; migrate all automation to the App Store Connect REST API.

---
_Source: WWDC22 Session 10043 page (abstract and transcript)._
