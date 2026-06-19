# Explore App Store Connect for Spatial Computing
**WWDC23 · Session 10012** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10012/)

_Platforms:_ visionOS 1

## Overview
This session covers everything needed to get a visionOS app into the App Store and into testers' hands via TestFlight. It explains the three distinct paths for making apps available on visionOS: creating a brand-new visionOS app record, adding the visionOS platform to an existing app for universal purchase, or enabling compatible iPad and iPhone apps to run without code changes.

The TestFlight section details how to distribute visionOS builds, manage beta groups with optional iOS-on-visionOS install controls, install apps on the headset, capture screenshots using the Digital Crown + top button combination, submit feedback, and analyze crash logs and engagement statistics in App Store Connect and Xcode Organizer. The session concludes with new visionOS-specific Privacy Nutrition Label data types that developers must correctly disclose.

## Key Topics

- **App setup options** — Three paths: (1) New app with visionOS platform; (2) Add visionOS to existing app (universal purchase — same name, URL, in-app purchases, automatic downloads); (3) Compatible iPad/iPhone apps auto-available on visionOS App Store (manageable via "iOS Apps on xrOS Availability" page or per-app Pricing and Availability).
- **Managing compatibility** — Individual app availability toggle in App Store Connect; verification of which builds are compatible with visionOS; releasing a native visionOS build replaces the iOS-compatible version on the store.
- **TestFlight for visionOS** — Distribute visionOS builds through new or existing beta groups; per-group toggle to enable/disable iPhone/iPad app installation on headset; browse installed apps in TestFlight sidebar with xrOS/iOS toggle; yellow dot badge on beta apps; app install directly from TestFlight or Home Screen; "What's new in this build" developer message on launch.
- **Feedback collection** — Screenshots via Digital Crown + top button simultaneously; submit via TestFlight Send Feedback flow with annotation/crop tools; crash feedback with step description and crash log; analysis in App Store Connect web/mobile and Xcode Organizer filtered by platform/build.
- **Privacy Nutrition Labels** — New visionOS-specific data types **[NEW]**: "Environment Scanning" (mesh, planes, scene classification, image detection of surroundings); "Hands" (hand structure and movement data); "Head" (head movement data); applicable to other platforms too.

## APIs & Frameworks

This session covers App Store Connect tooling, TestFlight, and privacy disclosure — no SDK APIs. Key workflow touchpoints:

**App Store Connect**
- New App dialog — platform selection now includes visionOS **[NEW]**
- Add Platform — adds visionOS to existing app record **[NEW]**
- "iOS Apps on xrOS Availability" — bulk control of compatible app availability **[NEW]**
- Pricing and Availability page — per-app "iPhone and iPad Apps on xrOS" toggle **[NEW]**
- App Privacy section — Environment Scanning, Hands, Head data type checkboxes **[NEW]**

**TestFlight**
- Group settings — "Enable iOS Apps on xrOS" per-group toggle **[NEW]**
- App page — xrOS/iOS version toggle for testers **[NEW]**
- iOS-only apps — separate category for incompatible apps **[NEW]**
- Screenshot capture — Digital Crown + top button shortcut **[NEW]**
- Crash feedback submission on visionOS **[NEW]**
- Feedback review — Xcode Organizer and App Store Connect both filterable by visionOS platform **[NEW]**

**App Store Connect API**
- Tester and group management available via API (unchanged)

## Code Highlights

No code samples — this is an App Store Connect and TestFlight workflow session.

Workflow summary for getting a visionOS app on the store:
1. In App Store Connect, click + → New App → select visionOS platform, fill in Name/Bundle ID/SKU → Create
2. Upload build from Xcode targeting the visionOS destination
3. Set up TestFlight group, add testers, optionally enable iOS app install on headset
4. Collect feedback via Digital Crown + top button screenshots and crash reports
5. Update App Privacy section with Environment Scanning / Hands / Head if applicable
6. Submit for App Review

## Takeaways

- There are three distinct ways to get an app on the visionOS App Store: new app record, universal purchase addition, or compatible iOS app availability — choose based on whether separate pricing/availability or a unified product page is desired.
- TestFlight for visionOS works like other platforms but adds a per-group control for whether iOS app builds can be installed on the headset — useful for rolling out compatibility testing incrementally.
- Any visionOS app that accesses environment data (scene mesh, plane detection), hand tracking, or head movement must now declare those in Privacy Nutrition Labels under the new "Environment Scanning," "Hands," and "Head" data types.
- Publishing a native visionOS build automatically replaces the previously available compatible iOS version on the visionOS App Store.

---
_Source: WWDC23 Session 10012 page (abstract, chapter summaries, code samples, and resource links)._
