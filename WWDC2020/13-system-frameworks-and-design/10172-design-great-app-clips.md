# Design Great App Clips
**WWDC20 · Session 10172** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10172/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
Presented by Grant Paul and Heena Ko from the Apple Design Team, this session covers the design principles and UX patterns for building effective App Clips — the lightweight, zero-install native app experiences introduced in iOS 14. The session defines what App Clips are, walks through all the entry points through which users discover them, explains how to design the App Clip Card (the first screen users see before entering any App Clip), and provides detailed guidance on how to select the right feature subset and design the in-App Clip user flow.

This is the design companion to the engineering sessions "Configure and Link Your App Clips" (10146), "Explore App Clips" (10174), and "Streamline Your App Clip" (10120).

## Key Topics

**What App Clips Are**
App Clips are lightweight, fully native experiences that deliver a subset of an app's functionality with no install required. They appear contextually when and where users need them. They do not persist on the device — swiping them away removes them. Users who want persistent access can download the full app. App Clips use the same iOS frameworks as full apps, including Apple Pay, Sign In with Apple, CoreLocation, ARKit, and Bluetooth.

**App Clip Icon**
The App Clip icon is automatically derived from the app's standard icon with a system-generated border. No extra design work is required.

**Entry Points (Discovery)**
App Clips are found contextually, not through App Store browsing:
- **App Clip Codes** — new visual codes designed specifically for App Clips; include both an NFC zone (center) and a visual scan code (outer ring). Instantly recognizable. The preferred physical entry point.
- **NFC tags** — hold phone near tag to invoke
- **QR codes** — scan to invoke
- **Maps** — App Clips attached to locations can surface in Maps (enables preordering before arriving)
- **Siri Suggestions** — popular/relevant App Clips may surface in lock screen widget and Spotlight
- **Safari Smart App Banner** — add to a web page's HTML to offer the App Clip directly from the page; if app is installed, opens app instead
- **Messages** — when a page with a Smart App Banner is shared in Messages, the link offers the App Clip inline

**Using Different Codes Per Context**
For a business with multiple locations, each location should have its own code so the App Clip launches directly into that location's context. Within a location, each table can have its own code so the App Clip auto-populates table number. This eliminates a navigation step the user would otherwise have to take and reduces errors.

**App Clip Card Design**
The first thing users see after invoking an App Clip. Customizable elements:
- **Title** — app/brand name, or a feature-specific title if the App Clip focuses on one feature (e.g., "Radio" for a music app's radio-only App Clip)
- **Subtitle** — explains what the App Clip does; critical for new-to-brand users who may not know the app name
- **Header image** — photograph of the physical location/service, or custom brand graphic; never use screenshots or text-heavy graphics
- **Action button** — one of: Open (default), View (for information display), Play (for media/games)
- **Attribution** — links to the full app's App Store page; populated from app metadata, not separately configurable

**Designing the App Clip Experience**

Three principles:
1. **Decide the single purpose** — one task, not a mini-app
2. **Determine the minimum flow** — shortest path to completing that task; use Apple Pay and Sign In with Apple to eliminate friction
3. **Offer the full app after completion** — at the end of the flow, after value is demonstrated, with a clear explanation of what additional benefits the full app provides

**What to Remove**
- Splash screens, introductions, tutorials
- Login/account creation pages (use Sign In with Apple if auth is needed)
- Global navigation, tab bars
- Settings or management screens
- Anything unrelated to the single task

**Flow Examples**
- Café ordering: browse menu → Apple Pay → offer app after payment
- Parking: pay for space → notification before expiry → extend time from notification → no full app prompt needed if task is complete
- Online store: browse/add to cart → Apple Pay → post-purchase Sign In with Apple offer
- Restaurant table order: scan table code → browse menu → add items continuously → Apple Pay checkout → offer app for future ordering and delivery
- AR experience: camera view → augmented annotations → share option

**Notifications**
App Clips can send notifications for up to 8 hours after first use by default. For experiences spanning more than one day (e.g., multi-day car rental), request explicit notification authorization for up to one week. Never send unexpected or promotional notifications; only send in direct response to user action.

**Full App Promotion**
Promote the full app only after the primary task is complete — never before. Clearly explain the additional value the full app provides. App Clips are not a Trojan horse; they must provide standalone value.

## APIs & Frameworks

### App Clips
- `NSUserActivity` with `NSUserActivityTypes` — carries the App Clip invocation URL to the App Clip code
- `SKOverlay` **[NEW]** — presents a non-modal overlay banner prompting download of the full app; use after task completion
- `SKStoreProductViewController` — alternative for full-screen App Store promotion
- `ASAuthorizationAppleIDProvider` (Sign In with Apple) — preferred auth mechanism; no password entry
- `PKPaymentAuthorizationController` (Apple Pay) — preferred payment mechanism; no card entry

### App Clip Code / Invocation
- App Clip Code generator — available via Apple (announced with session) for creating physical codes
- `targetContentIdentifier` in App Clip invocation URL — identifies the specific experience (location, table, context)
- App Store Connect → Advanced Experiences — configure per-location, per-context App Clip experiences and their associated URLs

### Notifications
- `UNUserNotificationCenter.requestAuthorization` — standard notification auth; App Clips get 8-hour ephemeral permission by default
- Explicit authorization request — for experiences spanning more than one day

### Human Interface Guidelines
- [HIG: App Clips](https://developer.apple.com/design/human-interface-guidelines/app-clips) — canonical reference for all App Clip design guidelines
- "Choosing the right functionality for your App Clip" — Apple documentation article

## Code Highlights

No code samples were shown in this design session. For implementation, see the engineering sessions:
- "Configure and Link Your App Clips" (10146)
- "Streamline Your App Clip" (10120)

## Takeaways
- An App Clip should do exactly one thing and do it in the minimum number of steps — remove all navigation, settings, login flows, and onboarding that are not strictly required to complete that single task.
- Use a different App Clip code (NFC tag / QR code) for each distinct context (location, table, product) so the App Clip launches pre-configured for that context — this removes at least one navigation step and eliminates a whole class of user errors.
- Never promote the full app before the App Clip has delivered its value; only offer the download after the primary task is successfully completed, with a clear explanation of what additional benefits the full app provides.
- The App Clip Card subtitle is critical for brand-new users who don't know the app — use it to explain exactly what the App Clip enables, not just to repeat the app name.

---
_Source: WWDC20 Session 10172 page (abstract and full transcript)._
