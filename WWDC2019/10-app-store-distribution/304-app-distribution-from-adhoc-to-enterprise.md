# App Distribution – From Ad-hoc to Enterprise
**WWDC19 · Session 304** · [Watch](https://developer.apple.com/videos/play/wwdc2019/304/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session provides a comprehensive tour of every app distribution model available to Apple developers, walking through the complete lifecycle of an app from prototype to public release and private enterprise deployment. Presenter Ashley Carroll uses a fictional app called "LunchControl" as the running example, illustrating how distribution needs evolve as an app and its audience grow.

The session distinguishes between two key roles—the user (who runs the app) and the customer (who purchases or deploys it)—and shows how identifying those roles leads directly to the correct distribution mechanism. Whether the audience is the general public, a small test group, or the employees of a specific company determines which program and workflow to use.

Custom Apps is highlighted as the modern replacement for the Apple Developer Enterprise Program for the vast majority of internal/private app distribution use cases, offering benefits including no certificate expiration worries, broader audience reach, and access to TestFlight and App Store infrastructure.

## Key Topics

**Personal Team (Free Apple ID)**
- Explore the SDK and sideload to personal devices; not a true distribution mechanism
- Limited capabilities: no CloudKit, Siri, or APNs; apps expire after a few days

**Ad Hoc Distribution**
- Requires Apple Developer Program membership
- Distribute to up to 100 registered devices per product family per year
- Device UDIDs must be registered on the developer portal; installs via Xcode, Apple Configurator, or over-the-air hosting
- Provisioning profiles expire annually; not scalable for large audiences

**TestFlight / App Store Beta**
- Up to 10,000 external testers; builds valid for 90 days
- External tester builds require App Review; internal team builds do not
- Managed entirely within App Store Connect

**App Store Distribution**
- For apps targeting the general public
- Supports bulk purchasing by organizations via Apple Business Manager and Apple School Manager (50% educational discount for 20+ licenses)
- Proper app versioning: avoid creating multiple store listings for the same app; consolidate into one binary
- Third-party developer workflows: assign Developer and Marketing roles only; keep Admin and App Manager roles internal to the client

**In-House Distribution (Apple Developer Enterprise Program)**
- Now reserved for edge cases: regions where Apple Business Manager is unavailable, specific government use cases, IP constraints
- Apps signed with org's distribution certificate; revocation stops all deployed apps immediately
- Certificates expire every 3 years; provisioning profiles annually; no TestFlight access

**Custom Apps**
- **[NEW]** Now supports distributing to your own employees (not just B2B)
- Hosted on App Store infrastructure; no expiration concerns
- Requires Apple Business Manager (or Apple School Manager, coming fall 2019)
- Customer provides their DEP ID and org name; developer lists app as private in App Store Connect Pricing & Availability
- Supports MDM distribution and redemption codes
- Apps must pass App Review; cannot change between public and private availability after submission

## APIs & Frameworks

- **Apple Developer Program** – membership for App Store and ad hoc distribution
- **Apple Developer Enterprise Program** – in-house distribution for qualified orgs
- **TestFlight** – beta distribution; up to 10,000 testers, 90-day build expiry **[NEW feature: external review required]**
- **App Store Connect** – app management, TestFlight, pricing/availability, user roles (Admin, App Manager, Developer, Marketing)
- **Apple Business Manager** – bulk app purchasing and license management for businesses; `business.apple.com`
- **Apple School Manager** – education bulk purchasing; Custom Apps support coming fall 2019 **[NEW]**
- **Custom Apps** – private App Store distribution via Apple Business Manager; now supports internal employee distribution **[NEW]**
- **MDM (Mobile Device Management)** – mechanism for deploying Custom App licenses to devices
- **Apple Configurator** – Mac App Store tool for tethered device provisioning
- **DEP ID** – organization identifier from Apple Business Manager enrollment, required for Custom App distribution setup
- **Redemption Codes** – alternative distribution mechanism for Custom App licenses

## Code Highlights

No code samples presented. The session is process- and policy-focused, covering portal configurations in App Store Connect, Apple Business Manager, and the Apple Developer website rather than SDK APIs.

## Takeaways
- Identify your user and customer first—that single decision drives the correct distribution mechanism.
- Custom Apps replaces in-house (Enterprise Program) distribution for virtually all private/internal app scenarios and now supports distributing to your own employees.
- Maintain a single App Store listing per app concept; versioning via separate bundle IDs creates a poor user experience.
- When working as a third-party developer for a client, submit under the client's App Store Connect account and limit your own access to Developer and Marketing roles only.

---
_Source: WWDC19 Session 304 page (abstract, full transcript, and resource links)._
