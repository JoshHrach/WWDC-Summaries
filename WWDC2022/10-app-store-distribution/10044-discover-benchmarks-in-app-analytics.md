# Discover Benchmarks in App Analytics
**WWDC22 · Session 10044** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10044/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, watchOS (App Store Connect tool — platform-agnostic)

## Overview
This session introduces App Benchmarks, a new feature in App Store Connect's App Analytics that allows developers to compare their app's performance against a privacy-preserving peer group of similar apps. The goal is to give developers context for their metrics — knowing whether a 5.5% conversion rate improvement represents strong performance or leaves room for further optimization requires knowing how peer apps are performing.

The peer groups are constructed using App Store category and monetization model (free, freemium, paid, paidmium, subscription) to ensure relevance. Privacy is maintained through differential privacy: small amounts of noise are added to each data point, and minimum peer group sizes prevent inference about individual apps. The feature was announced as coming "early next year" (early 2023).

## Key Topics

### Peer Group Benchmarking
- New **App Benchmarks** tool in App Store Connect App Analytics **[NEW]**.
- Shows how an app ranks among similar apps across the full customer lifecycle: acquisition, usage, monetization.
- Displays distribution percentiles: 25th, 50th (median), 75th — shows relative position without revealing individual app data.
- Peer groups formed by: App Store category + monetization model (free, freemium, paid, paidmium, subscription).
- Attributes tested to ensure meaningful comparisons over time.

### Benchmarked Metrics
- **Conversion Rate** — percentage of App Store page viewers who download/redownload; measures acquisition efficiency.
- **Day 1 Retention** — percentage returning 1 day after download.
- **Day 7 Retention** — percentage returning 7 days after download.
- **Day 28 Retention** — percentage returning 28 days after download.
- **Crash Rate** — rate of crashes relative to sessions; high crash rate negatively impacts engagement and monetization.
- **Average Proceeds Per Paying User** — monetization efficiency metric.

### Privacy Model
- **Differential Privacy** — the gold standard for privacy-preserving aggregation **[NEW application]**.
- Small amounts of calibrated noise added to each shared data point about the peer group.
- Minimum peer group size enforced so membership of any individual app cannot be inferred.
- Individual app performance is never revealed to other developers.

### Taking Action on Benchmarks
Benchmarks naturally pair with these existing App Store developer tools:
- **Product Page Optimization** — A/B test app icons, screenshots, previews to improve conversion rate.
- **Custom Product Pages** — audience-specific product pages targeting different user segments.
- **In-App Events** — timely events (competitions, premieres, livestreams) to re-engage and acquire users.
- **App Clips** — lightweight app entry points in relevant contexts to drive first-time engagement.
- **Pricing Tiers** — experiment with pricing to optimize monetization.
- **Promoted In-App Purchases** — browsable in-app purchase listings directly on App Store product pages.

## APIs & Frameworks
This is a business analytics/App Store Connect feature — no developer-facing APIs or frameworks.

- **App Store Connect** — App Analytics section, new Benchmarks tab **[NEW]**
- **App Analytics** — existing metrics: acquisition, usage, monetization
- **Differential Privacy** — privacy technique applied to peer group data aggregation
- **Product Page Optimization** — A/B testing for product page assets (existing)
- **Custom Product Pages** — per-audience product page variants (existing, introduced iOS 15)
- **In-App Events** — timely event showcases on App Store (existing, introduced iOS 15)
- **App Clips** — lightweight task-focused app experiences (existing)
- **Promoted In-App Purchases** — in-app purchase listings on App Store product pages (existing)

## Code Highlights
No code samples — this is an App Store Connect analytics feature accessed entirely through the web dashboard at appstoreconnect.apple.com.

## Takeaways
- App Benchmarks provides context for App Analytics metrics by showing percentile ranking against category/monetization peers — helping developers prioritize where to invest improvement efforts.
- Privacy is rigorously maintained through differential privacy; no individual app's data is exposed to competitors.
- The benchmarked metrics span the full customer lifecycle (conversion, retention, crash rate, monetization) giving a holistic view of app health relative to the market.
- Combine benchmark insights with Product Page Optimization, Custom Product Pages, In-App Events, and pricing experimentation to systematically improve weak areas.

---
_Source: WWDC22 Session 10044 page (abstract, chapter summaries, code samples, and resource links)._
