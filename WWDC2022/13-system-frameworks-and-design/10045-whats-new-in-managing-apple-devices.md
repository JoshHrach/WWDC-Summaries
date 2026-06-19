# What's New in Managing Apple Devices
**WWDC22 · Session 10045** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10045/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
This session covers the full breadth of device management enhancements across Apple platforms in 2022. Key areas include Apple Configurator for iPhone gaining the ability to add iPhone and iPad devices (not just Macs) to organizations; major identity improvements including Google Workspace federation, Enrollment SSO, OAuth 2 support, and Platform SSO for macOS; macOS Ventura software update enhancements; new iOS/iPadOS MDM capabilities including per-app DNS Proxy and Web Content Filter; improved eSIM management; Shared iPad enhancements; and a landmark change making MDM protocol documentation source available as open source on GitHub.

Platform SSO for macOS Ventura is particularly notable: it allows users to sign in once at the login window and have that identity flow through to apps, websites, SSO extensions, and Kerberos — a modern replacement for Active Directory binding.

## Key Topics

### Apple Configurator for iPhone
Previously limited to adding Macs to Apple School Manager / Apple Business Manager via Setup Assistant. Now also supports adding iPhone and iPad devices. Works by scanning the Wi-Fi pane in Setup Assistant (vs. the country/region pane for Macs).

### Identity Management
- **Google Workspace as Identity Provider**: Apple Business Manager integrates with Google Workspace for Managed Apple ID federation
- **Sign in with Apple at Work & School**: Works with Managed Apple IDs in apps supporting Sign in with Apple; configurable allow list in ASM/ABM
- **OAuth 2 Support** (iOS/iPadOS 16): Added as an authorization mechanism for MDM, enabling short-lived tokens with silent refresh
- **Enrollment SSO** (iOS/iPadOS 16): New enrollment flow combining extensible SSO + Account-Driven User Enrollment; user authenticates once via a native app, which then handles the MDM enrollment silently
- **Platform SSO** (macOS Ventura): Sign in once at login window; SSO tokens flow to all apps/websites/SSO extensions and Kerberos TGTs; supports password or Secure Enclave-backed key auth; built on OAuth/OpenID; replacement for AD binding

### macOS Ventura MDM Enhancements
- OS Update commands now respond when device is in Power Nap/sleep mode
- New `priority` key in `ScheduleOSUpdate` (High/Low) — High mimics user-initiated update
- Enhanced `OSUpdateStatus` response: `DeferralsRemaining`, `MaxDeferrals`, `NextScheduledInstall`, `PastNotifications`
- **Rapid Security Response** support: Two new restrictions (`allowRapidSecurityResponseInstallation`, `allowRapidSecurityResponseRemoval`)
- Automated Device Enrollment: Network required after erasing/restoring for ADE-registered Macs
- `profiles` CLI tool rate-limited to 10 requests/day per command; new `--cached` flag for non-consuming reads
- TLS trust policy change: Manually installed certificate payloads will require explicit user trust for TLS in a future macOS release
- `allowUSBRestrictedMode` restriction applies to new "Allow accessories to connect" security feature on Apple silicon portables
- New `allowUniversalControl` and `UIConfigurationProfileInstallation` restrictions

### iOS/iPadOS MDM Enhancements
- **Per-App DNS Proxy** (iOS/iPadOS 16): New `DNSProxyUUID` key in DNS Proxy payload
- **Per-App Web Content Filter** (iOS/iPadOS 16): New `ContentFilterUUID` key in Web Content Filter payload
- Both per-app features configured via payload UUID + matching `InstallApplication`/`Settings` app attributes; up to 7 per-app web filters + 1 system-wide
- `ServiceSubscriptions` query enhancements; some duplicate keys deprecated in iOS/iPadOS 16
- eSIM management: `RefreshCellularPlans` command for silent eSIM installation; `allowESIMModification` restriction; `PreserveDataPlan` key in `EraseDevice`
- **Shared iPad**: `ManagedAppleIDDefaultDomains` in `SharedDeviceConfiguration` for QuickType suggestions; `OnlineAuthenticationGracePeriod` key for controlling remote auth frequency
- MDM can manage accessibility settings (Text Size, VoiceOver, Zoom, Touch Accommodations, Bold Text, Reduce Motion, Increase Contrast, Reduce Transparency)
- Apps can now be installed during `AwaitDeviceConfigured` state in Automated Device Enrollment Setup Assistant
- `CertificateList` query returns `NotNow` before first unlock
- tvOS 16: Remote stays paired after Apple TV erase

