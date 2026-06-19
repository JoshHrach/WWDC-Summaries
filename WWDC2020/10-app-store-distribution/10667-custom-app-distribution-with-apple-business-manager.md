# Custom App Distribution with Apple Business Manager
**WWDC20 · Session 10667** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10667/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session provides a comprehensive guide to distributing custom apps through Apple Business Manager (ABM) and Apple School Manager (ASM), covering the full lifecycle from development and App Review submission through to enterprise device deployment and end-user activation. It addresses three distinct audiences — developers, IT administrators, and end users — explaining the benefits and responsibilities at each stage.

Custom apps are private App Store apps that only appear to organizations the developer explicitly authorizes. They go through the standard App Review process, can be tested via TestFlight, and are distributed via MDM just like volume-purchased App Store apps — but with the ability to include organization-specific features, branding, and functionality. In WWDC20, Apple extended Custom Apps support to Apple School Manager customers.

## Key Topics

**Distribution Options Compared**
- Enterprise (Ad-Hoc) Distribution: internal only, no App Review, certificates expire — now the legacy path.
- Custom App Distribution: goes through App Review, certificates don't expire, preferred path for internal enterprise deployment where ABM is available.
- Public App Store: broad audience, no private distribution.

**Custom Apps Extended to Apple School Manager**
Custom Apps are now supported in both Apple Business Manager and Apple School Manager, using the same workflow.

**Developer Workflow**
1. Enroll in the Apple Developer Program (Organization account with DUNS number required).
2. Submit the app in App Store Connect, providing metadata, screenshots, pricing, and region availability.
3. Set distribution type to "Custom App" and specify the org name/ID of authorized purchasers.
4. Submit for App Review with demo credentials and detailed notes.
5. After approval, authorized organizations can purchase the app through ABM.

**App Review Best Practices for Custom Apps**
- Provide demo credentials or a built-in demo mode with sample data so reviewers can access all features.
- Submit detailed metadata and notes about the app's target audience and market segment.
- Use only public APIs; avoid private/deprecated frameworks.
- Adhere to Apple's privacy guidelines for user data.
- Coordinate App Review timelines with customer deployment schedules (avoid holidays and product launches).
- Custom apps cannot be converted from existing consumer App Store apps — they require a new Bundle ID.

**Managing Multiple Variants**
- Minimize app variants by using App Configuration and user-authorization-based rules for branding and customization.
- If separate apps are needed (different Bundle IDs), collect shared code into reusable frameworks.
- Coordinate major releases with customer deployment waves; once a new version is published, the prior version cannot be deployed by customers.

**IT Administrator Workflow**
1. Enable Custom Apps in ABM/ASM under Settings.
2. Provide the exact org name and org ID (as shown in ABM) to the developer.
3. Connect ABM to an MDM server by exchanging public keys and enrollment tokens.
4. Download a location token for apps and books; upload to MDM.
5. Purchase custom app licenses in ABM and assign to a Location.
6. MDM distributes licenses to devices and users.

**MDM Managed App Capabilities**
- Assign/revoke/reassign licenses as needed.
- Manage app updates centrally; restrict end-user updates until IT is ready to deploy.
- Defer OS updates up to 90 days.
- Support device-based and user-based license assignment.
- Managed App restrictions prevent content flowing from managed to unmanaged sources.
- Key MDM capabilities: `Custom app support`, `Restrict app updates`, `Defer OS updates`.

**Federated Authentication and User Enrollment**
- Federated Authentication lets employees use existing corporate credentials in ABM.
- User Enrollment (introduced in 2019) supports BYOD with lightweight corporate control.

**Note on Volume Purchase Program**
The legacy Volume Purchase Program will no longer be available starting December 1, 2020. Organizations should migrate to ABM/ASM.

## APIs & Frameworks

### Apple Business Manager / Apple School Manager
- Apple Business Manager (ABM) — centralized platform for device enrollment, app purchasing, account management
- Apple School Manager (ASM) — same capabilities as ABM for education customers **[NEW: Custom Apps support]**
- Custom App distribution — private App Store distribution to authorized organizations
- Location Tokens — enable MDM to distribute app licenses
- Federated Authentication — corporate identity integration with ABM

### App Store Connect
- Custom App submission — distribution type selector for private vs. public app
- Organization authorization — specify org name and org ID for purchasing access
- TestFlight — beta distribution for custom apps (same as public apps)
- App Review — same process as public apps; provides quality vetting

### MDM (Mobile Device Management)
- Automated Device Enrollment (ADE) — zero-touch setup for organization-owned devices
- Managed Apps — license assignment/revocation, managed app data restrictions
- App update management — MDM-controlled update initiation; restrict self-service updates
- OS update deferral — defer OS updates up to 90 days via MDM restriction
- Device-based vs. user-based license assignment
- Profile Manager — Apple's reference MDM implementation (used in demo)

### Developer Program Requirements
- DUNS number — required for Organization Developer Program enrollment
- Paid Apps Agreement — required for paid custom apps
- App Groups / Shared frameworks — recommended for code reuse across custom app variants

## Code Highlights

No code samples were provided in this session. The session is focused on App Store Connect configuration, ABM administration, and MDM setup workflows.

## Takeaways
- Custom App distribution via Apple Business Manager is now the preferred replacement for legacy Enterprise (Ad-Hoc) distribution for internal deployments; it includes App Review vetting and non-expiring distribution certificates.
- Custom Apps are now supported in Apple School Manager as well as Apple Business Manager.
- Developers should minimize app variants by using App Configuration and user-based rules for customization rather than creating separate Bundle IDs; separate apps require separate review and build trains.
- MDM administrators can centrally control when custom app updates are deployed and defer OS updates for up to 90 days, giving IT control over rollout timing.

---
_Source: WWDC20 Session 10667 page (abstract, chapter summaries, code samples, and resource links)._
