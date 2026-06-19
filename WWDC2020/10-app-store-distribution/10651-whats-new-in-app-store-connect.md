# What's New in App Store Connect
**WWDC20 · Session 10651** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10651/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
App Store Connect receives several major capability additions for iOS 14. The largest is end-to-end support for App Clips: configuring App Clip Invocation URLs for TestFlight beta testing, setting App Clip Card metadata (default and advanced), associating App Clip experiences with Apple Maps places, and validating Associated Domains in real time. The session also covers Family Sharing for subscriptions and non-consumable in-app purchases (available to all developers starting Fall 2020), a new Game Center configuration page for Challenges and Recurring Leaderboards, and a major expansion of the App Store Connect API with over 200 new endpoints covering App Metadata and Power & Performance.

The session emphasizes the multi-step nature of App Clip setup: build delivery via TestFlight, App Clip Card metadata configuration, advanced experience registration (which can include Apple Maps place associations and URL-prefix matching), and Associated Domain validation. The API expansion moves App Store Connect tooling from a read-heavy supplemental API to a comprehensive automation layer that can create versions, set pricing, edit metadata, associate builds, and submit for review—all programmatically.

## Key Topics
- **App Clips in TestFlight** — App Clip builds visible in TestFlight; configure up to 3 Invocation URLs per build; testers launch via TEST button without an App Clip Card **[NEW]**
- **App Clip Card metadata** — header image, title, subtitle, call-to-action; default experience required for all App Clips; applies to Safari and Messages invocations **[NEW]**
- **Advanced App Clip Experiences** — per-URL metadata; Apple Maps place association; Maps Action verb; URL-prefix matching for wildcard coverage; two relationship models (own business / authorized by owner) **[NEW]**
- **Associated Domain validation** — cache status (drives on-device invocations); real-time debug status loads Apple App Site Association file from servers **[NEW]**
- **Game Center — Challenges** — enable player-to-player achievement/score challenges via checkbox on the Game Center landing page **[NEW]**
- **Game Center — Recurring Leaderboards** — start time, duration, recurrence rule; scores collected for a defined window that repeats on a schedule **[NEW]**
- **Family Sharing for subscriptions** — enable Family Sharing on any auto-renewable subscription or non-consumable IAP; once enabled, cannot be disabled; new subscribers opt-in automatically; existing subscribers must opt-in **[NEW, Fall 2020]**
- **App Store Connect API expansion** — 200+ new endpoints: App Metadata (versions, pricing, metadata, builds, submission) and Power & Performance data download **[NEW]**

## APIs & Frameworks

**App Store Connect (web UI)**
- App Clip Invocation URL configuration — up to 3 invocations per TestFlight build
- App Clip Card fields: header image, title, subtitle, call-to-action verb
- Advanced App Clip Experience — URL, bundle ID, card metadata, Maps Action, place selection, relationship type
- Domain validation — cache status (on-device) and debug status (real-time server validation)
- Game Center feature toggles — Challenges checkbox, Recurring Leaderboard scheduler (start date/time, duration, recurrence rule)
- Family Sharing toggle on IAP detail page — one-way enablement; applies to auto-renewable subscriptions and non-consumable IAPs

**Apple App Site Association (AASA) file**
- `appclips.apps` array — must include App Clip App ID for domain association
- `webcredentials`, `applinks` — existing fields; App Clip domain association adds `appclips` key **[NEW]**

**Safari Smart App Banner (HTML)**
- `<meta name="apple-itunes-app" content="app-id=…, app-clip-bundle-id=…">` — links Safari page to an App Clip; `app-clip-bundle-id` is the new attribute **[NEW]**

**App Store Connect REST API**
- App Metadata endpoints **[NEW]** — `POST /v1/apps/{id}/appStoreVersions`, pricing, availability, localizations, builds association, review submission
- Power and Performance API **[NEW]** — `GET /v1/apps/{id}/perfPowerMetrics` — download aggregate performance/power data matching Xcode Organizer metrics

**StoreKit (referenced)**
- Family Sharing entitlement — `SKProduct.isFamilyShareable` property (read via StoreKit); enabled in App Store Connect
- Auto-renewable subscription Family Sharing — see "What's New in In-App Purchase" session for StoreKit API details

## Code Highlights

HTML meta tag associating a web page with an App Clip:
```html
<meta name="apple-itunes-app"
      content="app-id=myAppStoreID,
               app-clip-bundle-id=com.example.MyApp.Clip,
               affiliate-data=myAffiliateData,
               app-argument=myURL">
```

Apple App Site Association file (appclips key):
```json
{
  "appclips": {
    "apps": ["TEAMID.com.example.MyApp.Clip"]
  },
  "applinks": {
    "apps": [],
    "details": [...]
  }
}
```

## Takeaways
- Every App Clip must have a default App Clip Card metadata configuration (header image, title, subtitle, call-to-action); use TestFlight App Clip Invocation URLs to test deep-linking and the full experience before release.
- Use URL-prefix matching when registering advanced App Clip experiences to cover large sets of per-item URLs (e.g., `/reserve/campsite/` covers all campsites) with a single registration rather than one per item.
- Family Sharing for subscriptions and non-consumable IAPs is a one-way gate in App Store Connect—once enabled on a live product, it cannot be disabled; update product metadata to reflect that the purchase is family shareable before enabling.
- The expanded App Store Connect API (200+ new endpoints) enables full CI/CD automation of App Store metadata, build association, and review submission without opening App Store Connect in a browser.

---
_Source: WWDC20 Session 10651 page (transcript and resource links)._
