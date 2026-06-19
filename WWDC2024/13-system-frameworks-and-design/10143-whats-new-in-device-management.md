# What's New in Device Management
**WWDC24 · Session 10143** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10143/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11

## Overview
This session covers the major 2024 updates to Apple's device management ecosystem: Apple Business Manager and Apple School Manager enhancements (domain capture, Activation Lock management, Managed Apple Account conversion), a new unified software update declaration that replaces legacy MDM commands, Safari extension management, comprehensive visionOS 2 MDM support (including Automated Device Enrollment), Mac-specific additions (executable service configs, disk management, Platform SSO improvements), iPhone/iPad updates (eSIM restrictions, app locking/hiding, Stolen Device Protection exception), and education-focused features (Classroom improvements, Schoolwork 3.0, Assessment Mode Multi-App).

## Key Topics

### Apple Business Manager and Apple School Manager
**Domain capture and Managed Apple Accounts**: IT admins can now limit new Apple Account creation on their domain to Managed Apple Accounts only, and can capture existing personal Apple Accounts using their domain without requiring an Identity Provider. Captured accounts may optionally convert to Managed Apple Accounts; if no action is taken in 30 days, they remain personal and are auto-renamed.

**Activation Lock management**: Organization-owned devices (iPhone, iPad, Mac, Apple Watch, Vision Pro) can now have Activation Lock turned off directly from Apple Business Manager or Apple School Manager, including accounts where a user enabled Activation Lock with their personal Apple Account before the Mac was enrolled in MDM.

### Software Update Management (New Declaration)
A new `SoftwareUpdate` declarative configuration replaces all legacy MDM software update commands, profiles, and restrictions. Works on supervised devices running iOS/iPadOS 18 and macOS 15. New capabilities include: notification behavior control (show notifications only one hour before enforcement), restart countdown management, and **beta update management** for AppleSeed for IT beta programs. Organizations can remotely enroll devices into beta programs using an organization token (no Apple Account sign-in required in Settings), and defer or enforce beta releases alongside production releases.

The beta enrollment flow: administrator enrolls at beta.apple.com/it, MDM checks the OS beta enrollment tokens endpoint (authenticated via OAuth), and receives program tokens. During ADE (iOS/iPadOS 17.5+, macOS 14.5+), MDM can return an HTTP 403 with `RequireBetaProgram` dictionary (Description + Token) to set the beta program during Setup Assistant.

### Safari Extension Management (New)
A new Safari extension configuration works on iOS, iPadOS, and macOS, supporting: defining which extensions are allowed (user can toggle on/off); forcing extensions always on or always off; configuring extension website access by domain and subdomain; and all of the above also works with Safari Private Browsing. Users see a visual indicator for managed extensions.

### visionOS 2 MDM
visionOS 2 adds **Automated Device Enrollment** (zero-touch enrollment via Setup Assistant, enables supervision). Devices associated with an Apple Customer Number appear in Apple Business Manager/Apple School Manager. Apps and Books for Organizations API now includes visionOS compatibility information. New MDM commands: `DeviceConfigured`, `DeviceLock`, various Settings sub-commands. New restrictions: Managed Open-In, account modification, `allowCamera` (reimagined — screenshot background removal when restricted). Passcode policy, Domains, Web Content Filter payloads all supported. Managing Vision Pro is now as straightforward as managing iPhone or iPad.

### Mac: Executable Service Configs and Disk Management
**Service configuration files**: Now supports executable files in addition to config files (sudo, PAM, SSH), enabling tamper-resistant delivery of IT management tools and scripts via zip archive. `launchd` configuration files can also be installed via background task services configuration, providing secure tamper-resistant storage for background tasks.

**Disk management configuration**: New declaration lets IT admins manage external and network storage — allow, disallow, or restrict to read-only. Replaces the previously deprecated media management payload.

**Platform SSO improvements**: IdP authentication can now unlock FileVault. Login policies can require IdP authentication across FileVault, login window, and lock screen. New security options include HPKE. `AllowOfflineGracePeriod` and `AllowTouchIDOrWatchForUnlock` provide flexible exceptions. Settings > Profiles section renamed to "Device Management" and moved under General.

