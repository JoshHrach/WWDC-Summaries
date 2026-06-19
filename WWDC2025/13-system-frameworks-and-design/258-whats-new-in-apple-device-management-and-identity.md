# What's New in Apple Device Management and Identity
**WWDC25 · Session 258** · [Watch](https://developer.apple.com/videos/play/wwdc2025/258/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26

## Overview
This session targets IT administrators, MDM developers, and identity providers. It covers a broad set of improvements to Apple Business Manager and Apple School Manager (ABM/ASM), Declarative Device Management, app management, and identity integration including Platform SSO and a new Tap to Login feature using NFC. The session also announces device management migration between MDM servers, ABM/ASM APIs for device inventory, and a new ManagedApp framework for app-level configuration.

## Key Topics

### Apple Business Manager / Apple School Manager Updates
- Administrators can now download a list of personal Apple Accounts on a domain and enforce that only Managed Apple Accounts can sign in on org-owned devices — no MDM dependency
- New **ABM/ASM APIs for Organizations**: REST endpoints for device inventory data and MDM server assignment (query devices, assign to MDM, get batch activity status)
- New device inventory fields: Bluetooth/Wi-Fi MAC addresses for iPhone and iPad (coming later this year), and AppleCare coverage information
- **Device Management Migration**: IT can reassign iPhone, iPad, or Mac to a new MDM server and set a deadline; users are notified and guided through migration; if deadline passes, migration is automatic; old configs removed, new ones installed; app data preserved via `await device configured` + `InstallApplication`
- Apple Configurator for iPhone can now add Apple Vision Pro to an organization (previously Automated Device Enrollment only)
- visionOS now supports skipping panes in Setup Assistant (new skip keys in device management documentation)
- Account-driven enrollment: MDM server can now configure the service discovery URL as a fallback when the well-known endpoint isn't on the org's domain

### Device Management (Declarative)
- **Software update management via Declarative Device Management** now supported on visionOS and tvOS (previously iOS/iPadOS/macOS only); deprecated older MDM-based software update management
- **Safari management**: new declarative configuration for bookmarks and default homepage; all Safari restrictions consolidated into Declarative Device Management
- **Apple Intelligence restrictions** for visionOS (writing tools, notification summaries, image playground)
- **Return to Service improvements**: iPhone/iPad can now preserve managed apps when reset (new `cloud configuration` key); visionOS gets Return to Service with a "Reset for Next User" Control Center option and Digital Crown reset at lock screen
- Additional new restrictions: battery health info for iPad, default apps for messaging and calling, per-SIM messaging/FaceTime restrictions, AirPods/Beats temporary use, FQDN support in network relay profile, new Network Extension URL Filtering API

### App Management
- Per-app update control (enforce/disable automatic update, pin to specific version) **[NEW]** — iOS/iPadOS managed apps
- Real-time app installation status and version info via status channel
- Cellular download restriction per app
- macOS Tahoe: App Store apps, custom apps, and packages deployable via Declarative Device Management (required or optional); `ManagedAppDistribution` framework for Mac coming later this year
- **ManagedApp framework** (iOS/iPadOS 18.4, visionOS 2.4) — allows apps to securely receive configurations, passwords, certificates, identities, API access tokens, and hardware-bound keys from MDM

### Identity Integrations: Platform SSO
- Platform SSO registration now happens during Automated Device Enrollment Setup Assistant (not post-setup); users authenticate with their IdP, local account is created with synced password or Secure Enclave-backed key; profile picture synced from IdP; blocks enrollment if SSO fails
- **Authenticated Guest Mode** (shared Mac use): users log in from the login window with cloud identity (password or SmartCard); all data wiped on logout; pairs with auto-advance for silent SSO registration at Setup Assistant
- **Tap to Login** — users tap iPhone or Apple Watch (with corporate badge / school ID Wallet pass containing an Access Key) on a Mac configured for Authenticated Guest Mode; Mac requires an external NFC reader; Access Keys stored in Secure Enclave; Express Mode allows login without waking/unlocking device

## APIs & Frameworks

**Apple Business Manager / Apple School Manager APIs** **[NEW]**
- Device inventory query and MDM server assignment REST endpoints
- API account creation (Administrators / Site Managers only) with Private API key

**Declarative Device Management**
- Software update declarations — now supported on visionOS, tvOS **[NEW]**
- Safari bookmark and default homepage declarative configurations **[NEW]**
- Managed app declarations — per-app update behavior keys **[NEW]**: enforce/disable automatic update, version pinning, cellular download restriction
- Return to Service cloud configuration key for app preservation **[NEW]**
- visionOS Setup Assistant skip keys **[NEW]**
- visionOS Return to Service ("Reset for Next User" Control Center option) **[NEW]**

**ManagedApp framework** (iOS/iPadOS 18.4, visionOS 2.4) **[NEW]**
- Secure app configuration delivery (settings, passwords, certificates, identities, API tokens, hardware-bound keys)

**ManagedAppDistribution framework** (macOS Tahoe, coming later this year) **[NEW]**
- Self-service app distribution for Mac

**Platform SSO**
- Setup Assistant integration during ADE **[NEW]**
- Authenticated Guest Mode **[NEW]**
- Tap to Login via NFC and Apple Wallet Access Keys **[NEW]**
- Access Keys (Secure Enclave-backed, Express Mode) **[NEW]**

**Network Extension**
- URL Filtering API **[NEW]** (see "Filter and tunnel network traffic with NetworkExtension" session)

## Code Highlights
This session is primarily a feature overview with no inline code samples. For implementation details, refer to the Apple School Manager / Apple Business Manager API documentation, `ManagedApp` framework documentation, and the "Get to know the ManagedApp Framework" WWDC25 session.

## Takeaways
- Begin migrating software update management to Declarative Device Management now — the older MDM approach is deprecated and will be removed in a future release.
- Adopt the ManagedApp framework in your iOS/iPadOS apps to enable secure MDM-delivered configuration, tokens, and hardware-bound keys.
- Evaluate Authenticated Guest Mode + Tap to Login for shared-use Mac environments (healthcare, retail, education) where fast, secure login is critical.
- Use the new ABM/ASM APIs to automate device inventory queries and MDM server assignment in your custom workflows.
- Plan ahead for device management migration if you need to move between MDM servers — the new built-in tooling avoids full device wipes.

---
_Source: WWDC25 Session 258 page (abstract, chapter summaries, code samples, and resource links)._
