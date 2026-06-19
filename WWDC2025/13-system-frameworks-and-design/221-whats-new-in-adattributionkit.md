# What's new in AdAttributionKit
**WWDC25 · Session 221** · [Watch](https://developer.apple.com/videos/play/wwdc2025/221/)

_Platforms:_ iOS 18.4+, iPadOS 18.4+

## Overview
AdAttributionKit is Apple's privacy-first ad attribution framework, launched in iOS 17.4 with install conversions and expanded to re-engagement in iOS 18. This session covers four major new features: conversion tags for overlapping re-engagement conversion windows, configurable attribution windows, configurable attribution cooldowns, and country-code geography data in postbacks. It also introduces significant new testability tools directly in the iOS Settings app.

## Key Topics

### Conversion Tags for Overlapping Re-engagement Conversions
Prior to iOS 18.3, only one active re-engagement conversion could exist at a time. As of iOS 18.4, apps can have multiple concurrent re-engagement conversion windows. Conversion tags act as bookmarks — they are appended to the re-engagement URL delivered to the app, allowing the app to extract the tag and use it to update only the specific postback tied to that campaign. Opt in via the `EligibleForAdAttributionKitOverlappingConversions` key in Info.plist.

### Configurable Attribution Window
Advertisers can now specify how long after an ad display time the ad can still be considered for attribution (previously fixed at 30 days for click-through, 1 day for view-through). Configured in Info.plist under `AdAttributionKitConfigurations → AttributionWindows` on a per-ad-network and per-interaction-type basis. A `global` key sets a fallback for all ad networks. The `ignoreInteractionType` key excludes a specific ad type. Attribution windows apply to install conversions only.

### Configurable Attribution Cooldown
Prevents a later conversion from "stealing" value measurements that should be attributed to an earlier conversion. A cooldown period (in hours) can be set per conversion type (`install-cooldown-hours`, `reengagement-cooldown-hours`) in Info.plist. During the cooldown after a conversion, other conversions are not attributed.

### Geography Data in Postbacks
Country code is now included as a new unsigned field in postbacks, derived from the App Store storefront for App Store installs or from the install verification token for alternative marketplace apps. The field is subject to crowd anonymity — it only appears when conversion volume is sufficient. For alternative marketplaces, the `ccode` field is added to the install verification token JWS payload.

### New Testability in Developer Settings
In iOS 18.4, development postbacks can be created directly in iOS Settings → Developer → Development Postbacks, without needing an end-to-end publisher app flow. Supports configuring bundle ID, server URL, postback properties (including country code), and data granularity tiers. Postbacks are signed with a new key (`kid` value) and use `development.adattributionkit` as the ad network identifier. Useful for testing server processing of different data tiers before production release.

## APIs & Frameworks

**AdAttributionKit**
- `EligibleForAdAttributionKitOverlappingConversions` (Info.plist key) **[NEW]** — opt in to multiple concurrent re-engagement conversion windows
- `Postback.reengagementOpenURLParameter` **[NEW]** — query parameter key to extract a conversion tag from a re-engagement deep-link URL
- `PostbackUpdate(fineConversionValue:lockPostback:conversionTag:)` **[NEW]** — init with `conversionTag` to target a specific concurrent postback
- `Postback.updateConversionValue(_:)` — existing API; still works without a tag (updates most recent conversion)
- `AdAttributionKitConfigurations → AttributionWindows` (Info.plist) **[NEW]** — per-ad-network / per-interaction-type attribution window length in days; supports `click`, `view`, `ignoreInteractionType`, and `global` keys
- `AdAttributionKitConfigurations → AttributionCooldown` (Info.plist) **[NEW]** — `install-cooldown-hours`, `reengagement-cooldown-hours`
- `country-code` postback field **[NEW]** — unsigned field subject to crowd anonymity; available for App Store and alternative marketplace installs
- Install verification token `ccode` field **[NEW]** — country code provided by alternative app marketplaces in their JWS payload

**iOS Settings / Developer Tools**
- Development Postbacks page (iOS Developer Settings) **[NEW]** — create, configure, and transmit development postbacks without an end-to-end publisher app

## Code Highlights

```swift
// Extract conversion tag from re-engagement URL
func retrieveConversionTag(fromURL url: URL) -> String? {
    guard let components = URLComponents(url: url, resolvingAgainstBaseURL: true),
          let queryItems = components.queryItems else { return nil }
    return queryItems.first { $0.name == Postback.reengagementOpenURLParameter }?.value
}
```

```swift
// Update a specific concurrent postback using its tag
func updateConversionValue(_ value: Int, conversionTag: String) async {
    let update = PostbackUpdate(fineConversionValue: value,
                                lockPostback: false,
                                conversionTag: conversionTag)
    try await Postback.updateConversionValue(update)
}
```

```json
// Info.plist: configurable attribution window and cooldown
{
  "AdAttributionKitConfigurations": {
    "AttributionWindows": {
      "global": { "install": { "view": 3 } },
      "com.example.adNetwork": { "install": { "click": 5, "ignoreInteractionType": "view" } }
    },
    "AttributionCooldown": {
      "install-cooldown-hours": 6,
      "reengagement-cooldown-hours": 1
    }
  }
}
```

## Takeaways
- Set `EligibleForAdAttributionKitOverlappingConversions` to `YES` and store conversion tags per re-engagement URL to accurately attribute multiple simultaneous campaigns.
- Configure `AttributionWindows` in Info.plist to prevent stale ads from competing for attribution.
- Use `AttributionCooldown` to protect high-value conversions from being overshadowed by subsequent re-engagement interactions.
- Migrate from SKAdNetwork to AdAttributionKit to take advantage of all these new measurement capabilities.
- Use the new iOS Developer Settings testability to validate postback processing at every data granularity tier before going to production.

---
_Source: WWDC25 Session 221 page (abstract, chapter summaries, code samples, and resource links)._
