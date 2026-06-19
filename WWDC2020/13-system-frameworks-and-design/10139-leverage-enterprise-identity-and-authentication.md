# Leverage enterprise identity and authentication
**WWDC20 · Session 10139** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10139/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
This enterprise-focused session covers Apple's identity and authentication toolkit for organizational deployments. It examines three complementary technology areas: macOS account types and device authentication (local, network/mobile accounts and directory binding), Single Sign-On (SSO) extensions and the built-in Kerberos extension, and Managed Apple IDs with federated authentication and the newly announced SCIM integration with Azure Active Directory.

The key architectural recommendation is to use local macOS accounts for 1:1 deployments instead of binding Macs to Active Directory. Directory connectivity can be provided through the built-in Kerberos extension or third-party SSO extensions, avoiding the latency and complexity of mobile/network accounts. For organizational Apple services, Managed Apple IDs paired with Azure AD federation and SCIM provide seamless, automated identity lifecycle management.

## Key Topics

### macOS Account Types
- **Local accounts** (recommended for 1:1 deployments): user record stored locally; authentication is local; no network dependency; best login reliability and password management
- **Network accounts** (legacy, not recommended): source of truth is a directory server; home folder optionally on a network volume; significant modern-app compatibility issues
- **Mobile accounts** (supported but not recommended for 1:1): network account with local credential cache and local home folder; can suffer login delays from DNS/network issues; appropriate for shared lab environments (can be configured to expire)
- Avoid binding Macs to Active Directory for 1:1 deployments — use the built-in Kerberos extension or SSO extensions instead
- Binding is still required for Active Directory DFS traversal or AD Certificate payload delivery via MDM

### Single Sign-On (SSO) Extensions
- Introduced in iOS/iPadOS 13 and macOS Catalina; enables native apps and WebKit to authenticate through an organization's identity provider
- Configured via MDM; supports native UI, web UI, or transparent (headless) flows
- **New in iOS 14 / macOS Big Sur**:
  - SSO extensions can now be configured on the **user channel** (not just system channel) on macOS and iPadOS with Shared iPad; user-channel settings take priority
  - **Associated domains now work with per-app VPN**: SSO redirect extensions can target on-premise identity providers
  - New **per-app VPN Excluded Domains** list: cloud IdP traffic can bypass VPN and go direct, avoiding unnecessary on-premise routing
  - Per-app VPN now triggered by **auth-only requests** (login/password change), no longer requiring a separate HTTP request
  - `ASAuthorizationProviderAuthorizationOperation.configurationRemoved` **[NEW]**: called when the MDM profile configuring the SSO extension is removed; gives extensions time to log out, revoke tokens, and clean up Keychain entries
  - **Caller information** now provided to extensions: `localizedCallerDisplayName`, `callerTeamIdentifier`, `isCallerManaged` — enables better UX and security decisions
  - **Embedded wildcard URLs** now supported in matching URL lists (e.g., `auth.pretendco.com/*/login`) for large multi-tenant IdPs
  - `com.apple.developer.associated-domains.mdm-managed` entitlement is now public — no longer requires DTS approval **[NEW]**
  - Apple App Site Association files now fetched via Apple-operated CDN (not directly by device); use `managed` parameter for intranet domains

### Built-in Kerberos Extension Improvements
- **macOS menu extra** now reflects accurate extension state (credential, network, DNS, TGT all checked)
- **Customizable identity label and help text**: organizations can display their own ID name (e.g., "Associate ID") and support URL in the sign-in window
- Now triggers per-app VPN for auth-only requests on iOS, iPadOS, and macOS
- Full support for app-to-per-app VPN on macOS (add App-SSO-Agent and Kerberos-Menu-Extra to payload)
- User-channel profile delivery supported (simplifies bundling user-level certificate identities with Kerberos config)
- New option to delay the first login prompt until an admin command or an authentication challenge is received
- Optional Managed App Access Control on iOS: blocks unmanaged apps from accessing credentials

### Managed Apple IDs
- Apple user accounts owned and managed by an organization (Apple Business Manager / Apple School Manager)
- Required for User Enrollment and Shared iPad
- Education: 200 GB free iCloud storage, Schoolwork integration

