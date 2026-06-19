# Deploy Apple Devices Using Zero-Touch
**WWDC20 · Session 10223** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10223/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session presents Apple's internal zero-touch deployment model as a reference architecture for enterprise IT teams. Presented by an Apple IT administrator, it walks through the end-to-end process of deploying thousands of Apple devices to remote employees without any IT hands-on time — from supply chain purchase through automated MDM enrollment, identity configuration, app distribution, and security hardening.

The session is particularly relevant in the context of the rapid shift to remote work in 2020, where Apple dropped shipped Macs directly to employees' homes and had them productive within minutes of unboxing. The 20,000 managed devices per MDM admin ratio cited demonstrates the scale efficiency of a well-designed zero-touch pipeline.

## Key Topics

**Zero-Touch Model Overview**
New devices are drop-shipped from the supply chain directly to end users. IT never physically touches the device. The user unboxes it, powers it on, and Setup Assistant handles enrollment automatically. The goal: online and productive within minutes, not hours.

**Supply Chain and Apple Business Manager Integration**
Resellers use the Apple Reseller API to register device purchase data (serial numbers, PO details, purchase date) into Apple Business Manager automatically at point of sale. Devices are validated against existing ABM registrations to prevent duplicates. Each device is assigned to an MDM server based on ABM default assignment rules.

**Customizing Setup Assistant**
Organizations can remove or skip specific Setup Assistant screens to streamline the user experience. Devices auto-enroll in MDM on first boot. Users create a local account with enforced password restrictions. At desktop arrival, required apps and security settings are already installed.

**APNs Proxy Support (New in macOS 10.15.4 / iOS 13.4)**
Apple Push Notification Service now supports proxy configurations via PAC (Proxy Auto-Config) files. This enables MDM communication through web proxies on default-deny networks typical in regulated industries. APNs traffic remains encrypted and cannot be inspected through the proxy.

**MDM Capabilities**
MDM can enforce passcodes, restrict settings, configure Wi-Fi, email and VPN, install apps, and enable FileVault encryption. Profiles and policies can be scoped by LDAP group or user role. If a device is lost, stolen, or out of policy: selective profile removal, remote lock (with admin-set PIN), or full remote wipe are available.

**Multi-Environment MDM Infrastructure**
Recommended environments: production, disaster recovery (geographically separate), test, and development. On-premise deployments should consider containers/VMs for web/app servers and bare metal for database-intensive workloads. Load balancers appropriate for larger organizations. Firewalls and ACLs at every layer.

**App Distribution**
Apps distributed via Apple Business Manager (VPP) + MDM — no Apple ID required on the device. Device-based app assignment eliminates per-user invitation flows. macOS Content Caching locally caches app and OS updates for efficiency at scale. Automatically pushed apps should be limited to business-critical tools; discretionary apps offered via self-service portal.

**Managed Apps (New in macOS Big Sur)**
Managed Apps in macOS Big Sur allow users to install previously-used apps via their Apple ID. IT admins can lock specific apps to prevent removal or misuse.

**Identity and Security**
Single Sign-On (SSO) authentication client deployed automatically. MDM integrates with LDAP/identity providers to pull user information and scope profiles by group. Baseline security settings enforced via MDM payloads (FileVault, passcode complexity, VPN, wireless). Additional security settings scoped by user role and data sensitivity. Custom scripts used where MDM lacks native LDAP integration hooks.

## APIs & Frameworks

### Apple Business Manager / MDM
- Apple Business Manager (ABM) — centralized device purchasing, assignment, and management
- Apple Reseller API — automated device registration at point of purchase
- MDM (Mobile Device Management) — remote device configuration, app distribution, security
- Automated Device Enrollment (ADE) — zero-touch MDM enrollment via Setup Assistant
- Configuration Profiles — payloads for security settings, network, email, VPN, restrictions
- Volume Purchase Program (VPP) / Apps and Books — device-based app license assignment
- Apple Push Notification Service (APNs) — MDM-to-device communication channel
- APNs Proxy Support via PAC file **[NEW in macOS 10.15.4 / iOS 13.4]** — MDM communication through web proxies

### macOS Security Features
- FileVault — full-disk encryption, enforceable via MDM
- Remote Lock — locks device with admin-set PIN via MDM
- Remote Wipe — permanently erases all data via MDM command
- Selective Profile/App Removal — removes specific managed content without full wipe
- macOS Content Caching — locally caches Apple content for bandwidth efficiency at scale

### Identity and Directory
- LDAP — directory integration for user identification and group-based policy scoping
- Single Sign-On (SSO) — federated authentication for automatic mail/calendar/VPN configuration
- Managed Apps (macOS Big Sur) **[NEW]** — user-installable managed apps via Apple ID; lockable by IT

### Infrastructure Components
- APNs — MDM push channel; encrypted, proxy-compatible
- PAC (Proxy Auto-Config) — proxy discovery mechanism now supported by APNs
- MDM environments: production, DR, test, development

## Code Highlights

No code samples were provided in this session. The content is IT administration process and infrastructure focused.

## Takeaways
- A mature zero-touch deployment at Apple achieves 20,000 managed devices per MDM admin by combining Apple Business Manager, automated device enrollment, and MDM at every step — from supply chain to desktop.
- APNs now supports proxy configurations via PAC files (new in macOS 10.15.4 / iOS 13.4), enabling full MDM functionality on default-deny corporate networks without sacrificing the encryption that makes APNs secure.
- App distribution should be deliberate: push only business-critical apps automatically at enrollment; offer discretionary apps via a self-service portal to reduce noise and improve user experience.
- Multiple MDM environments (production, DR, test, dev) are essential for safely testing new profiles and policies before rollout; start small and layer complexity over time rather than attempting a fully featured deployment from day one.

---
_Source: WWDC20 Session 10223 page (abstract, chapter summaries, code samples, and resource links)._
