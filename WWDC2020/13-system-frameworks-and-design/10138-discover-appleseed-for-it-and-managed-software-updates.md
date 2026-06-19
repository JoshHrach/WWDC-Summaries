# Discover AppleSeed for IT and Managed Software Updates
**WWDC20 · Session 10138** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10138/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14 (IT/enterprise administration focus)

## Overview
This session targets IT administrators in enterprise and education environments rather than app developers. It covers two major topics: participating effectively in the AppleSeed for IT beta program to test pre-release Apple software, and using MDM (Mobile Device Management) to control software update deployment timing across an organization.

A major new feature in Feedback Assistant is **Teams** — a collaboration layer that lets multiple members of an organization share, view, and respond to feedback submitted to Apple. Teams are configured through Apple Business Manager or Apple School Manager and enable reassignment of feedback when team members leave. A companion feature, **multi-device diagnostics**, allows a single feedback submission to gather sysdiagnose logs from multiple devices simultaneously.

On the managed software updates side, macOS Big Sur unifies its installation technology with iOS/iPadOS and introduces Authenticated APFS (cryptographic sealing of the system volume). The session also covers deferral policies and the deprecation of indefinite update suppression.

## Key Topics

### AppleSeed for IT Program
- Separate beta program from Developer Program and Public Beta, focused on IT admin workflows.
- Provides in-depth beta documentation, release notes, and IT-specific test plans.
- Enroll with a Managed Apple ID at appleseed.apple.com; accept program terms annually.
- Configuration profiles or the macOS Customer Beta Access Utility (downloaded from appleseed.apple.com) direct devices to pre-release updates.
- Focus feedback on **deployment blockers** and **regressions** rather than general usability issues.
- AppleSeed Discussion Forums enable conversation with other participants.

### Feedback Assistant: Teams **[NEW in 2020]**
- Teams allow multiple organization members to collaborate on feedback submitted to Apple.
- Configured via Apple Business Manager / Apple School Manager (AppleSeed for IT) or App Store Connect (Developer Program).
- Team members can view all team feedback, see Apple responses, and participate in back-and-forth.
- Feedback can be **reassigned** to another teammate (e.g., when someone leaves the company).
- Administrator-only actions: move feedback to/from team space, reassign, close teammates' feedback.
- Available in iOS 14, iPadOS 14, macOS Big Sur, and feedbackassistant.apple.com.

### Feedback Assistant: Multi-Device Diagnostics **[NEW]**
- When filing a new feedback on one device, Feedback Assistant can remotely trigger sysdiagnose collection on other devices signed into the same iCloud account.
- Diagnostics upload independently from each device directly to Apple — no need to wait for sync.
- Particularly valuable for Continuity, AirDrop, and syncing issues that require logs from multiple devices.

### Managed Software Updates: iOS, iPadOS, tvOS
- Software update deferral controlled via MDM on supervised devices.
- Default deferral: 30 days from Apple's release date.
- Configurable range: 1–90 days via MDM restriction.
- Deferral windows are date-based, not version-based.
- User sees a restricted update screen in Settings when an update is deferred.
- Downgrade not supported; only newer OS versions installable; older signed builds may be revoked.

### Managed Software Updates: macOS Big Sur **[NEW behaviors]**
- MDM commands: schedule update scan, fetch available updates list, get update status, schedule installation.
- Remote management requires supervision.
- Configuration profile can defer updates up to 90 days; Mac does not require supervision for profile-based deferral.
- Deferral is now supported during seeding (previously not available on macOS) **[NEW]**.
- Major macOS releases now deferrable under the same 90-day window (introduced in Catalina 10.15.4, continued in Big Sur).
- **Indefinite update suppression deprecated in macOS Big Sur** — the `ignore` MDM key no longer supported going forward (Catalina/Mojave retain it in aligned security releases under supervision).
- Third-party software update catalogs removed in macOS Big Sur.

### macOS Big Sur Security Changes
- Unified installation technology with iOS/iPadOS.
- System volume is snapshotted; the snapshot is patched while the user works (like iOS OTA behavior).
- **Authenticated APFS**: system snapshot is cryptographically sealed; blocks are hashed up the file system chain to a root hash signed by Apple; mismatch causes kernel panic.
- Disabling authenticated APFS requires modifying security settings in Recovery OS (strongly discouraged).
- Update qualification is now server-driven as on iOS.

## APIs & Frameworks

### MDM (Mobile Device Management)
- Software update deferral restriction key: `forceDelayedSoftwareUpdates` (iOS/iPadOS/tvOS/macOS)
- Deferral period key: `enforcedSoftwareUpdateDelay` (1–90 days)
- macOS-specific MDM commands: `AvailableOSUpdates`, `ScheduleOSUpdate`, `OSUpdateStatus`
- Supervision required for remote update commands on all platforms
- Content caching server support for bandwidth-constrained organizations

### Feedback Assistant
- App available on iOS, iPadOS, macOS, and web (feedbackassistant.apple.com)
- Teams feature: configured via Apple Business Manager / Apple School Manager
- Multi-device diagnostics: remote sysdiagnose collection via iCloud account
- Roles: Participate in AppleSeed for IT (privilege), Administer AppleSeed for IT (admin privilege)

### Apple Business Manager / Apple School Manager
- `Participate in AppleSeed for IT` privilege — required for all testers
- `Administer AppleSeed for IT` privilege — required for team administrators
- Student accounts in Apple School Manager excluded from AppleSeed participation

## Code Highlights
No code samples — this session is for IT administrators using MDM configuration profiles and Apple Business Manager, not for app developers.

## Takeaways
- Enroll in AppleSeed for IT with a Managed Apple ID to access pre-release software and IT-focused documentation; file deployment blockers and regressions early.
- Use the new Teams feature in Feedback Assistant to share organizational feedback with teammates and prevent loss of context when employees leave.
- On macOS Big Sur, indefinite update suppression is gone — adopt time-based deferral (up to 90 days) and qualify updates during the seeding period.
- macOS Big Sur's Authenticated APFS cryptographically seals the system volume, significantly hardening the OS against tampering.

---
_Source: WWDC20 Session 10138 page (abstract, transcript, and resource links)._
