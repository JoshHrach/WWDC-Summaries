# What's New in App Analytics
**WWDC21 · Session 10115** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10115/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12 (Mac with Apple Silicon)

## Overview
App Analytics in App Store Connect receives a significant redesign in 2021, adding dashboards and metrics that give developers a more complete picture of their app's performance across the entire App Store ecosystem — with no additional code required. The session walks through four major areas of improvement: richer transaction monitoring, macOS/iPhone-and-iPad-on-Mac usage data, App Clips analytics, and new dashboards tied to three new App Store features launching in fall 2021.

The new metrics cover the full customer lifecycle — from pre-orders through downloads, updates, and re-downloads — and introduce Proceeds as a directly trackable metric. All data is surfaced in privacy-friendly ways using aggregate signals rather than individual user tracking.

Companion sessions cover Product Page Optimization and Custom Product Pages in depth ("Get ready to optimize your App Store product page") and In-App Events ("Meet In-App Events on the App Store").

## Key Topics

### Transaction Monitoring Enhancements
A new pre-order dashboard surfaces Impressions, Product Page Views, and Pre-Order counts broken down by device, source type, and territory. After launch, new metrics include Proceeds (revenue), App Updates (tracking version adoption), Re-Downloads (measuring re-engagement), and Total Downloads (first-time downloads + re-downloads). These launch later in 2021 via App Store Connect.

### macOS and iPhone/iPad Apps on Mac
App Analytics now reports Mac usage for apps automatically available on Apple Silicon Macs with no additional code. Developers can segment Sessions, Installations, Active Devices, Crashes, and Deletions by device dimension to isolate macOS/Desktop data from iOS data.

### App Clips Analytics
A dedicated App Clips section on the Overview page shows Card Views, Sessions, Crashes, and Installations for app clips, including a breakdown of where clip installations originated and top territories by Card Views, Installations, and Sessions.

### Product Page Optimization, Custom Product Pages, and In-App Events
Three new App Store features each receive dedicated analytics dashboards. Product Page Optimization includes a Bayesian statistical analysis of A/B test variants to determine the best-converting icon, screenshots, and preview videos. Custom Product Pages analytics track views, downloads, conversion rate, proceeds, and retention per page variant. In-App Events analytics show event-specific Impressions, Notifications, Downloads, and App Opens from the event page.

## APIs & Frameworks

**App Store Connect / App Analytics (no SDK APIs — dashboard and web-based)**
- Pre-order dashboard **[NEW]** — Impressions, Product Page Views, Pre-Orders; dimensions: device, source type, territory
- Proceeds metric **[NEW]** — revenue from in-app purchases and paid downloads
- App Updates metric **[NEW]** — count of app version updates over time
- Re-Downloads metric **[NEW]** — distinct from first-time downloads
- Total Downloads metric **[NEW]** — first-time downloads + re-downloads combined
- macOS device segmentation **[NEW]** — Sessions, Installations, Active Devices (last 30 days), Crashes, Deletions for Desktop/Mac category
- App Clips dashboard **[NEW]** — Card Views, Sessions, Crashes, Installations; dimension: territory, source
- Product Page Optimization dashboard **[NEW]** — variant conversion rates with Bayesian statistical analysis
- Custom Product Pages analytics **[NEW]** — per-page views, downloads, conversion rate, proceeds, retention
- In-App Events analytics **[NEW]** — Impressions, Notifications, Downloads, App Opens per event
- App Store Connect API — programmatic access to analytics data

## Code Highlights

No code changes are required in the app itself. All new capabilities are accessed through App Store Connect dashboards and, where applicable, the App Store Connect API.

## Takeaways
- App Analytics now covers the full download lifecycle: pre-orders, first-time downloads, re-downloads, total downloads, updates, and proceeds — no SDK code needed.
- Mac usage of iPhone/iPad apps on Apple Silicon is now measurable via the Desktop device segment in existing analytics.
- App Clips receive a dedicated analytics view with card-view and installation funnels, enabling data-driven optimization.
- Three new fall 2021 App Store features — Product Page Optimization, Custom Product Pages, and In-App Events — each come with dedicated analytics dashboards using Bayesian analysis and per-variant metrics.

---
_Source: WWDC21 Session 10115 page (abstract, chapter summaries, code samples, and resource links)._
