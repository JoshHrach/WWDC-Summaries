# Explore Advances in Declarative Device Management
**WWDC23 · Session 10041** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10041/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10

## Overview
This session details major new features added to Declarative Device Management (DDM) in the iOS 17 / macOS Sonoma release. Having established foundational elements in WWDC21 and WWDC22, the focus shifts to implementing core management features that are exclusively available through DDM and provide parity with and improvements over traditional MDM.

The five major additions are: enforced managed software updates with user-facing deadline countdowns, managed app installation with optional installs and a new public SwiftUI framework, managed system service configuration files (tamper-resistant), security certificate/identity assets, and seamless MDM-to-DDM profile ownership transfer. New status items for software update state, FileVault, background tasks, and certificate details are also introduced.

## Key Topics

- **Managed software update** — New `com.apple.configuration.softwareupdate.enforcement.specific` configuration with `TargetOSVersion`, `TargetBuildVersion`, and `TargetLocalDateTime` keys; asynchronous status items for install state, pending version, install reasons, and failure reasons; user-facing deadline notifications on increasing cadence; declarations take precedence over MDM commands.
- **Managed apps** — New managed app configuration (App Store or enterprise manifest); `InstallBehavior` of `Required` or `Optional`; new status item reporting installed app state asynchronously; new `ManagedAppDistribution` framework (entitlement required) with SwiftUI view extension for building management app UIs; new "Apps and Books for Organizations" server API replacing `contentMetadataLookup`.
- **System service configuration files** — New configuration referencing a data asset (ZIP archive) for tamper-resistant service config; new `ServiceManagement` library function to locate managed config files programmatically; built-in services adopting: `sshd`, `sudo`, PAM, CUPS, Apache httpd, bash/zsh.
- **Background task and FileVault status** — New status item enumerating installed background tasks (identifier, uid, state, type: daemon/agent/login item/app/user item, launchd label, program arguments, checksum); new FileVault enabled/disabled status item for macOS boot volume.
- **Certificate and identity assets** — New asset types: PEM/DER certificate, PKCS#12 identity, ACME-provisioned identity, SCEP-provisioned identity (with hardware-bound key support); new certificate and identity installation configurations; new enterprise passkey attestation configuration using an identity asset for WebAuthn attestation; S/MIME support added to Mail and Exchange configurations.
- **MDM-to-DDM profile ownership transfer** — DDM can take over management of an already-installed MDM profile without removing/reinstalling it; eliminates management gaps and disruption.

## APIs & Frameworks

**Declarative Device Management (protocol/schema) [NEW items]**
- `com.apple.configuration.softwareupdate.enforcement.specific` configuration **[NEW]**
- `softwareupdate.install-reasons` status item **[NEW]**
- `softwareupdate.pending-version` status item **[NEW]**
- `softwareupdate.install-state` status item **[NEW]**
- `softwareupdate.failure-reason` status item **[NEW]**
- `com.apple.configuration.app` managed app configuration **[NEW]**
- `com.apple.status.app.managed-list` status item **[NEW]**
- `com.apple.configuration.services.configuration-files` system service config configuration **[NEW]**
- `com.apple.asset.data` data asset type **[NEW]** — delivers arbitrary ZIP data for service configs
- `com.apple.status.systemconfiguration.background-task-list` status item **[NEW]**
- `com.apple.status.diskmanagement.filevault.enabled` status item **[NEW]**
- `com.apple.asset.credential.certificate` certificate asset **[NEW]** (PEM/DER)
- `com.apple.asset.credential.identity` PKCS#12 identity asset **[NEW]**
- `com.apple.asset.credential.acme` ACME identity asset **[NEW]** (hardware-bound key support)
- `com.apple.asset.credential.scep` SCEP identity asset **[NEW]**
- `com.apple.configuration.security.certificate` certificate install configuration **[NEW]**
- `com.apple.configuration.security.identity` identity install configuration **[NEW]**
- `com.apple.configuration.security.passkey.attestation` enterprise passkey attestation configuration **[NEW]**
- `com.apple.status.security.certificate-list` status item **[NEW]**
- Legacy profile configuration — now supports DDM taking over existing MDM-installed profiles **[NEW behavior]**

**ManagedAppDistribution (new public framework) [NEW]**
- SwiftUI view extension for rendering managed app views in custom management apps
- Requires entitlement obtained through App Store submission process
- Available on macOS, iOS, iPadOS

**Apps and Books for Organizations API [NEW]**
- Replaces `contentMetadataLookup` server API
- Provides versioning, customization, better performance/uptime

**ServiceManagement (library)**
- New function to programmatically locate managed service configuration file directory **[NEW]**

## Code Highlights

No Swift/Objective-C code samples in this session — content is protocol/schema-level for MDM server developers and administrators. Example configuration JSON structure:

```json
// Software update enforcement configuration (simplified)
{
  "Type": "com.apple.configuration.softwareupdate.enforcement.specific",
  "Payload": {
    "TargetOSVersion": "17.0",
    "TargetBuildVersion": "21A329",
    "TargetLocalDateTime": "2023-09-20T22:00:00"
  }
}
```

```json
// Managed app configuration (simplified)
{
  "Type": "com.apple.configuration.app",
  "Payload": {
    "AppStoreID": "361309726",
    "InstallBehavior": { "Install": "Required" }
  }
}
```

## Takeaways

- All new major management features in iOS 17 / macOS Sonoma are DDM-only — MDM support is no longer the priority for new capabilities.
- Software update enforcement via DDM gives IT admins deadline-based control with asynchronous status reporting; user experience includes escalating reminders and a "past due" recovery flow.
- The new `ManagedAppDistribution` framework lets management app developers build rich optional-install UIs without server round-trips — entitlement required.
- Enterprise passkey attestation via DDM allows organizations to restrict passkey provisioning to managed devices using WebAuthn attestation backed by a managed identity asset.

---
_Source: WWDC23 Session 10041 page (abstract, chapter summaries, code samples, and resource links)._
