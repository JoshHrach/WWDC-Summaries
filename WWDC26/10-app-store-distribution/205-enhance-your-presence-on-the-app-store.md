# Enhance Your Presence on the App Store
**WWDC26 · Session 205** · [Watch](https://developer.apple.com/videos/play/wwdc2026/205/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, visionOS, watchOS (App Store product pages across all platforms)

## Overview
This session reimagines how apps and games are marketed on the App Store by introducing new visual placements that go beyond screenshots. Two new surfaces — a **product page header** and enriched **search result tiles** — allow developers to use brand-quality images and videos as the very first thing a prospective user sees, shifting the presentation from a grid of UI screenshots to an expressive, editorial feel.

Alongside the new placements, Apple is shipping the **Asset Library**: a single, centralised repository inside App Store Connect where all visual assets for a product — screenshots, preview videos, in-app event media, and the new creative assets — are stored and managed across platforms, sizes, and placements. This eliminates the friction of uploading the same media multiple times and ensures consistency across every surface.

The session closes with a preview tool that renders a product page exactly as it will appear to users before any version goes live, removing guesswork from the submission process.

## Key Topics

### Introduction (0:06)
Images and videos shape a user's first impression of an app in seconds. Previously these were limited to screenshots and app preview videos in the screenshot gallery. New placements dramatically expand where and how visual assets are used.

### New Asset Placements (0:52–4:24)
- **Product page header** — a new full-width visual zone at the very top of the product page, above the standard screenshot gallery. Accepts images or short looping videos. This is the first thing visitors see when they land on the page.
- **Search result placements** — enriched search result tiles that surface images or videos directly in search, allowing brands to stand out before a user even taps through to the full product page.
- Both placements pull assets from the Asset Library; no separate upload workflow is needed once assets are in the library.
- Assets can be used in **Apple Ads campaigns** through the same creative asset system, enabling visual consistency from ad impression through to product page.

### Meet Asset Library (4:24–7:35)
- **Asset Library** is a new centralised media management tool in App Store Connect.
- Stores all app visual assets in one place: screenshots, app preview videos, in-app event media, product page header images/videos, and search result creative assets.
- Manages assets across platforms (iPhone, iPad, Mac, Apple TV, Apple Watch, Apple Vision Pro), sizes, and placements.
- Replaces the previous fragmented workflow where the same asset had to be uploaded separately for different placements.
- Assets can be reused across product page versions, custom product pages, Apple Ads, and in-app events without re-uploading.

### Next Steps (7:35)
- Prepare brand-quality images and videos that showcase the app's core value.
- Upload them to Asset Library once for use across all supported placements.
- Use the new **product page preview tool** to view the page exactly as users will see it before submitting for review.

## APIs & Frameworks

### App Store Connect Features
- **Product page header placement** **[NEW]** — full-width image or video at the top of the product page
- **Search result visual placement** **[NEW]** — image or video displayed directly in App Store search results
- **Asset Library** **[NEW]** — centralised asset management in App Store Connect; supports screenshots, preview videos, in-app event media, product page header assets, and search result creative assets; assets are scoped per app and reused across placements
- **Product page preview tool** **[NEW]** — live render of how the product page will appear before submission
- **Custom product pages** — existing feature; now managed alongside new placements in Asset Library
- **In-app events** — existing feature; media now stored in Asset Library alongside other assets
- **Apple Ads creative assets** — images and videos uploaded to Asset Library can be used directly in Apple Ads campaigns (Design Your Own Ads workflow)

### Placement Surfaces (summary)
| Placement | Asset Type | Where Shown |
|---|---|---|
| Product page header | Image or video | Top of product page |
| Search result tile | Image or video | App Store search results |
| Screenshot gallery | Screenshots / preview video | Existing placement |
| Apple Ads | Creative assets | Ad impressions |

## Code Highlights
No code samples are present in this session — all configuration is performed through the App Store Connect UI and Asset Library. No StoreKit or client-side API changes are introduced.

## Takeaways
- Two new visual placements — **product page header** and **search result tiles** — let developers use brand-level images and videos as the first visual impression of their app, before screenshots.
- **Asset Library** centralises every visual asset for a product in one place, eliminating redundant uploads and ensuring consistency across all App Store surfaces and Apple Ads.
- Assets uploaded once to Asset Library are immediately available for product pages, custom product pages, in-app events, and Apple Ads campaigns.
- Use the new **product page preview tool** to validate the final presentation before submitting any version for review.

---
_Source: WWDC26 Session 205 page (abstract, chapter summaries, and resource links). No code samples included in this session._
