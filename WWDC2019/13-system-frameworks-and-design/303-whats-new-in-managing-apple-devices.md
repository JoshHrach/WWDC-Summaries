# What's New in Managing Apple Devices
**WWDC19 · Session 303** · [Watch](https://developer.apple.com/videos/play/wwdc2019/303/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, tvOS 13

## Overview
This session covers the major advances in Apple's enterprise device management platform for 2019. The two headline features are User Enrollment—a new MDM enrollment mode designed specifically for BYOD (personally owned) devices that balances employee privacy with corporate data security—and Extensible Single Sign-On, a new extension point that lets third-party identity providers integrate natively with the OS for authentication in apps and Safari on both iOS 13 and macOS Catalina.

Additional topics include updates to Apple Business Manager (ABM) and Apple School Manager (ASM), Classroom app updates, tvOS management additions, macOS Catalina-specific management changes (Activation Lock for T2 Macs, Bootstrap Token for FileVault, FileVault MDM enrollment requirements), deprecations, and a completely overhauled device management documentation system on developer.apple.com.

## Key Topics

### Apple Business Manager and Apple School Manager
- Custom Apps (formerly B2B apps) expanded: organizations can now distribute custom apps to their own employees and to other companies. Coming to ASM as well.
- Federated authentication with Microsoft Azure Active Directory coming to ABM (already in ASM).
- ABM and ASM now fully supported on iPad (shipping iOS).
- Apple Deployment Programs (predecessor to ABM/ASM) being retired at end of 2019.
- Managed Apple IDs now grant access to AppleSeed for IT (beta software, documentation, test plans) and Feedback Assistant.

### Classroom App
- Classroom for Mac updated to let teachers manage student Macs with the same feature set as student iPads.
- Student Macs show a Classroom pane in System Preferences.
- Screen viewing, Screen Sharing, Screenshot, and Apple Notes restrictions now respect Classroom restrictions on macOS.
- New "Hide Apps" feature (Fall 2019): teacher taps Hide to return students to the Home screen without locking.
- Fall 2019 Classroom supports Dark Mode on iOS 13.

### iPad Desktop-Class Browsing Impact
- iPad in iOS 13 identifies itself as a Mac in the User-Agent string; MDM servers and enrollment flows that detect device type via User-Agent must stop doing so.

### tvOS Management Additions
- Organizations can now control when software update prompts appear in tvOS UI.
- Auto-configure date and time via MDM.
- Content caching now includes tvOS screensavers.

### User Enrollment (BYOD) **[NEW]**
- New MDM enrollment mode for personally owned devices; requires a Managed Apple ID.
- Three pillars: Managed Apple ID as work identity anchor; cryptographic APFS volume separation; limited set of management capabilities.
- iOS creates a separate managed APFS volume with distinct cryptographic keys; volume and keys destroyed on unenrollment.
- No UDID or persistent hardware identifier exposed to MDM server; instead an Enrollment ID (destroyed at unenrollment) is used.
- MDM capabilities deliberately restricted to protect personal data: no Erase Device, no unlock token (can't clear passcode), no personal app visibility, no supervised-only restrictions.
- Managed Open In and lock-screen data restrictions supported.
- Per-app VPN supported; VPN server domain must match the second-level domain of configured services.
- Passcode policy limited to six-digit non-simple passcode.
- Wi-Fi payload: no proxy key support; use WPAD on access points.
- Apps installed via user enrollment always removed when enrollment ends; VPP uses user-based PurchaseMethod 1 (charges Managed Apple ID, not personal).
- Supported on both iOS/iPadOS 13 and macOS Catalina.
- Enrollment profile uses `ManagedAppleID` key (no AccessRights key needed).
- Account-only Remote Wipe available for Exchange accounts in user enrollment mode.

### Extensible Single Sign-On (SSO) **[NEW]**
- New extension point for third-party identity providers (Microsoft, Okta, Ping, etc.); separate from consumer "Sign in with Apple."
- Configured via the new `com.apple.extensiblesso` MDM payload on iOS 13 and macOS Catalina.
- Two extension types:
  - **Redirect extensions**: for modern protocols (OpenID Connect, OAuth 2, SAML). Extension receives request URL/headers/body, completes auth with IdP, returns URL response or tokens. Works in Safari and native apps. Native apps can send explicit operations (login, refresh, logout) to the extension.
  - **Credential extensions**: for challenge-response protocols (Kerberos, custom). Extension receives HTTP challenge from OS, completes auth, returns credentials/headers. No associated domains requirement.
- Both types support presenting native UI, loading webpages, appending device trust signals (Secure Enclave keys, OS version, etc.).
- Requires Associated Domains (for redirect extensions); new Managed Associated Domains MDM profile (macOS) and app attribute (iOS) allow enterprise-managed domains without compile-time entitlement changes.
- Built-in Kerberos/Active Directory extension **[NEW]** ships with iOS 13 / macOS Catalina (based on Enterprise Connect); supports username/password, certificate-based identity, and smart cards on macOS.

### macOS Catalina Management Changes
- **Remote Desktop via MDM**: two new MDM commands to enable Remote Desktop and configure common options.
- **Bootstrap Token for FileVault**: MDM server requests and escrows a bootstrap token; used to generate SecureToken for new users signing in on FileVault Macs (helps with password sync).
- **Privacy Policy Payload additions**: new entries for keyloggers/screen recording, and whitelisting internal apps to bypass notarization requirement.
- **FileVault via MDM** now requires user-approved MDM enrollment; `fdesetup` command-line credential passing no longer works.
- **Activation Lock for T2 Macs**: same MDM endpoint and API as iOS; APIs to be available late summer 2019.
- **Certificate Transparency**: new payload to opt out specific sensitive certificates/domains from public CT logs.
- **WPA3** added to Wi-Fi payload (personal and enterprise).
- **Token-based APNS authentication** for MDM servers now enforced for supervised device enrollments.
- Deprecations in Catalina: silent profile installation from command line; path-based app blacklists/whitelists (parental controls payload); user-channel-only enrollments.

### iOS Restriction Changes
- Supervised-only restrictions sent to unsupervised devices will no longer be honored in iOS 13 for new enrollments; grandfathered for existing unsupervised devices upgrading from iOS 12 (until they restore).
- Unlock token now only available in the first successful TokenUpdate after enrollment.
- New `managed-only` key filter for `InstalledApplicationList` MDM query.

### Custom Setup Assistant Enrollment UI **[NEW]**
- Organizations can now inject a custom web UI during Setup Assistant enrollment to present branding, authentication, privacy policy, acceptable use policy, etc.

### Updated Device Management Documentation
- Device management documentation moved to developer.apple.com alongside all other Apple developer documentation.
- "Show API Changes" toggle highlights modified and new keys between current OS and upcoming beta.
- Availability tables show which platforms and OS versions each payload key supports, including user enrollment compatibility.
- Keys auto-imported from Apple engineering code for accuracy and timeliness.

## APIs & Frameworks

### MDM Protocol
- User Enrollment profile: `ManagedAppleID` key **[NEW]** — triggers user enrollment mode
- `PayloadOrganization` key — identifies managed accounts in UI
- Enrollment ID (non-persistent, replaces UDID in user enrollment) **[NEW]**
- EAS device identifier for Exchange user enrollment mode **[NEW]**
- `ManagedOnly` key for `InstalledApplicationList` query **[NEW]**
- Bootstrap Token escrow commands **[NEW]** (`SetBootstrapToken`, `GetBootstrapToken`)
- Remote Desktop enable/configure MDM commands **[NEW]**
- Activation Lock MDM commands for T2 Macs **[NEW]** (same API as iOS)

### Configuration Profile Payloads
- `com.apple.extensiblesso` — Extensible SSO payload **[NEW]**
  - Keys: `ExtensionIdentifier`, `TeamIdentifier`, `Type` (`redirect` or `credential`), `URLs`, `ExtensionData`
- Managed Associated Domains (macOS) **[NEW]**
- Associated Domains app attribute (iOS) **[NEW]**
- Certificate Transparency exemption payload **[NEW]**
- WPA3 in Wi-Fi payload **[NEW]**
- Privacy Policy payload additions for macOS Catalina **[UPDATED]**
- Per-app VPN with domain guidance keys **[NEW]** (also for user enrollment with domain restrictions)

### Device Management Documentation
- `https://developer.apple.com/documentation/DeviceManagement` — new home for all documentation

## Code Highlights

Example user enrollment profile snippet (MDM payload):
```xml
<key>PayloadType</key>
<string>com.apple.mdm</string>
<key>ManagedAppleID</key>
<string>user@org.com</string>
<key>PayloadOrganization</key>
<string>Acme Inc.</string>
<!-- No AccessRights key for user enrollment -->
```

Example Extensible SSO payload (redirect type):
```xml
<key>PayloadType</key>
<string>com.apple.extensiblesso</string>
<key>ExtensionIdentifier</key>
<string>com.example.ssoextension</string>
<key>TeamIdentifier</key>
<string>TEAMID1234</string>
<key>Type</key>
<string>redirect</string>
<key>URLs</key>
<array>
    <string>https://login.example.com</string>
</array>
<key>ExtensionData</key>
<dict>
    <key>UserName</key>
    <string>user@example.com</string>
</dict>
```

## Takeaways
- User Enrollment provides a privacy-first BYOD MDM mode using cryptographic APFS volume separation and Managed Apple IDs; admins should evaluate it for all BYOD programs.
- Extensible SSO brings native, extensible enterprise authentication to all iOS 13 and macOS Catalina apps including a built-in Kerberos/Active Directory extension.
- Several macOS Catalina management behaviors have changed (FileVault, Activation Lock, Bootstrap Token); admins should audit scripts and MDM agents before upgrading.
- The completely rebuilt device management documentation on developer.apple.com with API change tracking is the definitive reference going forward.

---
_Source: WWDC19 Session 303 page (abstract, chapter summaries, code samples, and resource links)._
