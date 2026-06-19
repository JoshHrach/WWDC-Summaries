# What's New in Managing Apple Devices
**WWDC23 · Session 10040** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10040/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17

## Overview
This session surveys enterprise and education device management improvements across iOS 17, iPadOS 17, and macOS Sonoma 14. It covers six major areas: macOS Automated Device Enrollment hardening (FileVault enforcement, OS version requirements, mandatory enrollment after network connection), Platform Single Sign-On expansion (on-demand local account creation, SmartCard login, identity-provider group-based authorization), new macOS restrictions and password policy enhancements, Managed Device Attestation landing on macOS with new attestation properties, iOS/iPadOS Return to Service (touchless device recycling), educational Shared iPad improvements, private cellular network management (private 5G/LTE eSIM, geofenced SIM activation, network slicing), and Apple Configurator automation via Shortcuts.

## Key Topics

### macOS — Automated Device Enrollment Hardening
- **FileVault enforcement during Setup Assistant** **[NEW]** — MDM can require FileVault enablement before enrollment completes; choice to show or escrow the recovery key.
- **Minimum OS version requirement** **[NEW]** — MDM returns HTTP 403 with a JSON body when the device requests the enrollment profile on an OS below the required version; the device guides the user through an update and restarts automatically before resuming Setup Assistant.
- **Mandatory enrollment safeguard** **[NEW]** — if the Mac is offline during initial setup, enrollment is no longer silently skipped. A full-screen Setup Assistant launches when a network is later connected, with an 8-hour grace period ("Not Now") before enrollment becomes required.

### Platform Single Sign-On (macOS 14)
- **On-demand local account creation** **[NEW]** — users can log into a shared Mac with their IdP (Identity Provider) credentials at the login window to create a new local account on the fly. Requires: IdP connectivity, FileVault unlocked at login window, MDM Bootstrap Token support, and a new shared device key.
- **SmartCard authentication at login window and screen saver** **[NEW]** — Platform SSO now supports SmartCard credentials, not just username/password.
- **Group-based permissions** **[NEW]** — IdP group membership controls local administrator privileges, authorization group access, and network (non-local) account authorization prompts. Configured via the `PlatformSSO` dictionary in the Extensible SSO profile:
  - `UserAuthorization: Groups` — apply group membership to local users
  - `EnableAuthorization: true` — extend group authorization to network accounts
  - `AdministratorGroups` — groups whose members can respond to admin authorization prompts
  - `AuthorizationGroups` — groups granted access to specific rights (e.g., printer configuration) without full admin privileges
  - `AdditionalGroups` — creates local groups (e.g., for `sudo` configuration)
- **System Settings status UI** **[NEW]** — users can view Platform SSO registration status and reauthenticate/repair from System Settings.

### Password Policy and Restrictions (macOS 14)
- **Regular expression password policies** **[NEW]** — password policies can now use regex patterns for complex requirements not expressible with existing options.
- **Password compliance notifications** **[NEW]** — when a stricter policy is installed, users are notified that their password may not comply. Compliance is verified at next login; non-compliant users are prompted to change now or later (reminder shown on every subsequent login until resolved).
- **New restrictions** **[NEW]** — granular per-setting management (rather than hiding entire System Preference panes): restrict Apple ID logins, Internet Accounts modifications, adding local user accounts, Time Machine backups, and more.

### Managed Device Attestation — Now on macOS 14
- **Managed Device Attestation on macOS** **[NEW]** — macOS 14 gains full Managed Device Attestation support (previously iOS/iPadOS only). ACME attestation provisions hardware-bound keys stored in the data protection keychain; keys usable with VPN, 802.1x, Kerberos, Exchange, and MDM.
- **New attestation properties (all platforms)** **[NEW]**: low-level boot loader version, operating system version, Software Update Device ID, and **Secure Enclave Enrollment ID** (allows two different servers to verify they are communicating with the same device by comparing IDs; changes on unenroll/reenroll; non-device-identifying).
- **New macOS-specific attestation properties** **[NEW]**: SIP status, secure boot status, whether third-party kernel extensions are allowed.

### iOS/iPadOS — Return to Service
- **Return to Service** **[NEW]** — MDM `EraseDevice` command gains an optional dictionary enabling fully touchless device recycling: erase, Wi-Fi reconnection, MDM re-enrollment, and Home Screen landing — all without physical interaction.
- `EraseDevice` dictionary fields: Wi-Fi profile (required), enrollment profile (optional if device is in ASM/ABM — device self-discovers enrollment during activation), previously selected language/region are applied automatically.

### iOS/iPadOS — Education (Shared iPad)
- **Easy student sign-in (proximity-based)** **[NEW]** — teacher iPad can sign in a nearby student iPad using proximity + particle cloud scan flow. Requirements: same Apple School Manager location, physical proximity, student authorization for personal devices.
- **`AwaitUserConfiguration`** **[NEW]** — MDM can hold the Shared iPad at the login window after user authentication until all configurations are applied, then release via an MDM command.
- **`SkipLanguageAndLocaleSetupForNewUsers`** (MDM key) **[NEW]** — skips language/locale screens on first login for Shared iPad users, applying system defaults instead.
- **Temporary user quota** **[NEW]** — `QuotaSize` is now honored for temporary Shared iPad users, allowing IT to reserve storage for apps/content.