### iPhone and iPad
- Two new eSIM restrictions: `forcePreserveeSIMOnErase` (prevent eSIM removal on local erase) and `alloweSIMOutgoingTransfers` (control eSIM transfer to a new device)
- Network slicing + per-app VPN: traffic from managed apps routes to the specified 5G network slice while maintaining VPN benefits
- Multiple Private Cellular Network payloads: up to five private 5G/LTE networks
- App locking/hiding: iOS/iPadOS 18 users can lock apps (require Face ID/Touch ID/passcode) and hide apps from Home Screen; organizations can restrict this per-supervised device or per managed app
- Stolen Device Protection: MDM enrollment on a newly set up device (without familiar locations) will not trigger a security delay for the first 3 hours after Stolen Device Protection is enabled
- New team identity for in-house apps: requires a device restart once per new team ID (existing trusted identities migrated)

### Education
- Easy Student Sign-In and Unmanaged Nearby Classes in Classroom now controllable via Access Management in Apple School Manager
- **Schoolwork 3.0** (iPadOS 17.5+): assessment scanning/import (Pages, Numbers, Keynote, Google Suite, PDF), teacher scoring with per-question analytics
- **Assessment Mode Multi-App** (iPadOS 17.6+): secondary apps (notepads, spreadsheets, assistive apps, Calculator) can run alongside assessment apps; Calculator for iPad supports MDM configuration (disable Scientific mode, disable Math Notes) for standardized testing

## APIs & Frameworks

**MDM Protocol**
- Automated Device Enrollment for visionOS 2 **[NEW]**
- Device Assignment API Collection — new Vision Pro values **[NEW]**
- `DeviceConfigured`, `DeviceLock`, Settings sub-commands for visionOS **[NEW]**
- `Apps and Books for Organizations API` — visionOS compatibility info **[NEW]**
- OS beta enrollment tokens endpoint (OAuth-authenticated) **[NEW]**
- `RequireBetaProgram` dictionary in ADE HTTP 403 response **[NEW]**
- `forcePreserveeSIMOnErase` restriction **[NEW]**
- `alloweSIMOutgoingTransfers` restriction **[NEW]**
- Managed app locking/hiding per-app restriction **[NEW]**
- iPhone Mirroring restrictions **[NEW]**
- FaceTime Remote Control restrictions **[NEW]**

**Declarative Device Management**
- New unified `SoftwareUpdate` declaration (replaces legacy commands/profiles/restrictions) **[NEW]**
  - Notification behavior control (one hour before enforcement)
  - Beta update management with `Beta` dictionary
  - Declarative status report for beta program enrollments
- Safari extension configuration **[NEW]** (allowed/blocked extensions, forced on/off, website access by domain)
- Executable service configuration files for Mac **[NEW]**
- `launchd` background task services configuration **[NEW]**
- Disk management configuration (replaces deprecated media management payload) **[NEW]**

**Authentication**
- WebAuthn for ADE on macOS 15 — `ASWebAuthenticationSession` supports passkeys and hardware security keys **[NEW]**

**Platform SSO**
- FileVault unlock via IdP authentication **[NEW]**
- HPKE security option **[NEW]**
- `AllowOfflineGracePeriod`, `AllowTouchIDOrWatchForUnlock` configuration keys **[NEW]**

**Education**
- Schoolwork 3.0 — assessment/scoring APIs for teachers **[NEW]**
- Assessment Mode Multi-App for iPad **[NEW]**
- Calculator for iPad MDM configuration (Scientific mode, Math Notes) **[NEW]**

## Code Highlights
No developer-facing code snippets in the session. The primary integration points are: MDM server responses using `RequireBetaProgram` JSON/XML during ADE, `ASWebAuthenticationSession` for WebAuthn-based enrollment on macOS, and Platform SSO configuration keys in the declarative payload. See the Device Management Client Schema on GitHub for complete payload definitions.

## Takeaways
- The new unified `SoftwareUpdate` declaration replaces all legacy software update commands and profiles — adopt it for any supervised device running iOS/iPadOS 18 or macOS 15.
- visionOS 2 MDM is now comprehensive: Automated Device Enrollment, supervision, payloads, commands, and restrictions match the iOS/iPadOS feature set.
- IT admins can now remotely enroll devices into AppleSeed for IT beta programs and implement a phased OS rollout starting from the first beta release.
- Safari extension management is new on iOS/iPadOS/macOS — IT can now fully control which extensions run and what sites they can access.

---
_Source: WWDC24 Session 10143 page (abstract, chapter summaries, and resource links)._
