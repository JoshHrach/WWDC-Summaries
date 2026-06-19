# Optimize your monetization with App Analytics
**WWDC25 · Session 252** · [Watch](https://developer.apple.com/videos/play/wwdc2025/252/)

_Platforms:_ All Apple platforms (App Store Connect)

## Overview
App Analytics receives its most significant expansion since launch, adding over 100 new metrics focused on monetization, subscriptions, and offer performance. The interface moves to a new home within the Apps tab of App Store Connect, and filtering capabilities expand from three to seven simultaneous filters with multi-value selection per filter — enabling analyses like "conversion rate for a specific custom product page in two territories on two device types."

The session uses a fictional "Exercise" app as the running example, demonstrating how the new cohort metrics, subscription lifecycle charts, offer conversion funnels, and Analytics Reports API combine to give developers a complete picture of their revenue funnel from first download through long-term subscription retention.

## Key Topics

### New Home and Expanded Filtering
App Analytics moves from a standalone section to the Analytics tab within an app's page on App Store Connect. The sidebar now organizes sections by customer journey stage. Filtering expands to seven simultaneous filters with multi-value selection, enabling granular segment analysis without leaving the dashboard.

### Download-to-Paid Conversion and Cohort Analysis
Two new cohort metrics appear in the Monetization section:
- **Download-to-Paid Conversion** — shows what percentage of users from a download cohort become paying users over time (day 1, 7, 14, 35+)
- **Average Proceeds per Download** — tracks cumulative revenue per download cohort member over time

The new Cohorts Analysis page displays these as a heat-map table where each cell represents a download cohort × time-since-download combination. Two peer group benchmarks appear below each metric for instant competitive context.

### Subscription Analytics (50+ New Metrics)
A new Subscriptions section provides:
- **Net Paid Plans graph** — shows plan starts (activations + reactivations) minus churn (voluntary + involuntary) over time, making growth/decay immediately legible
- **Average subscription retention** — cohort-style retention curves showing what % of subscribers remain active at 1, 3, 6, 12 months
- **50+ subscription state and event metrics** — states include: offer period, full-price paying, billing issue, churned; events include: activation, reactivation, churn

### Offer Metrics and Conversion Funnel
The new Offers section tracks active offers, offer starts, and the conversion rate from offer period to fully paid plan. The Cohorts view adds "Subscription Retention by offer start" — showing what percentage of offer subscribers convert to paid and how long they retain.

### Analytics Reports API — Two New Reports
The Reports API (which exports App Analytics data via the App Store Connect API) adds two new subscription-specific reports:
- **Subscription State Report** — snapshot of all subscription states at a point in time
- **Subscription Event Report** — event-level data for state transitions

These replace the older Sales and Trends subscription reports and link download data to subscription performance in a privacy-preserving way.

## APIs & Frameworks

- **App Store Connect Analytics** — new home under Apps tab **[NEW layout]**
- **Download-to-Paid Conversion metric** **[NEW]** — cohort-based payer conversion
- **Average Proceeds per Download metric** **[NEW]** — cohort-based revenue accumulation
- **Cohorts Analysis page** **[NEW]** — heat-map table for cohort metrics
- **Net Paid Plans graph** **[NEW]** — subscription growth/churn visualization
- **Subscription state metrics (25+)** **[NEW]** — instantaneous subscription states
- **Subscription event metrics (25+)** **[NEW]** — subscription state transition events
- **Offers section** **[NEW]** — offer conversion and retention analytics
- **Analytics Reports API** (existing) — updated with two new subscription reports **[NEW reports]**
  - Subscription State Report **[NEW]**
  - Subscription Event Report **[NEW]**
- **StoreKit** — referenced for implementing introductory offers surfaced in analytics
- **Custom Product Pages** — first-class filter dimension in all new metrics

## Code Highlights

_This session covers App Store Connect dashboards and the Analytics Reports REST API. No Swift SDK code is presented. Reports are accessed via the App Store Connect API:_

```
GET https://api.appstoreconnect.apple.com/v1/analyticsReportRequests
```

_Report segments can be filtered by territory, device type, app referrer, custom product page, and other dimensions using the standard App Store Connect API query parameters._

## Takeaways

- The seven-filter, multi-value filtering system makes it practical to analyze custom product page performance by territory and device type simultaneously — use this to validate product page experiments rigorously.
- Download-to-Paid Conversion cohort data shows *when* users convert, not just *if* — this is critical for timing offer windows and onboarding optimization.
- The Net Paid Plans graph separates starts from churn, making it easy to attribute subscription growth or decline to the right lever.
- Migrate from the Sales and Trends subscription reports to the new Subscription State and Event Reports before they are deprecated.

---
_Source: WWDC25 Session 252 page (abstract, chapter summaries, code samples, and resource links)._