### Cellular Network Management
- **Private 5G/LTE eSIM on iPhone** **[NEW]** — iPhone 5G models gain the private LTE/non-standalone 5G eSIM MDM management that iPads received in 2022.
- **Standalone 5G private networks** **[NEW]** — both iPhone and iPad now support private standalone 5G networks.
- **Geofenced SIM activation** **[NEW]** — MDM can define geofence regions; the private network eSIM activates only when the device enters a defined geolocation, saving power.
- **Intelligent SIM selection** — devices select the best cellular SIM based on network quality; option to prefer the private network over available Wi-Fi.
- **5G network slicing** **[NEW]** — assign individual managed apps to a specific 5G network slice (with particular QoS/characteristics) via the `CellularSliceUUID` app attribute in the MDM install command or declarative app configuration. Not used when a VPN is active for the app or device.

### Network Relays
- **MDM-configured relays** **[NEW]** — a new MDM profile payload type configures network relays (secure proxies) for managed apps, specific domains, or the entire device — no app install required. Also configurable via `NERelayManager` API in the NetworkExtension framework. Can be combined with iCloud Private Relay.

### Apple Configurator
- **Apple Configurator for iPhone — MDM server assignment** **[NEW]** — when adding devices to ASM/ABM, IT admins can now directly assign each device to an MDM server (default for type, specific server, or no assignment) within Apple Configurator for iPhone, eliminating the separate web portal step.
- **Apple Configurator for Mac — Shortcuts actions** **[NEW]** — Shortcuts integration adds actions for update, restore, erase, and prepare for iPhone and iPad. Shortcuts can be triggered automatically on device attach/detach. MDM developers can expose MDM commands as Shortcut actions.

### Deprecations (Future Release)
- **APN payload** — will be removed; use `com.apple.cellular` payload.
- **Top-level cellular keys in DeviceInformation query** — will be removed; use `ServiceSubscriptions` response.
- Several restrictions will require supervision or will only apply to a personal Apple ID.

## APIs & Frameworks

- `Device Management` framework / MDM protocol
- `EraseDevice` MDM command — new optional `ReturnToService` dictionary **[NEW]**
- `AwaitUserConfiguration` MDM key for Shared iPad **[NEW]**
- `SkipLanguageAndLocaleSetupForNewUsers` MDM key **[NEW]**
- `CellularSliceUUID` app attribute (MDM install command / declarative config) **[NEW]**
- Extensible SSO profile — `PlatformSSO` dictionary with `UserAuthorization`, `EnableAuthorization`, `AdministratorGroups`, `AuthorizationGroups`, `AdditionalGroups` **[NEW]**
- ACME server protocol — hardware-bound key provisioning via Managed Device Attestation
- `DeviceInformation` MDM query — new `SecureEnclaveEnrollmentID`, boot loader version, OS version, Software Update Device ID attestation properties **[NEW]**
- `NERelayManager` API (NetworkExtension framework) — programmatic relay configuration **[NEW]**
- MDM relay profile payload type **[NEW]** — declarative relay configuration
- `InstallApplication` MDM command — now supports packages installing multiple apps into /Applications **[NEW]**
- Apple Configurator for iPhone — MDM server assignment during device registration **[NEW]**
- Apple Configurator for Mac — Shortcuts actions (update, restore, erase, prepare) **[NEW]**
- `NSMotionUsageDescription` — not applicable (MDM session, no Swift APIs)
- `Device Management Client Schema` — GitHub: `apple/device-management`
- `com.apple.cellular` payload — preferred replacement for deprecated APN payload

## Code Highlights
No Swift/Objective-C code samples in this session. Key MDM command structure:

```json
// EraseDevice command with Return to Service
{
  "Command": {
    "RequestType": "EraseDevice",
    "ReturnToService": {
      "Enabled": true,
      "WifiProfileData": "<base64-encoded-Wi-Fi-mobileconfig>",
      "ProfileData": "<base64-encoded-enrollment-mobileconfig>"
    }
  }
}

// PlatformSSO group configuration (Extensible SSO profile excerpt)
{
  "PayloadType": "com.apple.extensiblesso",
  "ExtensionData": {
    "PlatformSSO": {
      "UserAuthorization": "Groups",
      "EnableAuthorization": true,
      "AdministratorGroups": ["corp-admins"],
      "AuthorizationGroups": {
        "system.preferences.printing": ["helpdesk-staff"]
      },
      "AdditionalGroups": ["sudo-users"]
    }
  }
}
```

## Takeaways
- Return to Service eliminates physical touch for device recycling — erasing and re-enrolling a fleet of iOS/iPadOS devices can now be fully automated via MDM, a major operational win for high-turnover deployments.
- Platform SSO on macOS 14 can now create local accounts on demand from IdP credentials at the login window, enabling true identity-provider-managed shared Macs without pre-provisioning accounts.
- Managed Device Attestation on macOS brings the same hardware-bound key and device-property attestation story from iOS/iPadOS to Mac, with new properties (SIP, secure boot, kernel extensions, Secure Enclave Enrollment ID) for richer trust evaluation.
- 5G network slicing and geofenced SIM activation give enterprises fine-grained control over private network connectivity without compromising battery life or forcing per-app VPN configurations.

---
_Source: WWDC23 Session 10040 page (abstract, transcript, and resource links)._
