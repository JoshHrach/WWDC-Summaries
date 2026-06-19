# What's New in App Store Connect
**WWDC24 · Session 10063** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10063/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11

## Overview
App Store Connect receives a broad set of improvements in 2024 organized around three themes: getting discovered, testing your app, and reaching customers. New tooling spans Featuring Nominations (a formal channel to pitch upcoming content to the App Store Editorial team), TestFlight invitation and enrollment improvements (rich invitation UI, tester criteria for public links, and per-link enrollment metrics), deep links on custom product pages, a new "Promote Your App" marketing asset generator in the App Store Connect iOS/iPadOS app, and expanded App Store Connect API coverage for analytics reports and enterprise provisioning.

The session uses a travel app called AwayFinder as its running example, demonstrating each feature in end-to-end walkthroughs of the App Store Connect web interface and the iOS app.

## Key Topics

### Featuring Nominations
Featuring Nominations is a new App Store Connect feature that provides a structured way to inform the App Store Editorial team about upcoming app launches, content additions, or enhancements before they ship. Nominations are created in the Nominations dashboard (left nav in App Store Connect), where developers provide: a nomination name, the type of change (New Content, App Enhancement, or New App Launch), a description of the feature, an expected publish date, related apps, relevant regions, and optional "Helpful Details" (accessibility, inclusivity values, etc.). Nominations can be created one at a time or uploaded in bulk via spreadsheet. The Editorial team reviews all submissions and considers them for featuring across platforms and regions. Developers can update nomination details at any time if plans change.

### TestFlight Invitation Enhancements
The TestFlight invitation experience is redesigned to help testers make an informed decision about joining a beta. The new invitation displays the app name and icon against a rich background using the app's primary colors, the developer name, app category, build expiration, app screenshots (from the most recent App Store-approved build), the beta app description, and test details. Beta App Description is now a required field in the Test Information section of App Store Connect. Testers who choose not to enroll can now explicitly decline the invitation.

### Tester Criteria for Public Links
When enabling a public link in an external TestFlight group, developers can now set enrollment criteria filtering on device platform (iPhone, iPad, Apple Vision Pro, etc.) and OS version ranges. Testers who click the link but do not meet the criteria are notified rather than enrolled. Criteria can be broad (multiple platforms, all supported OS versions) or narrow (a single platform and version range). The groups page shows the public link URL alongside the active criteria.

### Public Link Enrollment Insights
App Store Connect now shows per-public-link enrollment metrics: unique testers who viewed the app in TestFlight, accepted, declined, and those who were ineligible based on criteria. Each metric includes a percentage change over the prior 30-day period, enabling data-driven decisions about criteria tuning or link sharing strategy.

### Deep Links for Custom Product Pages
Custom product pages now support a Deep Link field accepting any Universal Link or custom URL scheme recognized by the app. When a user opens the app from that custom product page (or from an Apple Search Ads ad using it), the app is launched at the destination defined by the deep link. Deep links are assigned in App Store Connect during custom product page creation or editing, and appear as a reference URL in Apple Search Ads when the page is selected for an ad campaign.

### Promote Your App (App Store Connect iOS/iPadOS App)
A new "Promote Your App" section in the App Store Connect mobile app generates animated marketing assets for key moments: a new app launch, a new version release, or an App Store featuring. Each moment offers multiple visual styles with the app's name and icon. Developers copy the App Store product page URL and share it alongside the asset via the iOS share sheet. When an app is featured on the App Store, a push notification is sent to the developer with a link to generate a special featuring marketing asset.

### Additional API and Console Improvements
- Required screenshots reduced: App Store Connect on iOS/iPadOS now requires only one set of iPhone screenshots and one set of iPad screenshots (down from multiple size requirements).
- App Store Connect API now supports provisioning and user management for the Apple Developer Enterprise Program.
- 50 new App Analytics reports are available via the App Store Connect API, covering App Store engagement, downloads, sales, and app usage, enabling direct ingestion into developer-owned data pipelines.

## APIs & Frameworks

