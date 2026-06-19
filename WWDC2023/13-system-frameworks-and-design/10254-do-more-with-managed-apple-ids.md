# Do More with Managed Apple IDs
**WWDC23 · Session 10254** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10254/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14 (enterprise/MDM)

## Overview
Darsh from Apple's Enterprise Services team presents the iOS 17 / macOS Sonoma updates to Managed Apple IDs — the Apple ID type created and owned by organizations through Apple Business Manager (ABM) or Apple School Manager (ASM). The session covers three areas: new apps and services now available to Managed Apple IDs, new enrollment options (account-driven Device Enrollment), and new access management and identity provider integration capabilities for IT administrators and MDM developers.

This session is primarily targeted at MDM solution developers and enterprise IT administrators rather than general app developers, though the Sign in with Apple at Work and School improvements and the new MDM Check-in message type are actionable for MDM SDK developers.

## Key Topics

### New Apps and Services for Managed Apple IDs **[NEW]**
Managed Apple IDs in iOS 17 / macOS Sonoma now support:
- **iCloud Keychain** — password autofill and sync, including passkeys (Face ID / Touch ID). See "Deploy passkeys at work" for details.
- **Messages, Stocks, News, Siri** — iCloud sync for data in these apps across devices.
- **Wallet** — credit/debit cards, driver's license or state ID, transit cards, keys, badges.
- **Continuity features** — Continuity Camera, Handoff, Sidecar, Instant Hotspot, Universal Clipboard. Requires Managed Apple ID signed in to multiple devices.

### Account-Driven Device Enrollment **[NEW]**
Previously, Device Enrollment required users to manually download and install a management profile. iOS 17 / macOS Sonoma introduce **account-driven Device Enrollment (ADDE)** — enrollment triggered by signing in with a Managed Apple ID in Settings, identical UX to account-driven User Enrollment (BYOD).

Key differences from User Enrollment:
- Delivers most management capabilities of profile-based Device Enrollment (passcode policies, remote wipe/lock, device supervision on macOS).
- Device data is cryptographically separated (personal vs. work partitions), same as User Enrollment.
- Results in supervised device on macOS — highest management level.
- Available on iOS, iPadOS, and macOS.

**MDM developer changes**: The existing well-known endpoint now also receives the device model in the query parameters. To switch to ADDE, set the `Version` key in the well-known response to `"mdm-adde"` and set `EnrollmentMode` to `"ADDE"` in the enrollment profile (previously `"mdm-byod"` / `"BYOD"` for User Enrollment).

**Sign in with Apple at Work and School** — apps using Sign in with Apple can now accept a Managed Apple ID via web-view authentication or "Use a different Apple ID" in Safari, letting users sign in with their work account in managed apps and personal account in personal apps.

### Access Management Controls **[NEW]**
New organization-wide policies configured in ABM/ASM:
- **Device Sign-In Policy**: restrict Managed Apple ID sign-in to any device (default), managed devices only, or supervised devices only.
- **Messages and FaceTime controls**: limit to internal-only communication or disable entirely.
- **Developer access**: control access to Xcode and the Apple Developer site.
- **Per-app iCloud controls**: disable iCloud sync for any individual supported app/service; policy is reflected as a grayed-out toggle on device.

**MDM implementation for sign-in policy** — a new MDM Check-in request/response type:
- **GetToken request**: `MessageType: GetToken`, `TokenServiceType: com.apple.maid`
- **GetToken response**: a JSON Web Token (JWT) signed by the MDM server's private key, with claims: `iss` (MDM server UUID from ABM/ASM Get Account Detail endpoint), `iat` (timestamp), `jti` (random UUID, one-use), `service_type` (`com.apple.maid`).
- The device sends the GetToken message to its MDM server when a Managed Apple ID attempts sign-in; the JWT is verified against the organization's policy.
- If the device management state becomes non-compliant with the policy, the Managed Apple ID is signed out automatically.

