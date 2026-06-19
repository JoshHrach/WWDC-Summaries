# What's New in Managing Apple Devices
**WWDC20 · Session 10639** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10639/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
WWDC20 brought a major wave of device management capabilities to macOS, closing long-standing gaps between Mac and iOS/iPadOS MDM. Many management features previously exclusive to iOS — including Managed Apps, software update deferral controls, and bootstrap token enhancements — arrived on macOS Big Sur. On the iOS/iPadOS side, Shared iPad gained business-oriented features, new security controls addressed data leakage, and VPN gained per-account configuration.

The session also covered important changes to deployment workflows, including randomized serial numbers, downloaded profile quarantine, stricter command-line profile installation, and networksetup tool privilege separation — all requiring MDM vendor attention.

## Key Topics

### macOS Deployment Enhancements
- **Automated Device Enrollment** — zero-touch enrollment with modern identity provider (Azure AD, Okta, Ping) support; pre-population of user full name and short name from IdP credentials; control over user channel management enablement
- **Auto-Advance for Automated Device Enrollment** — plug in power + Ethernet; all Setup Assistant screens are skipped automatically, reaching login in seconds (requires DHCP)
- **Lights Out Management (LOM)** **[NEW]** — remotely start up, shut down, and reboot Mac Pros via MDM commands; requires an MDM-enrolled LOM controller Mac on the same subnet

### macOS Management
- **Supervision for User-Approved MDM** **[NEW]** — any user-approved MDM-enrolled Mac is now treated as supervised, enabling activation lock control, bootstrap tokens, local user management, profile replacement, supervised restrictions, and software update scheduling
- **Managed Software Updates for macOS** **[NEW]** — force clients to accept updates and restart; defer major releases and non-OS updates up to 90 days; software update catalog removed; Ignore flag removed for major updates
- **Managed Apps for macOS** **[NEW]** — remove apps via MDM or on unenrollment; managed app configuration/feedback; convert unmanaged to managed apps
- **Content Caching** — now supports internet recovery (full 6 GB recovery image cached); new Content Caching Information MDM command exposes metrics (registration state, cache pressure, bytes served)

### macOS Security
- **Bootstrap Token Enhancements** — enables secure token and FileVault boot for network/mobile accounts; supports authorized software updates and kernel extensions
- **Downloaded Profiles** **[NEW for macOS]** — profiles downloaded outside MDM appear in a quarantine area in System Preferences for 8 minutes; user must manually confirm installation
- **Profiles Command-Line Tool** — complete profile installation via Terminal no longer allowed; treated as downloaded profile requiring manual confirmation in System Preferences
- **networksetup Tool Hardening** — standard users can only read settings, toggle Wi-Fi, and change access point; admin password requirement honored; admins can bypass with sudo
- **Randomized Serial Numbers** **[NEW]** — new devices get random 10-character serial numbers (replacing identifiable 12-character format); MDM vendors must handle both formats simultaneously

### iOS/iPadOS Deployment
- **Apple Configurator** — now supports Apple School/Business Manager app and book locations; `cfgutil` handles larger device counts
- **Setup Assistant Payload for iOS** **[NEW]** — skip keys applied during future upgrades on all supervised devices, not just ASM/ABM enrolled
- **Shared iPad for Business** — configurable via Apple Business Manager; managed Apple ID sign-in; federated auth with Azure AD; SSO extension support
- **Shared iPad Enhancements** — dynamic cached user counts (storage per user instead of fixed count); delete all users at once; queries for estimated resident users and quota size; **Temporary Session** (guest use without account; data deleted on sign-out)

### iOS/iPadOS Management
- **Non-Removable Managed Apps** **[NEW]** — mark specific managed apps as non-removable; prevents user deletion or offloading while allowing other app management
- **Shortcuts + Managed Open In** **[NEW]** — Shortcuts respects Managed Open In policy; stops running when data flow violates device policy
- **Notification Preview Type** **[NEW]** — new `PreviewType` key in Notification Settings payload; values: never, always, or whenAuthenticated; supervised devices only

### iOS/iPadOS Security
- **Per Account VPN** **[NEW]** — associate Contacts, Calendars, and Mail domain accounts with a specific VPN configuration
- **Encrypted DNS Management** **[NEW]** — manage DNS-over-HTTPS or DNS-over-TLS settings via MDM (DoH/DoT); no VPN required
- **Private Wi-Fi MAC Address** — iOS 14 uses random MAC address per network by default; new Wi-Fi payload key to disable per-network; falls back to real MAC if network join fails

## APIs & Frameworks

- **MDM Protocol** — device management command/response protocol
  - `LightsOutManagement` command **[NEW]** — remote power control for Mac Pro
  - `ContentCachingInformation` command **[NEW]** — cache metrics query
  - `SetTimezone` command **[NEW]** — set device time zone without Location Services
  - Managed App commands for macOS (install, remove, convert) **[NEW]**
  - Managed Software Update commands for macOS **[NEW]** (force update + restart, defer up to 90 days)
  - `DeleteUser` / list local users commands for macOS **[NEW]**
- **Configuration Profile Payloads**
  - `LightsOutManagement` payload **[NEW]**
  - `SetupAssistant` payload for iOS **[NEW]**
  - Wi-Fi payload — `DisableAssociationMACRandomization` key **[NEW]**
  - Notification Settings payload — `PreviewType` key **[NEW]**
  - VPN payload — Per Account VPN domain keys **[NEW]**
  - Encrypted DNS payload **[NEW]** — DoH/DoT configuration
- **Device Management framework** (`DeviceManagement`) — MDM server-side protocol reference
- **Network Extension** framework — VPN and DNS configuration
- **Apple Configurator 2** — mass configuration tool; location-aware app/book assignment **[NEW]**
- **Apple Business Manager / Apple School Manager** — zero-touch enrollment, location management
- **Bootstrap Token** — reserved FileVault encryption key for supervised Macs
- **Content Caching** — network-local caching service; internet recovery support **[NEW]**

## Code Highlights

No source code samples were presented. The session focused on MDM protocol payloads and commands. Key implementation notes:
- MDM vendors must accept both 12-character (old) and 10-character (new random) serial number formats simultaneously.
- The `ContentCachingInformation` MDM command should not be polled at high frequency — it is not designed as a real-time activity monitor.
- Per Account VPN requires replacing domain-level keys in the Domain payload; no new framework API is required on the device side.

## Takeaways

- User-approved MDM-enrolled Macs are now treated as supervised in macOS Big Sur, eliminating a major capability gap between enrollment methods.
- Lights Out Management enables true remote power control of Mac Pro at scale, a long-requested enterprise feature.
- MDM vendors must update serial number parsing to handle the new random 10-character format alongside existing 12-character serial numbers.
- iOS 14 enables per-account VPN and encrypted DNS management via MDM, giving organizations finer-grained network security control without requiring device-wide VPN.

---
_Source: WWDC20 Session 10639 page (abstract, transcript, and resource links)._
