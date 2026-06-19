# What's New in App Store Connect
**WWDC19 · Session 301** · [Watch](https://developer.apple.com/videos/play/wwdc2019/301/)

_Platforms:_ iOS 13, macOS Catalina 10.15, watchOS 6

## Overview
App Store Connect receives broad improvements across the entire app lifecycle in 2019: a new standalone Transporter app replaces Application Loader, TestFlight gains a complete in-app feedback system (screenshots, crash prompts, and onboarding), Arabic and Hebrew localization extends the App Store to 250+ million additional potential customers, App Analytics adds Mac App Store metrics and a new App Deletions metric, and a 24-Hour Dashboard provides hourly sales data without waiting until the next business day.

The session walks through the app lifecycle phase by phase — design and develop, upload, test, price, localize, submit for review, publish, and analyze — highlighting what changed at each step. Standalone watchOS apps (no iOS companion required) are supported end-to-end including TestFlight and App Store distribution.

## Key Topics

**Transporter App**
Application Loader is removed from Xcode 11. Its replacement, Transporter, is a dedicated Mac App (separate download, no Xcode installation required) that supports drag-and-drop IPA delivery, multi-package parallel validation, localized UI, and real-time delivery progress.

**Build Processing Improvements**
Post-upload validation emails now include the version number in the subject line, additional error context, and numeric error codes. A new Build Activity view in App Store Connect for iOS shows all recent builds with download and install sizes per device model. A new "Build Processing Changes" notification type is available to alert teams when processing completes.

**TestFlight Feedback**
A major new feature allowing testers to send structured feedback without leaving the app under test:
- New tester onboarding screen with developer-authored test notes on first launch
- Screenshot feedback: testers take a screenshot, tap "Share Beta Feedback", annotate, and submit with comments
- Crash feedback: automatic prompt after any crash, letting testers describe repro steps
- App Store Connect shows a Crashes tab and a Screenshots tab with device details (iOS version, model, screen resolution, battery, disk space), filterable by version/build/device
- Feedback can be downloaded as a zip file including crash logs and screenshots
- Feedback collection can be disabled/re-enabled at the tester group level
- Requires iOS 13; requires no SDK changes from developers

**TestFlight Beta Program**
Apple announces a TestFlight beta program allowing developers to receive early TestFlight builds via the Developer Downloads page.

**Localization — Arabic and Hebrew**
App Store and TestFlight now support Arabic and Hebrew with full right-to-left layout, bringing total language support to 39 languages. Enables reaching 250+ million customers in Arabic- and Hebrew-speaking regions.

**Pricing**
Korean won added; 45 unique currencies now supported across 155 territories. Subscription Offers feature (introduced earlier) allows discounted or free trial periods for targeted subscriber segments.

**App Analytics — New Metrics**
- Mac App Store support: Impressions, Product Page Views, App Units, Sales
- **App Deletions** metric **[NEW]**: count of uninstalls, viewable by source type
- **24-Hour Dashboard** **[NEW]**: hourly sales/IAP breakdown for the trailing 24 hours, eliminating next-business-day lag

**App Review Best Practices**
- Provide demo account credentials in review notes
- Explain non-obvious features and IAP in review notes
- Screenshots must accurately reflect the app in use on the correct device type
- Only request login when it is core functionality
- Purpose strings must clearly describe data usage
- Subscription terms must be clear and easy to understand

## APIs & Frameworks

**App Store Connect**
- Transporter (Mac App) **[NEW]** — replaces Application Loader for IPA delivery
- Build Activity view in App Store Connect for iOS **[NEW]** — all builds + per-device download/install size
- "Build Processing Changes" notification type **[NEW]**
- Numeric error codes in processing emails **[NEW]**

**TestFlight**
- In-app screenshot feedback flow **[NEW]** — no SDK required; iOS 13
- Crash feedback prompt **[NEW]** — automatic post-crash feedback dialog; iOS 13
- Tester onboarding screen with developer test notes **[NEW]**
- Crashes tab in TestFlight section of App Store Connect **[NEW]**
- Screenshots tab in TestFlight section of App Store Connect **[NEW]**
- Downloadable feedback zip files (crash logs + screenshots + metadata) **[NEW]**
- Feedback enable/disable toggle at tester group level **[NEW]**
- TestFlight localized in additional languages (including Arabic and Hebrew) **[NEW]**
- TestFlight Beta Program for early builds **[NEW]**

**App Store**
- Arabic and Hebrew language support with RTL layout **[NEW]**
- 39 total languages supported (up from 30)
- Korean won currency support **[NEW]** (45 currencies total)
- Standalone watchOS app distribution without iOS companion **[NEW]**
- App Store for Apple Watch **[NEW]**

**App Analytics**
- Mac App Store analytics: Impressions, Product Page Views, App Units, Sales **[NEW]**
- App Deletions metric for iOS **[NEW]**
- 24-Hour Dashboard with hourly granularity **[NEW]**

## Code Highlights

No SDK code changes are required for TestFlight Feedback — it is entirely platform-provided on iOS 13. Developers configure test notes in App Store Connect's TestFlight build settings.

For standalone watchOS apps, enable watchOS target in Xcode without requiring an iOS target; App Store Connect supports the resulting IPA natively.

## Takeaways
- TestFlight Feedback is opt-out and requires zero SDK work; any app distributed via TestFlight on iOS 13 automatically gains screenshot and crash feedback channels.
- Transporter is a permanent replacement for Application Loader — all teams should download it before upgrading to Xcode 11.
- The 24-Hour Dashboard and App Deletions metric together provide significantly faster signal on launch-day performance and user retention issues.
- Arabic and Hebrew RTL support in the App Store and TestFlight opens meaningful new markets with no code changes required; developer metadata localization in App Store Connect is the only required step.

---
_Source: WWDC19 Session 301 page (abstract, transcript, and resource links)._
