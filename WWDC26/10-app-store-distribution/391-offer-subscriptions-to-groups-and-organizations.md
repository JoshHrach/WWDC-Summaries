# Offer Subscriptions to Groups and Organizations
**WWDC26 · Session 391** · [Watch](https://developer.apple.com/videos/play/wwdc2026/391/)

_Platforms:_ iOS, iPadOS, macOS (StoreKit 2 required; Apple Business Manager / Apple School Manager integration)

## Overview
This session introduces two complementary distribution paths that let developers sell auto-renewable subscriptions to groups of users rather than individual accounts. The first is **Group Purchases** — an in-app flow where a single subscriber buys multiple seats and then invites others via invitation links. The second is **Volume Purchasing** — a B2B/education procurement pathway through **Apple Business Manager (ABM)** and **Apple School Manager (ASM)**, which enterprise and education IT administrators already use to deploy apps and content at scale.

Both paths are built on top of the existing StoreKit 2 auto-renewable subscription infrastructure and are configured entirely in App Store Connect. They represent a significant expansion of Apple's subscription model from consumer one-to-one sales into the organizational buying context, targeting workplaces, schools, clubs, and other multi-person groups.

## Key Topics

### Introduction (0:00)
Two distinct paths for selling subscriptions to groups:
1. **In-app Group Purchases** — subscriber-driven, buy-and-invite flow inside the app.
2. **Volume Purchasing** — administrator-driven procurement through Apple Business and Apple School Manager.

### Availability (2:17)
- Available for **all auto-renewable subscriptions** using **StoreKit 2**.
- Enabled by default for most subscriptions.
- Automatically **opted out** for subscriptions that use **Family Sharing** (the two features are mutually exclusive).
- Per-subscription opt-in/opt-out toggle available in **App Store Connect** for fine-grained control.

### Pricing (3:24)
- **Default**: each seat is priced at the subscription's current per-unit price.
- **Volume pricing** **[NEW]**: up to five price bands can be configured, with reduced per-seat prices at higher seat counts (e.g., 2–5 seats at one price, 6–20 seats at a lower price, etc.).
- Volume pricing tiers are configured in App Store Connect per subscription product.
- Applies to both Group Purchases and Volume Purchasing pathways.

### Purchasing (4:43)
- **Apple Business Manager / Apple School Manager**: administrators browse, select, and assign subscription licenses through the ABM/ASM portal; no in-app changes required from developers for this path.
- **In-app Group Purchases**: developers build the seat-selection UI (e.g., a stepper or quantity picker) and pass the desired **seat count** as a parameter to the **StoreKit 2 purchase flow**.
- The App Store handles payment, seat allocation, and the invitiation link generation infrastructure.

### Seat Management (5:25)
- **Volume Purchasing (ABM/ASM)**: seats are assigned to users through a **Mobile Device Management (MDM)** service; the administrator manages assignments via the MDM portal.
- **In-app Group Purchases**: seat holders receive invitation links; recipients can redeem a link to claim a seat. Developers can use either Apple's **included seat management system** or build a **custom seat management experience** within the app.

## APIs & Frameworks

### StoreKit 2
- **Auto-renewable subscriptions** — existing subscription type; Group Purchases and Volume Purchasing are layered on top with no new framework types described at the API surface level in this session
- **`Product.purchase(options:)`** — extended to accept a **seat count option** **[NEW]** for Group Purchase flows (specific option type not named in session content; integrate seat count via purchase options)
- StoreKit 2 requirement: Group Purchases and Volume Purchasing are only available for subscriptions implemented with StoreKit 2 (not the original StoreKit 1 / `SKPaymentQueue` API)

### App Store Connect Features
- **Group Purchases** availability toggle **[NEW]** — per-subscription on/off, disabled automatically for Family Sharing subscriptions
- **Volume Purchasing** availability toggle **[NEW]** — enables the subscription for ABM/ASM procurement
- **Volume pricing tiers** **[NEW]** — up to 5 seat-count bands, each with a distinct per-seat price; configured per subscription product
- **Seat management system** **[NEW]** — Apple-provided invitation link infrastructure for in-app group purchases; alternatively, developers build a custom system

### Apple Business Manager (ABM) / Apple School Manager (ASM)
- Existing enterprise/education procurement portals; subscriptions with Volume Purchasing enabled appear in their app catalogs
- MDM integration — seat assignment is handled through connected MDM solutions; no developer-side server work required

### Family Sharing
- **Mutual exclusion**: subscriptions using Family Sharing are automatically excluded from Group Purchases and Volume Purchasing

## Code Highlights
This session is primarily an App Store Connect configuration guide. No code snippets are shown in the session. The key developer action is to trigger the StoreKit 2 purchase API with an appropriate seat count parameter when the user selects a group size in the in-app Group Purchases UI. Refer to the StoreKit 2 `Product.purchase(options:)` documentation and the related session "What's new in managing Apple devices" (WWDC26 Session 206) for MDM-side integration details.

## Takeaways
- **Group Purchases** and **Volume Purchasing** are two distinct new pathways for selling subscriptions to multiple users under a single purchasing entity, both built on StoreKit 2 auto-renewable subscriptions.
- Both paths are configured in App Store Connect and are on by default for most StoreKit 2 subscriptions; developers can opt out per product and cannot use them alongside Family Sharing.
- **Volume pricing tiers** (up to five bands) allow per-seat price reductions at scale, making the offering competitive with direct B2B licensing.
- For **in-app Group Purchases**, developers build the seat-count UI and pass the quantity to StoreKit 2; Apple handles invitations, seat tracking, and payment. For **Volume Purchasing**, MDM administrators handle all assignment — no additional in-app changes are needed.

---
_Source: WWDC26 Session 391 page (abstract, chapter summaries, and resource links). No code samples included in this session._