- `App Store Connect` — Apple's web portal for managing apps, builds, and distribution
- `App Store Connect API` — REST API for automating App Store Connect workflows
- Featuring Nominations **[NEW]** — in-portal nomination system for Editorial team consideration
  - Nomination types: New Content, App Enhancement, New App Launch **[NEW]**
  - Bulk nomination upload via spreadsheet **[NEW]**
  - Helpful Details field for accessibility/values callouts **[NEW]**
- `TestFlight` — Apple's beta testing distribution tool
  - Redesigned invitation UI **[NEW]** — rich card with app icon, screenshots, description, category, expiration
  - Beta App Description — now required field **[NEW behavior]**
  - App screenshots in TestFlight invitations **[NEW]** — sourced from latest App Store-approved build
  - Tester decline option **[NEW]** — testers can explicitly decline beta invitations
  - Tester criteria for public links **[NEW]** — filter enrollment by device platform and OS version range
  - Public link enrollment insights **[NEW]** — per-link metrics: views, accepted, declined, ineligible, with 30-day trends
- Custom Product Pages — existing App Store Connect feature; enhanced with:
  - Deep Link field **[NEW]** — Universal Link or custom URL scheme for in-app destination
  - Apple Search Ads integration **[NEW]** — deep link reference displayed in Ads campaign setup
- Promote Your App **[NEW]** — marketing asset generator in App Store Connect iOS/iPadOS app
  - Moment types: new launch, new version, App Store featuring **[NEW]**
  - Push notification for featuring events **[NEW]**
  - Animated asset styles with app icon and name **[NEW]**
- App Store Connect API — expanded **[NEW]**:
  - Apple Developer Enterprise Program provisioning and user management APIs **[NEW]**
  - 50 new App Analytics reports **[NEW]**: App Store engagement, downloads, sales, app usage
- Screenshot requirements reduction **[NEW]** — one iPhone set + one iPad set required (iOS/iPadOS submission)

## Code Highlights

This session covers App Store Connect web and mobile tooling exclusively. No SDK code samples were presented. Key developer workflows are:

Configure a Featuring Nomination in App Store Connect:
1. App Store Connect → left nav → Nominations → Get Started
2. Create Nomination: name, type (App Enhancement), description, expected publish date, related apps, regions, Helpful Details
3. Submit Nomination for Editorial review

Set tester criteria on a TestFlight public link:
1. TestFlight tab → external group → Invite Testers → Public Link
2. Enable criteria, select platform (e.g., iPhone, iOS 15–18), add Apple Vision Pro (all visionOS)
3. Publish; ineligible testers see a rejection message

Add a deep link to a custom product page:
1. App Store Connect → Custom Product Pages → select/create page
2. Enter Universal Link or custom URL scheme in the Deep Link field
3. Submit for review; link appears automatically in Apple Search Ads campaign setup

Download analytics reports via App Store Connect API:
```
GET https://api.appstoreconnect.apple.com/v1/analyticsReportRequests
```
Select from 50 new report types (engagement, downloads, sales, usage) and download via the API for ingestion into custom data pipelines.

## Takeaways

- Featuring Nominations gives developers a direct, structured channel to the App Store Editorial team—use it before major releases with firm dates, relevant regions, and meaningful Helpful Details to maximize consideration.
- The redesigned TestFlight invitation (rich card with screenshots, description, and category) combined with tester criteria and enrollment insights lets teams recruit the right testers and measure enrollment funnel efficiency without manual tracking.
- Custom product page deep links close the gap between marketing and in-app experience: users tap an ad or product page variant and land exactly where the feature lives, improving conversion and reducing friction.
- Promote Your App in the App Store Connect iOS app makes social media marketing self-serve—no design tools or assets needed for launch moments and featuring celebrations.
- The 50 new App Analytics API reports enable teams to move App Store data into their own analytics infrastructure, replacing manual CSV downloads with automated pipeline integration.

---
_Source: WWDC24 Session 10063 page (abstract, chapter summaries, transcript, and resource links)._