### Open Source MDM Documentation
MDM protocol documentation source code published on Apple's GitHub under MIT license. Includes YAML files for all commands, profiles, and declarative management schemas.

## APIs & Frameworks

**Device Management / MDM Protocol**
- `ScheduleOSUpdate` command — new `priority` key (High/Low) **[NEW]**
- `OSUpdateStatus` command — new keys: `DeferralsRemaining`, `MaxDeferrals`, `NextScheduledInstall`, `PastNotifications` **[NEW]**
- `allowRapidSecurityResponseInstallation` restriction **[NEW]**
- `allowRapidSecurityResponseRemoval` restriction **[NEW]**
- `allowUniversalControl` restriction **[NEW]**
- `UIConfigurationProfileInstallation` restriction key **[NEW]**
- `com.apple.webcontent-filter` payload — new `ContentFilterUUID` key **[NEW]**
- `com.apple.dnsProxy.managed` payload — new `DNSProxyUUID` key **[NEW]**
- `InstallApplication` command — `ContentFilterUUID`/`DNSProxyUUID` in `Attributes` dict **[NEW]**
- `ServiceSubscriptions` DeviceInformation key — expanded eSIM data; deprecated duplicate keys in iOS 16 **[UPDATED]**
- `RefreshCellularPlans` MDM command — silent eSIM installation
- `allowESIMModification` restriction
- `EraseDevice` command — `PreserveDataPlan` key
- `SharedDeviceConfiguration` Settings command — new `ManagedAppleIDDefaultDomains` key **[NEW]**
- `SharedDeviceConfiguration` Settings command — new `OnlineAuthenticationGracePeriod` key **[NEW]**
- `Settings` command — new `AccessibilitySettings` item with accessibility keys **[NEW]**
- `CertificateList` query — returns `NotNow` before first unlock (behavior change in iOS 16)
- `allowUSBRestrictedMode` restriction — now also applies to Thunderbolt/USB accessory security on Apple silicon Macs

**Identity / SSO**
- Enrollment SSO extension entitlement **[NEW]** — requires Apple Developer Program application
- Extensible SSO (`com.apple.extensiblesso`) payload — new enrollment SSO keys
- Platform SSO (macOS Ventura) — new SSO protocol built on OAuth/OpenID **[NEW]**
- OAuth 2 authorization for MDM enrollment **[NEW in iOS/iPadOS 16]**

## Code Highlights

```json
// Enrollment SSO JSON document
{
  "iTunesStoreID": 12345678,
  "AssociatedDomains": ["betterbag.com"],
  "AssociatedDomainsEnableDirectDownloads": false,
  "ConfigurationProfile": "Base64EncodedProfile"
}
```

```xml
<!-- Per-App Web Content Filter payload -->
<key>ContentFilterUUID</key>
<string>063D927E-ACE2-445F-8024-B355A6921F50</string>

<!-- Per-App DNS Proxy payload -->
<key>DNSProxyUUID</key>
<string>063D927E-ACE2-445F-8024-B355A6921F50</string>

<!-- Manage Accessibility via MDM -->
<key>Item</key><string>AccessibilitySettings</string>
<key>VoiceOverEnabled</key><true/>
<key>BoldTextEnabled</key><true/>
```

## Takeaways

- Apple Configurator for iPhone now handles iPhone/iPad enrollment alongside Macs — eliminating the USB requirement for iOS/iPadOS device enrollment.
- Platform SSO on macOS Ventura is the modern AD-binding replacement: one login window authentication grants SSO tokens to all apps, websites, and Kerberos.
- Per-app DNS Proxy and Web Content Filter in iOS/iPadOS 16 finally give MDM control over BYOD (user-enrolled) app traffic security.
- MDM protocol documentation is now open source on GitHub under the MIT license — enabling new tooling and integrations.

---
_Source: WWDC22 Session 10045 page (abstract, chapter summaries, code samples, and resource links)._