### Identity Provider Integration **[NEW]**
ABM/ASM previously supported Microsoft Azure AD and Google Workspace for federated authentication and directory sync. iOS 17 / macOS Sonoma introduce support for **custom identity providers** using three open standards:
- **OpenID Connect** — federated authentication.
- **SCIM** (System for Cross-domain Identity Management) — directory sync (auto-create/update Managed Apple IDs from directory).
- **OpenID Shared Signals Framework** — account security events (password changes, etc.).

Okta is developing compatibility with this integration. Any identity provider supporting these three standards can now integrate with ABM/ASM.

## APIs & Frameworks

### MDM Protocol (New Messages / Fields)
- **GetToken check-in message** — new MDM Check-in request type for access management policy enforcement **[NEW]**
  - `MessageType: GetToken` **[NEW]**
  - `TokenServiceType: com.apple.maid` **[NEW]**
- **GetToken response** — JSON Web Token signed with MDM server private key **[NEW]**
  - Claims: `iss` (server UUID), `iat` (issued-at timestamp), `jti` (unique token ID), `service_type`
- **Well-known endpoint update** — now receives `device_model` query parameter alongside Managed Apple ID user identifier **[NEW]**
- `EnrollmentMode: ADDE` — enrollment profile value for account-driven Device Enrollment **[NEW]**
- `Version: mdm-adde` — well-known response value for account-driven Device Enrollment **[NEW]**
- **Get Account Detail endpoint** — returns MDM server UUID needed for JWT `iss` claim (used to configure GetToken response) **[NEW]**

### Apple Business Manager / Apple School Manager Configuration
- Device Sign-In Policy: Any Device / Managed Devices Only / Supervised Devices Only **[NEW]**
- Per-app iCloud disable controls (Messages, FaceTime, Reminders, etc.) **[NEW]**
- Messages/FaceTime scope restrictions (organization-only or disabled) **[NEW]**
- Custom Identity Provider integration (OpenID Connect + SCIM + Shared Signals) **[NEW]**

### Identity Provider Standards (Required for Custom IdP)
- OpenID Connect — federated authentication standard
- SCIM v2 — directory sync standard
- OpenID Shared Signals Framework — security event notification standard

### Sign in with Apple at Work and School (Updated)
- Now supports Managed Apple IDs in managed iOS, iPadOS, and macOS apps **[UPDATED]**
- Web view / "Use a different Apple ID" flows in Safari allow Managed Apple ID entry **[UPDATED]**

## Code Highlights
No code samples in the session — this is an enterprise configuration and MDM protocol session. The key MDM server implementation involves:

```json
// GetToken check-in request (MDM server receives this from device)
{
    "MessageType": "GetToken",
    "TokenServiceType": "com.apple.maid"
}

// GetToken response (JWT signed with MDM server's private key)
{
    "iss": "<MDM Server UUID from ABM/ASM>",
    "iat": <unix_timestamp>,
    "jti": "<random UUID string>",
    "service_type": "com.apple.maid"
}
```

```
# Well-known endpoint: account-driven Device Enrollment response
Version: mdm-adde

# Enrollment profile key
EnrollmentMode: ADDE
```

## Takeaways
- MDM developers supporting account-driven User Enrollment can enable account-driven Device Enrollment with two config changes: `Version: mdm-adde` in well-known response and `EnrollmentMode: ADDE` in the enrollment profile; use the new `device_model` query parameter to route users to the right enrollment type.
- To support the Managed Devices Only or Supervised Devices Only sign-in policies, implement the `GetToken` MDM Check-in message type — without it, those policies cannot be enforced on your managed devices.
- Custom identity providers can now integrate with ABM/ASM using OpenID Connect, SCIM, and OpenID Shared Signals — enabling federated authentication and directory sync for any organization regardless of IdP vendor.
- Wallet, iCloud Keychain (including passkeys), and Continuity features are now available to Managed Apple IDs — important for enterprise apps that need to support these features for work accounts.

---
_Source: WWDC23 Session 10254 page (abstract, chapter summaries, and resource links)._
