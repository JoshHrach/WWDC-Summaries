# What's New in App Store Connect
**WWDC23 · Session 10117** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10117/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
This session walks through the major App Store Connect updates across the full developer lifecycle: monetization, testing, store presence, and API automation. Key themes for 2023 include StoreKit for SwiftUI for quick in-app purchase integration, 900-price-point international pricing management, expanded TestFlight tester management tools, sandbox Family Sharing testing, visionOS-specific privacy data types, regional pre-orders, and substantial new App Store Connect API endpoints for Game Center and authentication.

The session also introduces several quality-of-life improvements for TestFlight: a new "TestFlight Internal Only" build flag in Xcode's distribution workflow, device/OS columns in the tester dashboard, bulk tester management, and the ability to upload "What to Test" notes via Xcode Cloud alongside a build.

## Key Topics

### Monetization and Pricing
- **StoreKit for SwiftUI** (new): Add a few lines of code in Xcode after setting up products in App Store Connect to generate fully accessible, localized in-app purchase views. Customizable backgrounds, buttons, and styles; optionally uses App Store promotional image.
- **International pricing upgrade**: Choose from 900 price points; set a base region to auto-generate prices in all other regions and currencies; let App Store auto-adjust for currency/tax changes or manage manually; set regional availability for in-app purchases and subscriptions.

### TestFlight Improvements
- **New tester data column**: Devices column shows the most recent device and OS on which the beta app was installed.
- **Tester filtering and bulk actions**: Filter testers by status, sessions, crashes, or feedback; bulk-select to resend invitations, add to groups, or remove testers. All data available via App Store Connect API.
- **TestFlight Internal Only builds** **[NEW]**: New selection in Xcode's distribution workflow that prevents a build from being submitted for External TestFlight or App Store review; clearly marked in App Store Connect.
- **Xcode Cloud "What to Test"** **[NEW]**: Upload plain text `What to Test` notes alongside a build via a file in a `TestFlight/` folder next to the Xcode project, or via custom build scripts pulling from commit messages.

### Sandbox Testing Enhancements
- **Sandbox Family Groups** **[NEW]**: Combine up to six sandbox test accounts into a Family Group to test Family Sharing for subscriptions and in-app purchases.
- **iOS sandbox on-device enhancements**: View Family Group members; stop sharing a subscription or nonconsumable with a family member; modify subscription renewal rate; test interrupted purchases; clear purchase history — all on-device.

### Store Presence and Privacy
- **New visionOS privacy data types** **[NEW]**: `Environment Scanning` (user's surroundings), `Hands` (hand structure/movements), `Head` (head movements) — relevant for visionOS and any iOS app using ARKit to collect this data. Appear on all platform App Stores where the app is released.
- **Regional pre-orders** **[NEW]**: Offer the app for download in some regions and pre-order in others simultaneously; redesigned availability page to manage app state per region.
- **Product page optimization**: Tests now run continuously until manually stopped; new versions no longer terminate running tests.

### App Store Connect API
- **In-app purchases and subscriptions API** (launched earlier in 2023): Automate product management.
- **Customer reviews and responses API**: Automate review management.
- **Sandbox testers API**: Manage sandbox accounts programmatically.
- **Game Center API** **[NEW, coming in 2023]**:
  - Create, configure, and archive leaderboards and achievements.
  - Archiving: new feature to remove leaderboards/achievements from a game.
  - Server-to-server score submission and achievement unlock events (great for cross-platform games).
  - Remove scores and players from leaderboards to manage fraudulent activity.
  - Custom matchmaking rules (skill level, region, etc.).
- **API key enhancements** **[NEW]**:
  - Marketing API keys: scoped to manage only marketing metadata.
  - Customer service API keys: scoped to respond to App Store reviews only.
  - User-based API keys: key inherits the permissions of the generating user's role, created from the user profile page.

## APIs & Frameworks

- `App Store Connect API` — REST API for automating App Store management
- `App Store Connect API` / Game Center endpoints **[NEW]** — leaderboards, achievements, score submission, matchmaking, player removal
- `App Store Connect API` / marketing API key **[NEW]** — scoped key for marketing metadata management
- `App Store Connect API` / customer service API key **[NEW]** — scoped key for review responses
- `App Store Connect API` / user-based API key **[NEW]** — role-scoped key per user profile
- `App Store Connect API` / tester management endpoints **[NEW]** — bulk actions, device data, filtering
- `StoreKit for SwiftUI` **[NEW]** — framework/API for generating in-app purchase product views from App Store Connect products in SwiftUI
- `TestFlight` — beta distribution platform; new internal-only build flag **[NEW]**
- `TestFlight Internal Only` build flag **[NEW]** — Xcode distribution workflow option preventing external distribution
- `Xcode Cloud` — CI/CD; new `TestFlight/` folder convention for "What to Test" notes **[NEW]**
- Sandbox Family Groups **[NEW]** — test account grouping for Family Sharing testing
- Privacy data types: `Environment Scanning`, `Hands`, `Head` **[NEW]** — visionOS/ARKit-specific App Store privacy label categories
- International pricing: 900 price points, base region selection **[NEW]** — App Store Connect pricing tool upgrade
- Regional pre-orders **[NEW]** — per-region app availability and pre-order state management
- Product page optimization — continuous test running, unaffected by new app versions **[updated behavior]**

## Code Highlights
No code samples in this session (App Store Connect tooling and API overview). Key workflow details:

- Add `TestFlight/WhatToTest.en-US.txt` (plain text) next to your `.xcodeproj` or `.xcworkspace` for Xcode Cloud to pick up "What to Test" notes automatically.
- Use the App Store Connect API Game Center endpoints for server-to-server score submissions and custom matchmaking instead of manual configuration in App Store Connect.

## Takeaways
- StoreKit for SwiftUI dramatically reduces the code required to offer in-app purchases; the 900-price-point international pricing system makes global launch pricing far more manageable.
- TestFlight gains meaningful tester management and quality-control features: Internal Only builds prevent accidental external distribution; "What to Test" automation removes a repetitive manual step for Xcode Cloud users.
- Sandbox Family Groups finally allow developers to properly test Family Sharing subscription flows before going live.
- The App Store Connect API continues its expansion into Game Center (leaderboards, achievements, matchmaking, score management) and adds role-scoped API keys for tighter permission control.

---
_Source: WWDC23 Session 10117 page (abstract, chapter summaries, and resource links)._
