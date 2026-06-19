# What's New in Managing Apple Devices

**WWDC26 · Session 206** · [Watch](https://developer.apple.com/videos/play/wwdc2026/206/)

_Platforms:_ iOS 27 · macOS 27 · iPadOS 27 · tvOS · watchOS · visionOS

## Overview

This session surveys the 2026 updates across Apple's device management ecosystem, spanning Apple Business Manager / Apple School Manager services, declarative device management protocol additions, app management, identity integrations, and education features. The overarching theme is moving fully to declarative management — where the device proactively maintains its own desired state rather than polling for imperative commands — while simultaneously broadening the scope of what declarative management can control.

Apple Business Manager expands significantly: it is now available in over 200 countries and exposes new REST APIs for Blueprints, configurations, users, and audit events. A new volume licensing mechanism enables App Store subscription purchases at scale. On the device management side, managed migration for Mac allows enrollment and settings to survive a data migration to a new machine — a long-standing pain point for enterprise deployments.

The identity integrations chapter covers Platform SSO enhancements on macOS 27, including a redesigned login and unlock experience and an option for administrators to require Touch ID as a built-in second factor. The education chapter introduces Authenticated Guest Mode on Shared iPad and a new guided browsing feature in the Classroom app that lets teachers lock students to specific websites or a single tab.

## Key Topics

### Apple Services
- **[NEW]** Apple Business Manager available in 200+ countries.
- **[NEW]** Apple Business Manager REST API: Blueprints, configurations, users, audit events.
- **[NEW]** Volume licensing for App Store subscriptions (group/organization purchases).
- Apple School Manager: parallel updates for education deployments.

### Device Management
- Declarative Device Management (DDM) established as the new standard; legacy MDM commands deprecated path forward.
- **[NEW]** Managed migration for Mac: migrate device data to a new Mac while preserving enrollment and MDM settings/policies.
- New declarative configurations and status items covering additional device subsystems.

### App Management
- **[NEW]** Declarative app configuration comes to macOS 27.
- **[NEW]** Hardware-bound configuration keys and Managed Device Attestation for macOS app config.
- **[NEW]** Package file cleanup on app removal (managed apps leave no residual files).
- **[NEW]** New privacy settings controls via declarative configuration.

### Identity Integrations
- **[NEW]** Platform SSO redesigned login and unlock experience on macOS 27.
- **[NEW]** Administrator option to require Touch ID as a built-in second authentication factor via Platform SSO.
- Continued Kerberos SSO and identity provider integration support.

### Education
- **[NEW]** Authenticated Guest Mode on Shared iPad — guest sessions with institutional identity authentication.
- **[NEW]** Guided browsing in Classroom app — teachers can lock students to specific websites or a single Safari tab.
- Classroom app and Schoolwork integration updates.

## APIs & Frameworks

**Declarative Device Management (DDM)**
- DDM protocol — device-driven state maintenance, replaces legacy MDM command polling
- **[NEW]** Managed migration declaration — preserves enrollment across Mac data migrations
- **[NEW]** Declarative app configuration for macOS 27
- **[NEW]** Hardware-bound configuration keys (Managed Device Attestation)
- **[NEW]** Package cleanup declaration — automatic file removal on app uninstall
- **[NEW]** Privacy settings declarations

**Apple Business Manager / Apple School Manager**
- **[NEW]** Available in 200+ countries
- **[NEW]** REST API: Blueprints endpoint
- **[NEW]** REST API: Configurations endpoint
- **[NEW]** REST API: Users endpoint
- **[NEW]** REST API: Audit events endpoint
- **[NEW]** Volume licensing for App Store subscriptions

**Platform SSO (macOS 27)**
- **[NEW]** Redesigned login and unlock experience
- **[NEW]** Touch ID as second authentication factor option
- `AuthenticationServices` framework — underlying SSO mechanism
- Identity provider (IdP) integration (Entra ID, Okta, etc.)

**Education**
- Shared iPad — **[NEW]** Authenticated Guest Mode
- Classroom app — **[NEW]** Guided browsing (lock to website/tab)
- Schoolwork app updates

**Related Sessions (WWDC26)**
- Secure your apps with App Attest (201)
- What's new in assessment on macOS (230)
- Offer subscriptions to groups and organizations (391)

## Code Highlights

No code samples in this session — it is an MDM protocol and service management overview. API integration is done via REST (Apple Business Manager API) and MDM profile/declaration payloads (XML/JSON).

Example DDM declaration concept (not shown in session, for reference):
```json
{
  "Type": "com.apple.configuration.app.managed",
  "Payload": {
    "AppIdentifier": "com.example.myapp",
    "Configuration": { "serverURL": "https://api.example.com" },
    "HardwareBound": true
  }
}
```

## Takeaways

- Declarative Device Management is now the definitive standard; MDM vendors and enterprise IT should prioritize migration away from legacy command-based MDM for all new capability work.
- The Apple Business Manager REST API unlocks programmatic management of the full device and user lifecycle — previously only available through the UI or limited legacy APIs.
- Hardware-bound configuration keys via Managed Device Attestation close a meaningful security gap in enterprise app configuration delivery on macOS.
- Authenticated Guest Mode on Shared iPad and Guided Browsing in Classroom are high-value education features that reduce the friction of shared-device classroom deployments.

---
_Source: WWDC26 Session 206 page (abstract, chapter summaries, and resource links)._
