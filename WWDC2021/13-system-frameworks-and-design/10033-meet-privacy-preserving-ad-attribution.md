# Meet Privacy-Preserving Ad Attribution
**WWDC21 · Session 10033** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10033/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session covers two complementary privacy-preserving ad attribution technologies: Private Click Measurement (PCM) for web and app-to-web campaigns, and updates to SKAdNetwork for app-to-App Store campaigns. Both approaches allow advertisers to measure ad campaign effectiveness without identifying, tracking, or profiling individual users.

Private Click Measurement is a W3C Privacy Community Group proposed standard implemented in WebKit/Safari. It supports both web-to-web and (new in iOS 15) app-to-web attribution. Reports are on-device, sent with random 24–48 hour delays, include IP address protection (iOS 15/macOS 12), and use cryptographic blind signatures for fraud prevention. App Tracking Transparency requirements do not apply for PCM uses.

SKAdNetwork received multiple updates (versions 2.1–3.0): stronger 256-bit signing keys, view-through attribution, multiple postbacks (winner + up to 5 runner-ups), IP address protection, and a new postback-to-developer feature.

## Key Topics

**PCM Web-to-Web Attribution**
Source site embeds `attributionsourcenonce` and `attributiondestination` link attributes. Destination site triggers a conversion by making an HTTP GET to the source site, which redirects to a `.well-known/private-click-measurement/trigger-attribution/` endpoint with trigger data and priority. Browser combines click and conversion, sends anonymized JSON report to both parties on a 24–48 hour random delay.

**PCM App-to-Web Attribution (New in iOS 15)**
Apps display ads using `UIEventAttributionView` overlaid on the ad. On tap, create a `UIEventAttribution` with source identifier, destination URL, source description, and purchaser. Pass this to `UIApplication.open(_:options:)` or `UIScene.open(_:options:)`. Reports go to the endpoint specified in `NSAdvertisingAttributionReportEndpoint` in Info.plist.

**PCM Fraud Prevention**
RSA blind signatures: browser fetches source site's public key, constructs a blinded message (hidden in an "envelope"), source site signs it, browser unblinds — source never saw the original message so it cannot link click and conversion. Reports include the unblinded signature for validation.

**PCM Debug Mode**
Available in Safari's Develop menu (macOS) or WebKit Experimental Features (iOS). Speeds reports to every 10 seconds and adds enhanced Web Inspector logging.

**SKAdNetwork Version History**
- v2.1 (iOS 14.0): stronger 256-bit Apple public key
- v2.2 (iOS 14.5): view-through attribution via `SKAdImpression`, `startImpression`, `endImpression`; `fidelityType` parameter (1 = click-through, 0 = view-through)
- v2.2+/iOS 14.6: multiple postbacks (winner + up to 5 runner-ups); IP address protection; postback-to-developer feature

**SKAdNetwork Postback to Developer (New)**
Winning postback is now also sent to the advertised app's developer. Requires `NSAdvertisingAttributionReportEndpoint` in the app's Info.plist (same key as PCM).

## APIs & Frameworks

### WebKit / Safari — Private Click Measurement

**Web-to-Web (Link Attributes)**
- `attributionsourcenonce` — link attribute (8-bit source ID, 0–255) **[NEW]**
- `attributiondestination` — link attribute specifying the conversion site
- `.well-known/private-click-measurement/trigger-attribution/<trigger-data>/<priority>` — conversion endpoint
- JSON report format: `{ "source_site", "destination_site", "source_id", "trigger_data" }`

**App-to-Web — UIKit**
- `UIEventAttributionView` — transparent view placed over ad that verifies user gesture **[NEW]**
- `UIEventAttribution` — struct carrying attribution data **[NEW]**
  - `init(sourceIdentifier:destinationURL:sourceDescription:purchaser:)`
  - `sourceIdentifier: UInt8` — 8-bit campaign ID
  - `destinationURL: URL` — where conversion will occur
  - `sourceDescription: String` — description of tapped ad content
  - `purchaser: String` — description of the ad buyer
- `UIOpenURLOptions.eventAttribution: UIEventAttribution` — new option for `open(_:options:)` **[NEW]**
- Scene-based: `UIScene.OpenExternalURLOptions.eventAttribution` **[NEW]**

**Info.plist Key**
- `NSAdvertisingAttributionReportEndpoint` — URL for PCM app-to-web reporting endpoint (shared with SKAdNetwork postback-to-developer) **[NEW]**

### StoreKit — SKAdNetwork **[UPDATES]**

**SKAdNetwork v2.1**
- Apple 256-bit public key update — use new key to validate postbacks

**SKAdNetwork v2.2**
- `SKAdImpression` — represents a view-through ad impression **[NEW]**
  - `adNetworkIdentifier`, `adCampaignIdentifier`, `adImpressionIdentifier` (unique nonce), `sourceAppStoreItemIdentifier`, `advertisedAppStoreItemIdentifier`, `adType`, `adDescription`, `adPurchaserName`
  - `fidelityType: Int` — 1 = click-through (StoreKit-rendered), 0 = view-through **[NEW]**
- `SKAdNetwork.startImpression(_:completionHandler:)` — begin a view-through impression **[NEW]**
- `SKAdNetwork.endImpression(_:completionHandler:)` — end a view-through impression **[NEW]**

**SKAdNetwork v3.0 (iOS 14.6)**
- Multiple postbacks — winner postback + up to 5 runner-up postbacks **[NEW]**
  - Winner receives crowd-anonymity-controlled values (sourceApp, conversion value)
  - Runner-ups receive only basic attribution info
- IP address protection on all postbacks **[NEW]**
- Postback to developer — winning postback sent to `NSAdvertisingAttributionReportEndpoint` in advertised app's Info.plist **[NEW]**
- Existing APIs: `SKAdNetwork.registerAppForAdNetworkAttribution()`, `SKAdNetwork.updateConversionValue(_:)`

## Code Highlights

App-to-web attribution with `UIEventAttribution` (UIScene-based):
```swift
let eventAttribution = UIEventAttribution(
    sourceIdentifier: 55,
    destinationURL: URL(string: "https://longboardshop.biz")!,
    sourceDescription: "Lemon Yellow Longboard Ad",
    purchaser: "SearchForLongboard"
)

let options = UIScene.OpenExternalURLOptions()
options.eventAttribution = eventAttribution
scene.open(destinationURL, options: options, completionHandler: nil)
```

PCM trigger endpoint format (HTTP GET by destination site):
```
GET https://searchforlongboard.biz/.well-known/private-click-measurement/trigger-attribution/15/10
// trigger-data = 15, priority = 10
// Source site must redirect to same well-known path
```

Info.plist key for reporting endpoint:
```xml
<key>NSAdvertisingAttributionReportEndpoint</key>
<string>https://my-reporting-endpoint.example</string>
```

## Takeaways
- Private Click Measurement enables privacy-preserving web and app-to-web ad attribution without any identifying data — no ATT prompt required because no cross-site or cross-app user tracking occurs.
- `UIEventAttributionView` + `UIEventAttribution` is the app-side integration for PCM app-to-web; it requires user gesture verification and a reporting endpoint in Info.plist.
- SKAdNetwork v2.2 adds view-through attribution with `SKAdImpression`; v3.0 adds multiple postbacks and direct postback delivery to the advertised developer's endpoint.
- Both PCM and SKAdNetwork use the same `NSAdvertisingAttributionReportEndpoint` Info.plist key for their respective reporting destinations.

---
_Source: WWDC21 Session 10033 page (abstract, chapter summaries, code samples, and resource links)._
