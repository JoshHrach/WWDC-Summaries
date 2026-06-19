# Expanding Automation with the App Store Connect API
**WWDC20 · Session 10004** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10004/)

_Platforms:_ App Store Connect REST API (platform-agnostic)

## Overview
The App Store Connect API received its largest expansion since launch: over 200 new endpoints covering two major capability areas. The **App Metadata API** provides full programmatic control of App Store presence — creating and editing versions, setting pricing schedules, uploading screenshots and app previews, and submitting for App Review. The **Power and Performance Metrics and Diagnostics API** exposes the same aggregate data that drives the Xcode Organizer's Power and Performance section, enabling automated performance monitoring pipelines.

An OpenAPI 3.0 specification file is now available for download, enabling code generation in virtually any language and integration with tools like Swagger UI.

## Key Topics

### App Metadata API — Full Version Lifecycle **[NEW]**

**Creating a version**: POST to `v1/appStoreVersions` with the target `platform` (iOS, macOS, tvOS), `versionString`, and a relationship to the App resource ID. The App ID is found by filtering `GET v1/apps` by `bundleId`.

**Pricing and Price Schedule**: Apps resource has a relationship to `AppPrices`. Each price has a `startDate` (null = currently active) and a relationship to a `PriceTier`. To set a complex price schedule (e.g., a temporary promotion), PATCH the app's `prices` relationship in a single atomic request using temporary IDs (prefixed with `${}` syntax) in the `included` array. All prices are created and linked atomically. Important: price and territory changes take effect immediately for already-released apps — they do not wait for the next version to clear App Review.

**App metadata resources**:
- `AppInfo` — app-level: App Store category, primary locale. Has localized variants (`AppInfoLocalization`: name, subtitle, privacy policy).
- `AppStoreVersionLocalization` — version-level, one per locale: screenshots, app previews, promotional text, keywords.
- `AppScreenshotSet` / `AppPreviewSet` — one per display type (e.g., `IPHONE_65`). Each contains up to 10 screenshots or 3 previews.

**Multi-part asset upload** (for screenshots, previews, App Review attachments, routing coverage files):
1. POST a reservation (`v1/appPreviews`, etc.) with filename and file size → receive upload operations.
2. Each operation specifies HTTP method, URL, byte offset, length, and headers. Upload each part (can be done in parallel, in any order; retry failures individually).
3. PATCH the asset with `uploaded: true` + an MD5 checksum of the source file → App Store Connect reassembles, validates integrity, dimensions, codec, etc.
4. Poll the asset's `state` attribute: `COMPLETE` (success) or `FAILED` (check `errors` array).

**Associating a build**: PATCH `v1/appStoreVersions/{id}/relationships/build` with the build ID. Builds are created by Xcode or Transporter, not via the API.

**Submitting for review**: POST to `v1/appStoreVersionSubmissions` with a relationship to the version. App Review details (`v1/appReviewDetails`) and attachments can be added before submission. Deleting the submission resource cancels review (before App Review picks it up).

**Limitations**: In-app purchase configuration, Game Center, and manual release after review still require the App Store Connect website.

### Power and Performance Metrics and Diagnostics API **[NEW]**
Exposed via the `perfPowerMetrics` relationship on both `Apps` and individual `Builds`. The response uses a custom media type (`Accept` header required). Returns structured data for the same metrics shown in Xcode Organizer: hang rate, memory, launch time, disk writes, battery consumption.

**Diagnostic Signatures** (disk writes): the `diagnosticSignatures` relationship on a `Build` returns a list of signatures — locations in the app's runtime responsible for disk writes — each with a `signature` string (call site identifier) and a `weight` (percentage contribution). Each signature has a `logs` relationship that returns detailed call stack data in the same custom media type.

### OpenAPI Specification **[NEW]**
A full OpenAPI 3.0 spec is downloadable from the API documentation page. Use with:
- Swagger UI for interactive browsing.
- Code generators (openapi-generator, swagger-codegen) for any language.
- IDE integrations for auto-complete and type checking.