### Federated Authentication
- Connects Apple Business Manager / Apple School Manager to Microsoft Azure Active Directory via SAML
- No new Apple credential for users; sign in to Apple services using existing Azure AD credentials
- Managed Apple ID is created automatically on first federated sign-in
- Azure AD can itself federate to on-premise ADFS or other IdPs — Apple sees only Azure AD
- **Domain verification** **[NEW in ABM/ASM]**: DNS TXT record-based ownership verification required before federation; codes valid for 14 days
- Setup process: verify domain → admin grants Azure AD consent → test user sign-in → resolve username conflicts (60-day window) → enable federation

### SCIM Integration (Coming Later in 2020)
- Automated account lifecycle synchronization between Azure AD and Apple Business Manager / Apple School Manager
- Handles account creation, profile updates (name, department), and deletions
- Setup: Apple Business Manager generates a SCIM token and tenant URL; admin configures the Apple Business Manager Enterprise Application in Azure AD Provisioning section
- Status visible in Apple Business Manager Activity sidebar; accounts labeled with SCIM as data source
- Available for all federated organizations; adds to existing SIS/SFTP options in Apple School Manager

## APIs & Frameworks

- **Authentication Services** (`AuthenticationServices`)
  - `ASAuthorizationProviderExtensionAuthorizationRequest` — SSO extension request object
  - `ASAuthorizationProviderExtensionAuthorizationRequest.localizedCallerDisplayName` **[NEW]** — localized name of the calling app
  - `ASAuthorizationProviderExtensionAuthorizationRequest.callerTeamIdentifier` **[NEW]** — team ID of the calling app
  - `ASAuthorizationProviderExtensionAuthorizationRequest.isCallerManaged` **[NEW]** — whether the calling app is MDM-managed
  - `ASAuthorizationProviderAuthorizationOperation.operationLogin` — log-in operation
  - `ASAuthorizationProviderAuthorizationOperation.operationRefresh` — refresh operation
  - `ASAuthorizationProviderAuthorizationOperation.operationLogout` — log-out operation
  - `ASAuthorizationProviderAuthorizationOperation.configurationRemoved` **[NEW]** — called on MDM profile removal; requires iOS 14 / macOS Big Sur SDK
- **MDM payloads / entitlements**
  - `com.apple.developer.associated-domains.mdm-managed` entitlement **[NEW - now public]** — allows MDM to supplement associated domains
  - Per-app VPN Excluded Domains list **[NEW]** — bypass VPN for specified cloud domains
  - User-channel SSO extension profile delivery **[NEW]**
  - Managed Associated Domains payload (macOS) / Managed App configuration (iOS)
  - Kerberos SSO extension MDM payload — configurable `localizedCallerDisplayName`, help text, VPN settings
- **Kerberos / Directory**
  - Built-in Kerberos SSO extension — obtains TGT for Active Directory resources (file servers, printers, PKINIT)
  - Active Directory binding — still required for DFS traversal and AD Certificate payload delivery
- **Apple Business Manager / Apple School Manager APIs**
  - SCIM 2.0 endpoint **[NEW]** — standard REST API for Azure AD to push user lifecycle events
  - Domain verification via DNS TXT records **[NEW]**
  - SCIM token and tenant URL generation

## Code Highlights

New SSO extension properties available on authorization requests:
```swift
var localizedCallerDisplayName: String   // human-readable name of calling app
var callerTeamIdentifier: String         // App Store team ID of calling app
var isCallerManaged: Bool                // whether calling app is MDM-managed
```

New profile removal operation constant (requires iOS 14 / macOS Big Sur SDK):
```swift
// Existing operations:
static let operationLogin: ASAuthorization.OpenIDOperation
static let operationRefresh: ASAuthorization.OpenIDOperation
static let operationLogout: ASAuthorization.OpenIDOperation

// New in iOS 14 / macOS Big Sur:
static let configurationRemoved: ASAuthorizationProviderAuthorizationOperation
```

## Takeaways
- Use local macOS accounts for 1:1 deployments; replace AD binding with the built-in Kerberos extension or SSO extensions for directory connectivity.
- Three new SSO extension capabilities in iOS 14 / macOS Big Sur are particularly important: per-app VPN support for on-premise IdPs, the `configurationRemoved` cleanup operation, and caller identity metadata (`localizedCallerDisplayName`, `callerTeamIdentifier`, `isCallerManaged`).
- Federated authentication (ABM/ASM + Azure AD SAML) plus SCIM automates the full Managed Apple ID lifecycle — creation, profile updates, and deletions — without manual IT intervention.
- The `com.apple.developer.associated-domains.mdm-managed` entitlement is now public and no longer requires DTS approval.

---
_Source: WWDC20 Session 10139 page (abstract, transcript, code samples, and resource links)._
