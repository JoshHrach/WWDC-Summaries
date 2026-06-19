# What's new with SKAdNetwork
**WWDC22 · Session 10038** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10038/)

_Platforms:_ iOS 16 (SKAdNetwork 4.0)

## Overview
SKAdNetwork 4.0 delivers four major advances over the existing privacy-preserving install attribution system, all while maintaining crowd anonymity guarantees for users. The most significant change is a transition from a simple 2-digit campaign identifier to a 4-digit hierarchical source identifier, which allows ad networks to encode multi-dimensional campaign data (campaign, placement, geo bucket, etc.) in a single number. The system returns progressively more digits as the install crowd grows, so privacy is never compromised.

Alongside the new source identifier, conversion values are split into fine-grained (6-bit, same as before) and coarse-grained (three values: low/medium/high) variants. The coarse value is disclosed at a lower crowd-anonymity threshold, allowing advertisers to receive signal sooner. The single-postback model is replaced with three postbacks tied to consecutive time windows, enabling re-engagement measurement across the full user lifecycle — not just the first conversion period.

A new web-attribution flow extends SKAdNetwork to ads served in Safari web pages. An `<a>` element with `attributionDestination` and `attributionSourceNonce` attributes triggers a signed-impression fetch from the ad network server, and the App Store handles the rest identically to in-app impressions. For developers and ad networks, the StoreKitTest framework (available since Xcode 13.3) now includes `SKAdTestSession` APIs for validating signed impressions and testing postback delivery locally or to a remote server.

## Key Topics

### Crowd Anonymity
Install counts gate how much attribution data is returned. At low counts (high uniqueness), only the least granular data is returned. As count grows, progressively more data is disclosed. All four features in 4.0 are designed around this model.

### Hierarchical Source Identifier (4 digits)
The 2-digit campaign ID is replaced by a 4-digit `sourceIdentifier` (range 0000–9999). Ad networks should compose source identifiers as a hierarchy: the least significant 2 digits carry meaning at the lowest crowd-anonymity tier; adding the 3rd and 4th digits encodes higher-resolution signal revealed only at higher tiers. The field is renamed `sourceIdentifier` (from `campaignIdentifier`).

### Hierarchical Conversion Values
Two conversion values per postback: **fine** (6-bit integer, 0–63, same as today) and **coarse** (enum: `.low`, `.medium`, `.high`). The coarse value is disclosed at a lower crowd threshold; the fine value requires a higher threshold. Only one level of value is delivered to the ad network per postback. The new `updatePostbackConversionValue(_:coarseValue:completionHandler:)` API sets both.

### Multiple Conversions (Three Postbacks)
Three postbacks are sent across three consecutive time windows (replacing today's single postback). Only the first postback can carry the fine conversion value; the second and third carry only the coarse value. Only the winning ad network and the developer receive the second and third postbacks.

### Web Attribution
Links in Safari web pages can drive SKAdNetwork attribution. An `<a>` element with `attributionDestination` (the ad network's domain) and `attributionSourceNonce` attributes triggers an HTTP POST from Apple to a well-known URL at `attributionDestination`. The ad network responds with a signed `SKAdImpression` JSON that includes the new `sourceDomain` field (analogous to `sourceAppStoreItemIdentifier` for in-app flows). The link can appear on first-party sites or in cross-origin iframes.

### SKAdNetwork Testability (Xcode 13.3+)
`SKAdTestSession` in the `StoreKitTest` framework validates signed impressions and allows controlled postback delivery. `validate(impression:publicKey:)` checks both configuration and signature. `SKAdTestPostback` configures a postback for local or remote testing; `setPostbacks(_:)` registers it; `flushPostbacks()` dispatches it.

## APIs & Frameworks

**StoreKit — SKAdNetwork 4.0** **[NEW]**
- `SKAdImpression.sourceIdentifier` **[NEW]** — 4-digit source identifier (replaces `campaignIdentifier`)
- `SKAdNetwork.sourceIdentifier` key **[NEW]** — dictionary key for impression dictionaries
- `SKAdNetwork.CoarseConversionValue` **[NEW]** — enum: `.low`, `.medium`, `.high`
- `SKAdNetwork.updatePostbackConversionValue(_:coarseValue:completionHandler:)` **[NEW]** — sets both fine and coarse conversion values
- `SKAdImpression.sourceDomain` **[NEW]** — domain of web ad source (web attribution)
- `<a attributionDestination="..." attributionSourceNonce="...">` **[NEW]** — HTML link format for web attribution
- Three-postback system **[NEW]** — second and third postbacks tied to time windows post-install

**StoreKitTest — SKAdNetwork Testing (Xcode 13.3+)**
- `SKAdTestSession` — test session for SKAdNetwork
- `SKAdTestSession.validate(impression:publicKey:)` **[NEW]** — validates impression signature and configuration
- `SKAdTestPostback` **[NEW]** — configures a test postback
- `SKAdTestSession.setPostbacks(_:)` **[NEW]** — registers test postbacks
- `SKAdTestSession.flushPostbacks()` **[NEW]** — dispatches registered postbacks to specified URL

## Code Highlights

Setting a hierarchical source identifier on an `SKAdImpression`:
```swift
let impression = SKAdImpression(
    sourceAppStoreItemIdentifier: 123456789,
    advertisedAppStoreItemIdentifier: 987654321,
    adNetworkIdentifier: "example123.skadnetwork",
    adCampaignIdentifier: 1)
impression.sourceIdentifier = 5739  // 4-digit: campaign=57, geo=3, placement=9
```

Updating both fine and coarse conversion values:
```swift
SKAdNetwork.updatePostbackConversionValue(
    42,                          // fine conversion value (0–63)
    coarseValue: .high,          // coarse: .low / .medium / .high
    completionHandler: { error in
        if let error { print(error) }
    }
)
```

Web attribution link format:
```html
<a href="https://apps.apple.com/app/id987654321"
   attributionDestination="https://adnetwork.example.com"
   attributionSourceNonce="abc123nonce">
    Download Food Truck
</a>
```

Validating a signed impression in tests:
```swift
let session = try SKAdTestSession()
let impression = SKAdImpression(/* ... */)
try session.validate(impression: impression, publicKey: publicKey)
```

Sending a test postback:
```swift
let postback = SKAdTestPostback(
    version: SKAdNetwork.Version._4_0,
    adNetworkIdentifier: "example.skadnetwork",
    adCampaignIdentifier: 1,
    sourceIdentifier: 5739,
    advertisedAppStoreItemIdentifier: 987654321,
    purchasedItemIdentifier: nil,
    sourceAppStoreItemIdentifier: 123456789,
    conversionValue: 42,
    coarseConversionValue: .high,
    didWin: true,
    postbackURL: URL(string: "https://localhost:8080/postback")!)
try session.setPostbacks([postback])
try session.flushPostbacks()
```

## Takeaways
- SKAdNetwork 4.0's 4-digit hierarchical `sourceIdentifier` lets ad networks pack multi-dimensional campaign data into one field, with crowd anonymity gating how many digits are disclosed per postback.
- Coarse conversion values (`.low`/`.medium`/`.high`) lower the install-count threshold needed to receive attribution signal, enabling faster feedback at reduced privacy cost.
- Three postbacks tied to consecutive time windows replace the single-postback model, allowing re-engagement (e.g., user completing a critical action days after install) to be measured.
- Web attribution (`attributionDestination`/`attributionSourceNonce` link attributes) extends SKAdNetwork privacy-preserving attribution to Safari web ads without any code changes in the advertised app.

---
_Source: WWDC22 Session 10038 page (abstract, full transcript, and resource links)._
