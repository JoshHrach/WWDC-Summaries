# Create Your Privacy Nutrition Label
**WWDC22 · Session 10167** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10167/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, watchOS (App Store Connect — all platforms)

## Overview
Privacy Nutrition Labels, introduced in App Store Connect, provide users with a clear, standardized summary of an app's data collection and use practices on the app's product page. This session walks through the complete process of creating and maintaining an accurate label: building a data inventory, answering questions in App Store Connect, understanding key policy definitions, and keeping the label up to date as the app evolves.

The session emphasizes that labels must cover all data collected from the app — including data collected by third-party SDKs, analytics tools, and advertising networks — and explains the three disclosure dimensions for each data type: use case, linkage to identity, and use for tracking. It also covers the optional disclosure policy for infrequent, supplemental data collection and practical guidance on data minimization.

## Key Topics

### What the Label Covers
A Privacy Nutrition Label is organized into three sections:
1. **Data Used to Track You** — data linked to the user and shared with third parties for advertising or shared with data brokers
2. **Data Linked to You** — data associated with an account, device, or profile
3. **Data Not Linked to You** — collected data that is not associated with any identity

If no data is collected, an alternative "No data collected" label is shown.

### Building a Data Inventory
Recommended process before entering data in App Store Connect:
- List all app features, then identify what data each feature collects and retains
- Consult marketing, legal, and engineering stakeholders
- Use App Privacy Report or a network proxy to observe domains contacted
- Review server-side database schemas and access controls
- Obtain privacy documentation from all third-party SDK and analytics partners
- Use the inventory to identify opportunities to minimize data collection

### App Store Connect Label Creation Flow
1. Declare whether the app collects data (data transmitted off-device and retained beyond the immediate request)
2. Select data categories collected (e.g., email address, phone number, payment info, location, product interaction)
3. For each category, specify: use cases (app functionality, analytics, product personalization, advertising, etc.), identity linkage, and tracking usage
4. Preview and publish the label (published independently of app updates)

### Key Definitions
- **Collected**: data transmitted off-device and accessible for longer than needed to service a real-time request (server logs, profiles, and analytics all count)
- **Linked to identity**: data associated with an account, device, or user profile
- **Tracking**: linking data from the app about a user/device with third-party data for targeted advertising or advertising measurement, or sharing with a data broker

### Data Type Examples
- **IP address**: disclose based on actual use — declare Location if used for location inference or local content
- **Product Interaction**: in-app screen views and feature interactions
- **Browsing History**: activity in an in-app browser (not part of the app's core UI)
- **Search History**: any searches performed within the app

### Optional Disclosure
Data collection that is infrequent, optional, clearly disclosed at submission time, independent from core functionality, and not used for tracking or advertising may qualify for optional disclosure. Feedback forms and report-a-problem flows are typical examples.

### Keeping the Label Updated
Labels can be updated at any time without releasing a new app version. Re-evaluate when: adding new features, integrating new SDKs/partners, or changing how existing data is used.

## APIs & Frameworks

This session covers policy and process; there are no code-level APIs. The relevant platform for label management is:

### App Store Connect
- **App Privacy section** — accessible to account holders, admins, and app managers
- Data category declarations (e.g., Contact Info, Health & Fitness, Financial Info, Location, Usage Data, Diagnostics, Identifiers, Purchases, User Content, Browsing History, Search History, Sensitive Info)
- Use case declarations: App Functionality, Analytics, Product Personalization, App Performance, Other Purposes, Third-Party Advertising, Developer's Advertising or Marketing
- Identity linkage: Linked to You / Not Linked to You
- Tracking disclosure: Used to Track You / Not Used for Tracking

### Related Frameworks (informational)
- `AppTrackingTransparency` — required for apps that use data for tracking; must request permission before tracking (see Session 10166)
- App Privacy Report (iOS Settings > Privacy & Security > App Privacy Report) — useful for auditing domains contacted by the app

## Code Highlights
No code samples — this is a policy and process session.

## Takeaways
- All data collected from your app must be disclosed — including data collected by SDKs and third-party partners. Contact SDK providers for their privacy documentation.
- Data "collection" is defined broadly: any data transmitted off-device and retained beyond serving the immediate request counts, regardless of whether users consented elsewhere.
- Labels are published independently of app updates; keep them current when features, data practices, or SDK integrations change.
- Use the inventory process as an opportunity to minimize data collection, process data on-device, and store data unlinked to identity where possible.

---
_Source: WWDC22 Session 10167 page (abstract, chapter summaries, code samples, and resource links)._
