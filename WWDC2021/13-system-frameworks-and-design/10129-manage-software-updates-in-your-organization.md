# Manage Software Updates in Your Organization
**WWDC21 · Session 10129** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10129/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
In managed device environments, organizations need to control the pace of software updates while testing compatibility with their existing software and workflows. This session covers the full lifecycle of managed updates: beta testing with AppleSeed for IT, deferring updates during the testing window, deploying updates via MDM commands, and enforcing updates once they're approved.

macOS Monterey introduces significant parity improvements between macOS and iOS update management, including unified version-based update deployment via the Apple Software Lookup Service, bootstrap token support for automated InstallLater flows on Apple silicon, and a new MaxUserDeferrals enforcement mechanism. iOS 15 also adds control over which software update version is recommended to users.

A key theme is minimizing disruption to users while keeping devices secure and current. Admins can now schedule updates for overnight windows, notify users of the remaining deferrals before a forced update, and precisely control what update versions appear in the iOS Settings app.

## Key Topics

**Beta Testing with AppleSeed for IT**
AppleSeed for IT allows any non-student Managed Apple ID from Apple School Manager or Apple Business Manager to access and test pre-release Apple software, with test plans and feedback channels.

**Deferring Software Updates**
Deferral restrictions prevent supervised devices from surfacing updates to users for a configurable delay (1–90 days, default 30) after Apple publishes them. Deferral does not block MDM commands. macOS 11.3+ allows independently deferring major, minor, and supplemental updates using separate profile keys.

**Deploying Updates via MDM**
Prior to macOS Monterey, Macs required a `ScheduleOSUpdateScan` round-trip to determine eligibility. macOS Monterey aligns with iOS by supporting the Apple Software Lookup Service and the `ProductVersion` key in `ScheduleOSUpdate`. A new `SoftwareUpdateModelID` DeviceInformation key enables server-side eligibility checks without device round-trips for minor updates.

**Enforcing Updates (macOS Monterey)**
A new `MaxUserDeferrals` key in `ScheduleOSUpdate` lets admins specify how many times a user can defer an InstallLater update before it is forced. Users receive progressive notifications showing remaining deferrals.

**iOS Recommendation Cadence**
A new `RecommendationCadence` key in the `SoftwareUpdateSettings` dictionary controls which update version (major upgrade vs. current-major security update) is presented to the user in Settings.

## APIs & Frameworks

**MDM Profile Keys (macOS)**
- `forceDelayedMajorSoftwareUpdates` — defer major OS releases **[NEW in macOS 11.3]**
- `forceDelayedSoftwareUpdates` — defer minor OS releases
- `forceDelayedAppSoftwareUpdates` — defer supplemental updates
- `ManagedDeferredInstallDelay` — fallback deferral period key

**MDM Commands**
- `ScheduleOSUpdateScan` — triggers device-side scan for available updates (pre-Monterey Mac)
- `ScheduleOSUpdate` — deploy updates; install action values:
  - `InstallASAP` — immediate install, user can cancel; uses bootstrap token on Apple silicon **[NEW bootstrap token support]**
  - `InstallLater` — schedules overnight install (2–4 a.m. window based on ML usage patterns)
  - `DownloadOnly` — background download without install
  - `Default` — primary mechanism for iOS/iPadOS
  - `NotifyOnly` — alerts users, no install
  - `InstallForceRestart` — forced hard restart for userless devices
- `MaxUserDeferrals` — max times user may defer InstallLater before forced update **[NEW in macOS Monterey]**

**MDM DeviceInformation Query Keys**
- `SoftwareUpdateModelID` — returns hardware model string for use with Apple Software Lookup Service **[NEW in macOS Monterey]**

**MDM Settings Command**
- `SoftwareUpdateSettings` dictionary with `RecommendationCadence` key **[NEW in iOS 15]**
  - `0` — default, shows both major and minor updates
  - `1` — shows lower version number (stay on current major)
  - `2` — shows higher version number (upgrade to next major)

**Apple Software Lookup Service**
- JSON feed providing available OS versions with hardware device IDs for eligibility checks across iOS, iPadOS, and macOS Monterey

**Bootstrap Token (Apple Silicon)**
- Required for automated non-interactive updates on Apple silicon (macOS 11.2+)
- Now supports MDM-initiated InstallLater flows **[NEW in macOS Monterey]**
- Requires update signed by Apple

**Existing Keys**
- `enforceSoftwareDelayUpdate` — existing deferral key (interacts with RecommendationCadence)
- `ProductVersion` — now supported on macOS Monterey (previously Mac-only used `ProductKey`)
- `ProductKey` — still takes precedence if both keys are specified

## Code Highlights

No sample code is provided in this session. The session is focused on MDM configuration profiles and MDM command payloads rather than app-level APIs.

Key MDM command payload (conceptual):
```xml
<!-- ScheduleOSUpdate with MaxUserDeferrals -->
<key>InstallAction</key>
<string>InstallLater</string>
<key>MaxUserDeferrals</key>
<integer>3</integer>
```

Profile deferral keys (macOS Monterey):
```xml
<key>forceDelayedMajorSoftwareUpdates</key>
<true/>
<key>enforcedSoftwareUpdateMajorOSDeferredInstallDelay</key>
<integer>30</integer>

<key>forceDelayedSoftwareUpdates</key>
<true/>
<key>enforcedSoftwareUpdateMinorOSDeferredInstallDelay</key>
<integer>14</integer>
```

## Takeaways
- macOS Monterey achieves near parity with iOS for MDM-based software update management, using the Apple Software Lookup Service and `ProductVersion` for server-side eligibility determination without device scan round-trips.
- The new `MaxUserDeferrals` key gives admins a way to enforce updates on macOS while still giving users advance notice, reducing surprise forced restarts.
- Bootstrap token support for `InstallLater` on Apple silicon enables fully automated overnight updates without disrupting active users.
- The `RecommendationCadence` setting on iOS 15/iPadOS 15 lets admins control whether users are steered toward the current major release security updates or prompted to upgrade to the next major OS.

---
_Source: WWDC21 Session 10129 page (abstract, chapter summaries, code samples, and resource links)._
