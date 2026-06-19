# Meet AdAttributionKit
**WWDC24 · Session 10060** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10060/)

_Platforms:_ iOS 18, iPadOS 18

## Overview
AdAttributionKit is Apple's new privacy-preserving ad attribution framework for iOS and iPadOS, built on the foundations of SKAdNetwork. It uses crowd anonymity to send conversion data to ad networks while preserving user privacy — the higher the crowd size (number of installs from the same campaign), the more conversion data is returned. AdAttributionKit is fully interoperable with SKAdNetwork, so ad networks already registered for SKAdNetwork require no additional enrollment.

The session covers the complete attribution lifecycle: fetching JWS-formatted ad impressions, displaying ads (custom click, view-through, SKOverlay, SKStoreProductViewController), updating postback conversion values, the new re-engagement conversion type, and a developer mode for faster testing.

## Key Topics

**Ad Impressions (Fetching Ads)**
- Ads are delivered as compact JWS (JSON Web Signature) strings from the ad network
- Key JWS payload fields: `advertised-item-identifier` (advertised app ID), `publisher-item-identifier` (publisher app ID), `source-identifier` (4-digit campaign integer, same as SKAdNetwork 4.0)
- `AppImpression` — initialized with the JWS string; valid for 15 minutes; create a new instance after expiry

**Displaying Ads**
Three supported ad display methods:
1. **Custom click ads** — custom `UIView` with a `UIEventAttributionView` subview overlaid on top; call `appImpression.handleTap()` when tapped; AdAttributionKit navigates user to a marketplace; ad interaction type = "click"
2. **View-through ads** — any custom presentation; call `appImpression.beginView()` when content appears and `appImpression.endView()` when it disappears; minimum 2 seconds between begin/end; ad interaction type = "view"
3. **SKOverlay / SKStoreProductViewController** — same integration as SKAdNetwork; set `appImpression` on `AppConfiguration.appImpression`; both can trigger "click" or "view" ad interaction types

**Postbacks**
- Postbacks are the output signals sent to ad networks after a conversion
- Delivered as JWS signed payload + unsigned fields
- Key postback fields: `conversion-type` (download, redownload, re-engagement), `marketplace-identifier`, `publisher-item-identifier`, `source-identifier` (2–4 digits, crowd-anonymity-controlled), `ad-interaction-type` (click or view), `conversion-value` (fine: 0–63; coarse: low/medium/high)
- Conversion values updated via `Postback.updateConversionValue(_:)` using a `PostbackUpdate` struct
- `PostbackUpdate` fields: `fineConversionValue` (0–63), `lockPostback` (freeze and schedule the postback immediately), `coarseConversionValue` (low/medium/high), `conversionTypes` (filter which postbacks to update)
- Locking does not cause instant transmission — delayed by design for privacy

**Re-engagement**
- New conversion type in AdAttributionKit; brings lapsed users back to an already-installed app
- Opt-in via `eligible-for-re-engagement` field in the JWS impression payload
- When tapped, call `appImpression.handleTap(reengagementURL:)` with a universal link; AdAttributionKit opens the advertised app at that URL
- AdAttributionKit appends a query parameter to the URL so the advertised app can detect re-engagement opens
- Re-engagement postbacks have `conversion-type = "re-engagement"` and only support click (not view-through)
- First conversion value update must occur within 48 hours of re-engagement
- Use `conversionTypes: [.reengagement]` in `PostbackUpdate` to update only re-engagement postbacks; `[.install]` for install postbacks; `nil` updates all

**Testing**
- AdAttributionKit Developer Mode: enable in iOS Settings → Developer → AdAttributionKit Developer Mode
- Developer mode removes time randomization, shortens conversion windows, and speeds up postback transmission for easier testing

## APIs & Frameworks

**AdAttributionKit** **[NEW]**
- `AppImpression` **[NEW]** — represents an ad impression; initialized with a JWS compact string
- `AppImpression(jws:)` **[NEW]** — initializer
- `AppImpression.handleTap()` **[NEW]** — record a click impression; valid for 15 minutes after initialization
- `AppImpression.handleTap(reengagementURL:)` **[NEW]** — initiate re-engagement; opens advertised app at URL
- `AppImpression.beginView()` **[NEW]** — start a view-through impression
- `AppImpression.endView()` **[NEW]** — end a view-through impression (must be at least 2 seconds after `beginView`)
- `Postback` **[NEW]** — type for updating conversion values in the advertised app
- `Postback.updateConversionValue(_:)` **[NEW]** — async method to update conversion data
- `PostbackUpdate` **[NEW]** — struct with conversion update parameters
- `PostbackUpdate.fineConversionValue` **[NEW]** — integer 0–63
- `PostbackUpdate.lockPostback` **[NEW]** — Bool; freeze and schedule this postback
- `PostbackUpdate.coarseConversionValue` **[NEW]** — `.low`, `.medium`, `.high`
- `PostbackUpdate.conversionTypes` **[NEW]** — `[ConversionType]`; `.install`, `.reengagement`; `nil` = update all

**StoreKit (updated for AdAttributionKit)**
- `SKOverlay.AppConfiguration.appImpression` **[NEW]** — set an `AppImpression` instance on SKOverlay config
- `SKStoreProductViewController.loadProduct(withParameters:impression:completionBlock:)` **[NEW]** — `impression` parameter accepts an `AppImpression`
- `SKOverlay.AppConfiguration.adAttributionReengagementURL` **[NEW]** — URL for re-engagement via SKOverlay
- `SKStoreProductViewController.loadProduct(withParameters:reengagementURL:completionBlock:)` **[NEW]** — `reengagementURL` for re-engagement via SKStoreProductViewController

**UIKit**
- `UIEventAttributionView` — must overlay any interactive subview of a custom click ad (unchanged from Private Click Measurement)

## Code Highlights

Initialize an `AppImpression` from a JWS string:
```swift
let jwsString = fetchAdImpressionJWS() // from your ad network
let appImpression = try AppImpression(jws: jwsString)
```

Handle a click on a custom ad:
```swift
// In tap handler
try await appImpression.handleTap()
```

View-through ad lifecycle:
```swift
// When ad content appears
try await appImpression.beginView()
// (at least 2 seconds later, when ad content disappears)
try await appImpression.endView()
```

Update conversion value after a high-value event:
```swift
let update = PostbackUpdate(
    fineConversionValue: 42,
    lockPostback: true,
    coarseConversionValue: .high)
try await Postback.updateConversionValue(update)
```

Update only re-engagement postbacks:
```swift
let update = PostbackUpdate(
    fineConversionValue: 20,
    lockPostback: false,
    coarseConversionValue: .medium,
    conversionTypes: [.reengagement])
try await Postback.updateConversionValue(update)
```

## Takeaways
- AdAttributionKit replaces SKAdNetwork as the recommended attribution framework; both remain interoperable, so migrate incrementally.
- Use `lockPostback: true` only for your highest-value conversion events — it freezes the postback and schedules it for faster transmission at the cost of missing later updates.
- Plan your `fineConversionValue` schema (0–63) in advance with your ad network and any mobile measurement partners, since the values must be meaningful for campaign analysis.
- Enable AdAttributionKit Developer Mode during development to collapse conversion windows and remove timing randomization for faster iteration.

---
_Source: WWDC24 Session 10060 page (abstract, chapter summaries, transcript, and resource links)._
