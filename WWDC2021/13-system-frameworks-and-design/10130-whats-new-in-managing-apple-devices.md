# What's New in Managing Apple Devices
**WWDC21 · Session 10130** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10130/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session is a high-level overview of all major device management improvements across Apple platforms in 2021, with pointers to deeper-dive sessions for each topic. The iOS/iPadOS section covers a redesigned VPN & Device Management settings pane, a new Required App mechanism for unsupervised devices, managed pasteboard restrictions, and Shared iPad enhancements. The macOS section covers System Extension management improvements, kernel extension changes, iOS app management on Apple silicon Macs, and two major new features: remote device lock with PIN on Apple silicon and Erase All Content and Settings via MDM.

The overarching theme is balancing privacy, user agency, and administrative control across supervised and unsupervised enrollment types. Each new MDM command or payload key is framed in terms of which device types and supervision states support it. All new capabilities are documented at developer.apple.com/documentation/DeviceManagement.

## Key Topics

### iOS and iPadOS 15

**VPN & Device Management Settings**
- Settings app now combines VPN and Device Management into a single "VPN & Device Management" pane **[NEW]**
- Provides a comprehensive, unified view of device management state for end users

**Required App (Unsupervised Devices)**
- New Required App mechanism **[NEW]**: one app can be designated for install on unsupervised devices without additional user approval at install time
- User consent is obtained during MDM enrollment; only the designated app benefits from this streamlined install
- Implementation: add iTunes Store ID to MDM profile, send `InstallApplication` command with a managed app attribute to prevent user removal

**Managed Pasteboard**
- New `requireManagedPasteboard` restriction **[NEW]**: controls whether copy/paste is subject to managed open-in rules
- System apps honoring the restriction: Calendar, Notes, Mail, Files; third-party apps require no changes
- When paste is blocked by the restriction, a "Paste Not Allowed" notification is shown; organization name in notification configurable via `OrganizationInfo` Settings command

**Shared iPad Enhancements**
- Three new keys in `SharedDeviceConfiguration` Settings command (added in iOS 14.5, discussed here):
  - `TemporarySessionOnly` **[NEW]**: restricts device to Temporary Sessions only (no Managed Apple ID login)
  - `TemporarySessionTimeout` **[NEW]**: auto-logout after inactivity during a Temporary Session
  - `UserSessionTimeout` **[NEW]**: auto-logout after inactivity during a Managed Apple ID session

**Apple TV Changes**
- tvOS 15: Apple TV no longer broadcasts MAC addresses over Bonjour (security enhancement)
- New `TVDeviceName` key added to TV Remote payload **[NEW]**: use alongside `TVDeviceID` to filter Apple TV devices by name in the Remote widget, preventing unwanted pairing

**Protocol Changes**
- All payload types within a single profile now require unique payload identifiers
- For unsupervised devices, the Take Management prompt for apps can be declined up to 3 times before a 24-hour cooldown
- Various payload keys updated to use more inclusive language

### macOS Monterey

**System Extensions**
- macOS 11.3+: Installing the System Extension payload activates a pending extension; removing the payload deactivates it
- New `RemovableSystemExtension` feature **[NEW]**: allows an app to deactivate its own system extension (e.g., at uninstall time) without requiring an admin password — useful in no-admin deployments

**Kernel Extensions**
- New `RestartDevice` command option: `KextPaths` key **[NEW]** specifies kernel extension paths not yet discovered by the OS, enabling MDM to install and prepare extensions before the user launches the app
- New `NotifyUser` option **[NEW]** in `RestartDevice`: displays a reboot notification to the user for graceful restart
- `AllowNonAdminUserApprovals` key in System Policy Kernel Extensions payload: allows standard users to complete kernel extension installation

**iOS App Management on Apple Silicon**
- `DeviceInformation` query key reporting iOS app install support returns `true` on Apple silicon Macs running macOS 11.3+
- `InstallApplication` command gains new flag to indicate an iPhone or iPad app **[NEW]**
- In-house enterprise apps: manifest URL must point to a `.ipa` file (not `.pkg`)
- iOS-style provisioning profiles can now be independently managed via MDM **[NEW]**

**Remote Lock with PIN (Apple Silicon)**
- `DeviceLock` command enhanced for Apple silicon **[NEW]**: supports six-digit PIN, message, and phone number
- Mac reboots and presents the PIN screen; device is locked until PIN is entered, then reboots normally with data intact

**Recovery Lock (Apple Silicon)**
- New `SetRecoveryLock` MDM command **[NEW]**: sets a password required before booting into macOS Recovery
- New `VerifyRecoveryLock` MDM command **[NEW]**: verifies current recovery password from MDM server
- Recovery password can only be set/removed by MDM; removed when device is erased; MDM must know existing password to set a new one; use in conjunction with Activation Lock

**Erase All Content and Settings (macOS)**
- `EraseDevice` command now supported on Mac **[NEW]**: erases all user data and reboots to Setup Assistant
- Supported on: Mac computers with Apple silicon and Mac computers with Apple T2 security chip
- If multiple partitions exist, only the current system volume boots to Setup Assistant; other volumes are erased
- On Apple silicon, also resets security settings modified in recovery
- New `allowEraseContentAndSettings` restriction for Mac **[NEW]**: prevents users from performing this action themselves

## APIs & Frameworks

- Device Management protocol (MDM framework — server-side protocol, no app-level APIs)
- `InstallApplication` MDM command (enhanced with iPhone/iPad app flag **[NEW]**)
- `EraseDevice` MDM command (now supports macOS **[NEW]**)
- `DeviceLock` MDM command (PIN support on Apple silicon **[NEW]**)
- `SetRecoveryLock` MDM command **[NEW]**
- `VerifyRecoveryLock` MDM command **[NEW]**
- `RestartDevice` MDM command (new `KextPaths` and `NotifyUser` keys **[NEW]**)
- `SharedDeviceConfiguration` Settings command (new `TemporarySessionOnly`, `TemporarySessionTimeout`, `UserSessionTimeout` keys **[NEW]**)
- `OrganizationInfo` Settings command (organization name in system notifications)
- `DeviceInformation` query (iOS app install support key)
- `SecurityInfo` query (bootstrap token requirement)
- `requireManagedPasteboard` restriction key **[NEW]**
- `allowEraseContentAndSettings` restriction key (Mac) **[NEW]**
- `RemovableSystemExtension` (System Extension payload feature) **[NEW]**
- `AllowNonAdminUserApprovals` key (System Policy Kernel Extensions payload)
- `TVDeviceName` key (TV Remote payload) **[NEW]**
- `TVDeviceID` key (TV Remote payload, existing)

## Code Highlights

No app-level code samples; all APIs are MDM protocol payload keys and commands configured server-side. See developer.apple.com/documentation/DeviceManagement for full payload schemas.

## Takeaways

- The new Required App mechanism gives MDM vendors a privacy-preserving path to ensure a single critical agent app installs on unsupervised devices without repeated user prompts.
- `requireManagedPasteboard` closes a data-leakage gap between managed and unmanaged apps without requiring any changes to third-party apps.
- Erase All Content and Settings via `EraseDevice` on Mac (T2 and Apple silicon) finally brings return-to-service workflows to Mac fleets that previously required Erase Assistant or manual steps.
- The new `SetRecoveryLock` and enhanced `DeviceLock` commands for Apple silicon Macs bring feature parity with iOS device lock and provide defense-in-depth — use both in conjunction with Activation Lock.

---
_Source: WWDC21 Session 10130 page (abstract, chapter summaries, code samples, and resource links)._
