# Explore Testing In-App Purchases
**WWDC23 · Session 10142** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10142/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10

## Overview
App Store provides three complementary tools for testing in-app purchases throughout the development lifecycle: StoreKit Testing in Xcode, the App Store sandbox, and TestFlight. Each serves a different stage of development and offers unique capabilities for validating purchase flows, subscription states, and server integration.

StoreKit Testing in Xcode enables fully offline, local testing without requiring App Store Connect product setup. Xcode 15 adds new static renewal rates (independent of subscription duration), multi-device transaction manager views, and the ability to purchase directly from the transaction manager without launching the app.

The App Store sandbox has been updated with support for billing problem message simulation, billing grace period configuration, and — arriving later in 2023 — Family Sharing testing and new on-device Account Settings options (renewal rate, interrupted purchases, and purchase history clearing).

## Key Topics

### StoreKit Testing in Xcode
- Fully offline, no server dependency; products defined in a StoreKit configuration file.
- New in Xcode 15: static renewal rate options independent of subscription duration; multi-device transaction manager; direct IAP purchasing from transaction manager.
- Supports error simulation, offer code redemption, price increase sheet, and billing retry/grace period scenarios.
- Automation via the `StoreKitTest` framework; products can be synced from App Store Connect.

### App Store Sandbox
- Tests end-to-end client + server flows using products set up in App Store Connect.
- Requires a paid application agreement and sandbox Apple IDs created in Users and Access.
- New billing problem message simulation: disable "Allow Purchases & Renewals" in sandbox Account Settings to trigger billingIssue StoreKit Message.
- Billing grace period: configured per app in App Store Connect under App Subscriptions; sandbox uses preset durations based on renewal rate.
- Upcoming Family Sharing support: set up sandbox family members in App Store Connect, enable Family Sharing on products, then test entitlement, revocation, and App Store Server Notifications per family member.
- Upcoming on-device Account Settings additions: renewal rate adjustment, interrupted purchase testing, and purchase history clearing.

### TestFlight
- End-to-end beta testing using the tester's real Apple ID (not charged for purchases).
- Subscription renewal rates mirror sandbox default rates.
- New in 2023: tester filtering by status/sessions, bulk select for group management, and an "Internal Only" distribution method that prevents accidental App Store submission.
- Supports App Store Server Notifications and App Store Server API (shared with sandbox).

### Tool Comparison
- Offer code redemption and price increase sheet: Xcode only.
- Billing retry and grace period: Xcode and sandbox.
- Server-side validation (Server Notifications, Server API): sandbox and TestFlight.
- External tester feedback: TestFlight only.

## APIs & Frameworks
- `StoreKit` — core in-app purchase framework
- `StoreKit.Message` **[NEW context]** — message API with `reason: .billingIssue` for billing problem presentation
- `StoreKit.Message.Reason.billingIssue` — reason triggering the billing problem sheet
- `StoreKitTest` framework — automation and unit testing of IAP flows
- `Product.isFamilySharable` — property to merchandise family-sharable products
- `Transaction.revocationDate` — date when family member loses access to a shared purchase
- `JWSTransaction` — signed transaction payload including `revocationDate`
- `showManageSubscription` API — presents subscription management sheet (TestFlight-testable)
- App Store Server Notifications — real-time server callbacks for subscription events, family member transactions
- App Store Server API — server-side transaction verification
- StoreKit configuration file (`.storekit`) — local IAP product definitions for Xcode testing

## Code Highlights
No code samples were provided in this session. The session is primarily conceptual/workflow-focused, demonstrating tooling through the Xcode transaction manager UI and App Store Connect settings rather than code.

## Takeaways
- Use StoreKit Testing in Xcode for rapid, offline, local iteration; use sandbox for complete client + server validation before launch.
- Xcode 15 adds static renewal rates and direct transaction manager purchases, reducing friction in subscription testing.
- Family Sharing sandbox support (arriving later in 2023) enables full validation of entitlement, revocation, and server notifications for shared purchases.
- TestFlight's new Internal Only build distribution prevents beta builds from being accidentally submitted to App Store review.

---
_Source: WWDC23 Session 10142 page (abstract, chapter summaries, code samples, and resource links)._
