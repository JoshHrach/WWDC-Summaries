# Explore Apple Business Essentials
**WWDC22 · Session 110335** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110335/)

_Platforms:_ iOS 15+, iPadOS 15+, macOS Monterey 12+, tvOS 15+

## Overview
Apple Business Essentials is a subscription service for U.S.-based small businesses that unifies device management, 24/7 AppleCare+ support, and per-employee iCloud storage into a single plan. Rather than requiring a separate MDM solution, businesses enroll through Apple Business Manager and manage all devices directly from the same console.

The session walks through the complete setup workflow: creating an Apple Business Manager account, subscribing to Apple Business Essentials, configuring settings and purchasing apps, organizing them into Collections assigned to users or groups, and finally the employee enrollment experience on both company-owned and personally-owned devices.

## Key Topics

**Subscription plans** — Two plan types: Employee plans (for devices assigned to specific people, up to 3 devices per employee) and Device plans (for shared devices like conference room displays, kiosks, and loaners). Each plan is configurable with storage tier (up to 200 GB iCloud per employee) and optional AppleCare+ for Business Essentials coverage. Billing is usage-based.

**Settings** — Pre-built configuration profiles pushed automatically to enrolled devices. Categories include Security (FileVault, Application Layer Firewall, Gatekeeper on macOS; passcode policy on iOS), Network (AirPrint, Wi-Fi credentials), and Personalization (Conference Room Display mode for Apple TV, Login Window message for macOS). Settings are created once and applied at the Collection level.

**Apps and Collections** — Apps (from the App Store or Custom Apps) are purchased in Apple Business Manager and assigned to Collections. A Collection bundles settings + apps and is assigned to individual users or smart user groups. When changes are made to a Collection, they are automatically pushed to enrolled devices.

**Smart User Groups** — Automatically populate based on user attributes (division, location, role) synced from Microsoft Azure Directory or Google Workspace, eliminating manual group management at scale.

**Employee enrollment** — Company-owned devices (purchased through Apple or an authorized reseller) present a Work sign-in screen during setup and are automatically configured on first boot. Personally-owned (BYOD) devices can enroll via Settings > General > VPN & Device Management > Sign in with a Work or School Account. Work and personal data are cryptographically separated.

**Device lifecycle management** — Lost or reassigned devices can be signed out, locked, or remotely erased from Apple Business Manager. Employees replacing a device simply sign in with their Managed Apple ID on the new device to restore apps, settings, and iCloud work data.

**Repair and support** — Employees initiate repair requests from within the Essentials app. IT admins approve or deny requests in the Service and Support section of Apple Business Manager. Repair credits are shared across the organization.

## APIs & Frameworks

This session covers an IT administration service, not a developer SDK. Key components and tools referenced:

- **Apple Business Manager** — web portal at `business.apple.com`; requires company DUNS number for enrollment
- **Apple Business Essentials subscription** — managed within Apple Business Manager under the Subscription sidebar item
- **Managed Apple ID** — work identity credential issued through Apple Business Manager; cryptographically separated from personal Apple ID on the same device
- **Apple Business Essentials app (Essentials app)** — installed automatically on enrolled devices; employees use it to install assigned apps and initiate AppleCare+ repair requests
- **Collections** — grouping of Settings + Apps assigned to users or user groups
- **Smart User Groups** — auto-populated groups based on user attributes from directory sync (Azure AD, Google Workspace)
- **FileVault** setting — full-disk encryption enforcement for macOS
- **Application Layer Firewall** setting — macOS firewall configuration
- **Gatekeeper** setting — prevents unsigned/untrusted apps from running on macOS
- **iCloud Drive for Work** — per-employee iCloud storage included with the subscription; used for file access, collaboration, and device backup
- **AppleCare+ for Business Essentials** — shared repair credit pool across the organization

## Code Highlights

No developer APIs or code samples are covered in this session. Apple Business Essentials is configured entirely through the Apple Business Manager web interface.

## Takeaways
- Apple Business Essentials combines MDM, iCloud storage, and AppleCare+ into a single subscription, removing the need for a separate MDM provider for small businesses.
- Collections are the core organizational primitive — bundle settings and apps together, then assign to user groups; any future changes to a Collection automatically propagate to all enrolled devices.
- Employee-owned (BYOD) and company-owned devices are both supported; work and personal data are cryptographically isolated so personal data stays private.
- Smart User Groups powered by Azure AD or Google Workspace sync eliminate most manual group maintenance as organizations grow.

---
_Source: WWDC22 Session 110335 page (abstract, chapter summaries, and resource links)._
