# Get Ready to Optimize Your App Store Product Page
**WWDC21 · Session 10295** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10295/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
Apple introduced two major App Store product page features planned to launch later in 2021: Custom Product Pages and Product Page Optimization. Custom product pages allow up to 35 distinct versions of an app's page (with unique URLs, different screenshots/previews/promotional text) to target specific audiences or features. Product page optimization enables A/B testing of visual assets (icon, screenshots, previews) against the default page to identify the highest-converting variant.

Both features are managed in App Store Connect without requiring a new app version submission, and both will be fully accessible via the App Store Connect API.

## Key Topics

**Custom Product Pages**
Up to 35 custom product pages can be created per app. Each custom page can have a different set of app preview videos, screenshots, and promotional text. Pages are fully localizable and each has its own unique shareable URL. App Analytics will track impressions, downloads, conversion rate, retention, and average proceeds per paying user per page. No new app version is needed — metadata is submitted independently through App Store Connect.

Use cases include: highlighting a specific feature for a targeted ad audience (e.g., live streaming vs. GPS tracking for a hiking app), promoting specific game characters or modes, or showcasing video content for particular shows/channels.

**Product Page Optimization**
Allows A/B testing of up to three treatment variants against the default product page. Testable elements: app icon, app preview videos, screenshots. Tests can be scoped to specific localizations. Traffic is split by percentage (e.g., choosing 30% means each of three treatments receives 10% of total traffic). Metrics: impressions, downloads, conversion rate, and improvement vs. baseline.

App icon testing requires variant icons to be included in the app binary (all device sizes + 1024x1024 App Store version) for the currently live app version. Users who see a test icon on the product page will see that same icon on their home screen after download. To apply a winning icon as the default, it must be set in the binary of the next app version.

**App Store Connect API**
Both custom product pages and product page optimization will be fully supported via the App Store Connect API (spec to be published later in 2021).

## APIs & Frameworks

- **App Store Connect** — management UI for both features **[NEW]**
- **Custom Product Pages** **[NEW]**
  - Up to 35 per app
  - Per-page assets: app preview videos, screenshots, promotional text
  - Fully localizable
  - Unique shareable URL per page
  - App Analytics metrics per page: impressions, downloads, conversion rate, retention, average proceeds per paying user
  - No new app version required for submission
- **Product Page Optimization** **[NEW]**
  - Up to 3 treatment variants per test
  - Testable: app icon, app preview videos, screenshots
  - Traffic split control (percentage of audience)
  - Localization scoping (run test in specific locales only)
  - Metrics: impressions, downloads, conversion, improvement vs. baseline
  - Icon variants must be included in the app binary (all sizes + 1024x1024)
  - Winning icon applied via next app version binary update
  - App Review approval required before test begins
- **App Store Connect API** — full API support for both features coming later in 2021 **[NEW]**
- **App Analytics** — extended with per-custom-page metrics, product page optimization test results **[NEW]**

## Code Highlights

No Swift/Objective-C code samples — this is an App Store Connect / product management session. Icon variant assets for product page optimization tests must be embedded in the app binary at the standard asset catalog paths. No StoreKit API changes introduced in this session.

## Takeaways

- Custom product pages let you send different audiences (from different ad campaigns or links) to different App Store pages showcasing the specific features they care about — up to 35 per app, no new binary required.
- Product page optimization enables data-driven decisions about visual assets (icon, screenshots, previews) through statistically measured A/B tests, without requiring a new app release to set up or run.
- App icon A/B testing requires pre-embedding all variant icons in the currently live app binary, so plan app releases to include icon variants before starting a test.
- Both features will be automatable via the App Store Connect API for integration into CI/CD pipelines and marketing workflows.

---
_Source: WWDC21 Session 10295 page (abstract, chapter summaries, code samples, and resource links)._
