# What's New in App Store Pricing
**WWDC23 · Session 10014** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10014/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
In March 2023, Apple shipped the largest upgrade to App Store pricing since the App Store launched. This session provides a comprehensive walkthrough of those new capabilities: 900 price points (up from the previous set), base-region pricing with automatic global equalization across 175 regions and 44 currencies, and advanced per-region tools for temporary promotional prices and permanent custom prices.

The session is structured in two parts: global pricing (set one base price, let the App Store generate and auto-adjust all other regional prices) and advanced regional pricing (manually manage prices for specific regions or all regions at once for promotions and custom strategies). Both the App Store Connect UI and the new App Store Connect API v2.3 endpoints are demonstrated.

## Key Topics

### Enhanced Global Pricing
- **900 price points**: 800 available by default, 100 additional higher price points available on request; all for apps, in-app purchases, and subscriptions across 44 currencies.
- **Base region**: Select any of 175 regions as the pricing basis; the App Store generates equivalent prices for all other regions based on foreign exchange rates and taxes.
- **Automatic global equalization**: Prices in non-base regions adjust automatically as exchange rates and taxes fluctuate; base price is not auto-adjusted.
- **Global price changes**: Schedule future price changes from a base price; App Store generates comparable regional prices with the same future effective date.
- **14-day advance notice**: Upcoming automatic price adjustments are shown in App Store Connect at least 14 days before they take effect.
- **Existing apps**: Apps created before March 9, 2023, default to United States as the base region; changing the base region resets the price schedule.

### Advanced Pricing by Region
- **Temporary price changes**: Manually set a lower (or different) price in specific regions for a defined time window (start and end date). Prices return to auto-adjusted values after the period ends. No limit on number of regions. Auto-adjustment is paused for those regions during the period.
- **Custom (permanent) price changes**: Manually set a permanent price in any or all regions. When a custom price is set for the base region, automatic adjustment is disabled globally — the developer becomes responsible for tracking exchange rates and taxes.
- Mix: some regions can be manually managed while others remain on auto-adjustment.

### App Store Connect UI Workflow
1. Pricing and Availability → Price Schedule → Add Pricing (or "+" button).
2. Choose base country/region.
3. Pick from 25 common price points (or "See Additional Prices" for all 900).
4. Review and optionally override per-region comparable prices; confirm.
5. For temporary/custom changes: pick type, set date range (temporary) or effective date (custom), select regions and prices.

### App Store Connect API (v2.3)
- **`AppPriceSchedules` resource**: Contains three relationships — `baseTerritory`, `manualPrices`, `automaticPrices`.
- **`AppPrice` resource**: Linked to an `AppPricePoint` and a `Territory`.
- **`AppPricePoint` resource**: Read-only reference data containing price point details.
- **`Territory` resource**: Read-only reference data for country/region.
- New pricing APIs replace deprecated older pricing APIs; deprecated APIs will be retired later in 2023.

Key API endpoints:
- `GET /v1/apps/{id}/appPriceSchedule/baseTerritory` — get base region
- `GET /v1/apps/{id}/appPriceSchedule/manualPrices` — get manually set prices
- `GET /v1/apps/{id}/appPriceSchedule/automaticPrices` — get auto-generated prices for all other regions
- `GET /v1/appPricePoints/{id}/equalizations` — preview comparable price points in other regions for a given price point
- `POST /v1/appPriceSchedules` — create/overwrite the full price schedule (current + upcoming prices, base territory, manual prices)

## APIs & Frameworks

- `App Store Connect API v2.3` **[NEW]** — new pricing resource set
- `AppPriceSchedules` resource **[NEW]** — top-level price schedule with `baseTerritory`, `manualPrices`, `automaticPrices` relationships
- `AppPrice` resource **[NEW]** — individual price entry with `startDate`, `endDate`, linked `AppPricePoint`
- `AppPricePoint` resource **[NEW]** — read-only price point reference (price value + currency + territory)
- `Territory` resource — read-only country/region reference
- `GET /v1/apps/{id}/appPriceSchedule/baseTerritory` **[NEW]**
- `GET /v1/apps/{id}/appPriceSchedule/manualPrices` **[NEW]**
- `GET /v1/apps/{id}/appPriceSchedule/automaticPrices` **[NEW]**
- `GET /v1/appPricePoints/{id}/equalizations` **[NEW]** — preview comparable prices across regions
- `POST /v1/appPriceSchedules` **[NEW]** — create or overwrite price schedule
- 900 price points **[NEW]** — expanded from prior set; 800 default + 100 higher on request
- Base region configuration **[NEW]** — choose any of 175 regions as pricing origin
- Automatic global equalization **[NEW]** — FX and tax-adjusted prices for non-base regions
- Temporary price change **[NEW]** — time-bounded manual price override per region
- Custom price change **[NEW]** — permanent manual price override per region
- App Store Connect Pricing and Availability UI — redesigned Price Schedule management

## Code Highlights
No Swift/Objective-C code samples. Key API pattern:

```json
// POST /v1/appPriceSchedules — Schedule a global price change + current price
{
  "data": {
    "type": "appPriceSchedules",
    "relationships": {
      "app": { "data": { "type": "apps", "id": "APP_ID" } },
      "baseTerritory": { "data": { "type": "territories", "id": "USA" } },
      "manualPrices": {
        "data": [
          { "type": "appPrices", "id": "${newprice-0}" },
          { "type": "appPrices", "id": "${newprice-1}" }
        ]
      }
    }
  },
  "included": [
    {
      "type": "appPrices", "id": "${newprice-0}",
      "attributes": { "startDate": null, "endDate": "2024-01-01" },
      "relationships": { "appPricePoint": { "data": { "type": "appPricePoints", "id": "CURRENT_PRICE_POINT_ID" } } }
    },
    {
      "type": "appPrices", "id": "${newprice-1}",
      "attributes": { "startDate": "2024-01-01", "endDate": null },
      "relationships": { "appPricePoint": { "data": { "type": "appPricePoints", "id": "NEW_PRICE_POINT_ID" } } }
    }
  ]
}
```

## Takeaways
- The 900 price points and base-region auto-equalization system dramatically reduce the operational burden of global pricing; developers set one price and the App Store handles FX/tax adjustments.
- Temporary prices enable time-limited regional promotions without manual cleanup; the price automatically reverts to the auto-adjusted value after the end date.
- Custom prices give full manual control per region, but disable automatic adjustment — use with care and review all 175 regions before confirming.
- Developers should migrate from deprecated pricing APIs to the new `AppPriceSchedules` / `AppPrice` / `AppPricePoint` resources in App Store Connect API v2.3 before the old APIs are retired.

---
_Source: WWDC23 Session 10014 page (abstract, transcript, and resource links)._
