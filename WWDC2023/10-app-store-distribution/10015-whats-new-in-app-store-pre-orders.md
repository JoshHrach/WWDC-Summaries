# What's New in App Store Pre-Orders
**WWDC23 · Session 10015** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10015/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
This session covers the significant expansion of App Store pre-order capabilities in 2023: regional publishing. Previously, pre-orders were available globally or not at all. Now developers can offer an app for pre-order in specific regions simultaneously with a live release ("soft launch") in other regions, enabling a much more nuanced global rollout strategy.

The session walks through the redesigned App Store Connect Pricing and Availability page, which provides a per-region status view and new per-region pre-order management. All pre-order functionality is also exposed through the App Store Connect API for automation.

## Key Topics

### Regional Pre-Orders (NEW)
- For **unreleased apps**: choose specific regions for pre-order; expected release date up to 180 days in the future.
- For **already-released apps** (soft launch scenario): publish a live version in select regions while simultaneously publishing pre-orders in other regions; expected release date up to 365 days from when the pre-order was originally published in those regions.
- Pre-orders can be expanded to additional regions at any time as the app's geographic reach grows.
- Once an app is released in a region, it cannot be put back into pre-order status in that same region.

### App Availability Page (Redesigned)
- New per-region status view: shows whether each country/region has the app "Ready for Sale," "Published for Pre-order," or "Unavailable" with explanatory labels.
- "Set Up Availability" dialog for new apps; "Manage" section for already-published apps.
- Manage existing pre-orders: update release date, release immediately, or remove the pre-order from a specific region.

### Pre-Order Setup Workflow
1. Navigate to Pricing and Availability in App Store Connect.
2. Click "Set Up Availability" (new apps) or "Manage" (existing apps).
3. Select "Pre-Order in Specific Countries or Regions."
4. Enter expected release date (up to 180 days for new apps, up to 365 days for apps already live in other regions).
5. Select target regions.
6. Changes appear on the App Store within 24 hours.

### Managing Existing Pre-Orders
- **Update release date**: Change to any future date within the 365-day window.
- **Release immediately**: Triggers customer notifications and automatic app download.
- **Remove pre-order**: Pulls the pre-order from the App Store in the selected region.

### Marketing and Analytics
- Localized pre-order badges available for websites and marketing materials (deep link to product page).
- Pre-order apps discoverable via App Store search and may be featured on Today, Games, or Apps tabs.
- App Analytics tracks purchased pre-orders by date and by region.
- Custom product pages and product page optimization tests now supported for pre-order apps; performance trackable in App Analytics.

### App Store Connect API Support
- All regional pre-order functionality (set up, manage, update, release, remove) is also available through the App Store Connect API.

## APIs & Frameworks

- **App Store Connect** — primary tool for configuring regional pre-orders
- **App Store Connect Pricing and Availability page** (redesigned) **[NEW]** — per-region status view and pre-order management
- **App Store Connect API** — pre-order management endpoints support regional pre-orders **[NEW functionality]**
- **App Analytics** — tracks pre-order purchases by date and region; supports custom product page performance for pre-order apps **[NEW]**
- **Product page optimization** — now available for pre-order apps **[NEW]**
- **Custom product pages** — now available for pre-order apps **[NEW]**
- Pre-order localized badges — Apple-provided marketing assets for developer websites

## Code Highlights
No code samples. This session covers App Store Connect UI workflows and API configuration with no client-side SDK changes required.

## Takeaways
- Regional pre-orders enable soft-launch strategies: release in a handful of markets first, then build pre-order momentum in remaining regions before expanding globally.
- The redesigned Availability page gives a clear, per-region dashboard of an app's distribution state across all markets.
- Pre-order apps now support product page optimization and custom product pages, closing the gap between pre-order and live app marketing capabilities.
- All pre-order management is available via the App Store Connect API, enabling automation of global launch sequencing.

---
_Source: WWDC23 Session 10015 page (abstract, transcript, and resource links)._