## APIs & Frameworks

### App Store Connect REST API (new endpoints)
- `GET /v1/apps?filter[bundleId]={id}` — look up App resource by bundle ID
- `POST /v1/appStoreVersions` — create new version **[NEW]**
- `GET /v1/apps/{id}/prices?include=priceTier` — fetch current price schedule **[NEW]**
- `PATCH /v1/apps/{id}` with `included` — set complex price schedule atomically **[NEW]**
- `GET /v1/appStoreVersions/{id}/appStoreVersionLocalizations` — list localizations **[NEW]**
- `POST /v1/appPreviewSets` — create preview set for a localization and display type **[NEW]**
- `POST /v1/appPreviews` — reserve asset upload slot **[NEW]**
- `PUT <uploadOperation.url>` — upload asset part (standard HTTP, not v1 path)
- `PATCH /v1/appPreviews/{id}` — commit asset upload **[NEW]**
- `POST /v1/appReviewDetails` — add App Review contact info and notes **[NEW]**
- `POST /v1/appStoreVersionSubmissions` — submit version for App Review **[NEW]**
- `DELETE /v1/appStoreVersionSubmissions/{id}` — cancel submission **[NEW]**
- `GET /v1/apps/{id}/perfPowerMetrics` — aggregate performance metrics **[NEW]**
- `GET /v1/builds/{id}/perfPowerMetrics` — per-build performance metrics **[NEW]**
- `GET /v1/builds/{id}/diagnosticSignatures` — disk write signature list **[NEW]**
- `GET /v1/diagnosticSignatures/{id}/logs` — call stack data for a signature **[NEW]**

## Code Highlights

Atomic price schedule update (three prices with one PATCH):
```json
PATCH /v1/apps/{appId}
{
  "data": {
    "type": "apps",
    "id": "{appId}",
    "relationships": {
      "prices": {
        "data": [
          { "type": "appPrices", "id": "${new-price-1}" },
          { "type": "appPrices", "id": "${new-price-2}" },
          { "type": "appPrices", "id": "${new-price-3}" }
        ]
      }
    }
  },
  "included": [
    { "type": "appPrices", "id": "${new-price-1}", "attributes": { "startDate": null },
      "relationships": { "priceTier": { "data": { "type": "appPriceTiers", "id": "1" } } } },
    { "type": "appPrices", "id": "${new-price-2}", "attributes": { "startDate": "2020-06-29" },
      "relationships": { "priceTier": { "data": { "type": "appPriceTiers", "id": "0" } } } },
    { "type": "appPrices", "id": "${new-price-3}", "attributes": { "startDate": "2020-07-06" },
      "relationships": { "priceTier": { "data": { "type": "appPriceTiers", "id": "1" } } } }
  ]
}
```

Submitting for App Review:
```json
POST /v1/appStoreVersionSubmissions
{
  "data": {
    "type": "appStoreVersionSubmissions",
    "relationships": {
      "appStoreVersion": {
        "data": { "type": "appStoreVersions", "id": "{versionId}" }
      }
    }
  }
}
```

## Takeaways
- The 2020 App Store Connect API expansion adds 200+ endpoints covering the full version metadata lifecycle — from version creation through App Review submission.
- The multi-part asset upload system (reserve → upload parts → commit with MD5 → poll state) must support multiple upload operations even for currently-small files, as the number of parts may change.
- Pricing changes via the API are immediate for released apps; the temporary-ID (`${}`) syntax in `included` allows atomic multi-resource creation in a single PATCH.
- The `perfPowerMetrics` and `diagnosticSignatures` endpoints expose the same data as Xcode Organizer, enabling automated performance monitoring and alerting in CI/CD pipelines.
- Download the OpenAPI spec to bootstrap client code generation in any language.

---
_Source: WWDC20 Session 10004 page (abstract and transcript)._
