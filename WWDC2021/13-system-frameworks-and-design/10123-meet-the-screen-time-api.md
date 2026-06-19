# Meet the Screen Time API
**WWDC21 · Session 10123** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10123/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
iOS 15 introduces the Screen Time API, a trio of new Swift/SwiftUI frameworks that lets third-party parental controls apps apply the same restrictions as built-in Screen Time, monitor device usage, and execute code on schedules — all with strong privacy guarantees. The three frameworks are:
- **FamilyControls** — authorization gate; prevents unauthorized removal; supplies opaque tokens for apps and websites.
- **ManagedSettings** — apply restrictions (shield apps, block content, lock accounts, filter web traffic) with your own branding.
- **DeviceActivity** — run extension code on time-based schedules or usage-threshold events without the app being foregrounded.

All user activity data (apps used, websites visited) remains on-device and is invisible outside the Family Sharing group.

## Key Topics

**FamilyControls Authorization**
Before a parental controls app can access Screen Time APIs, a Family Sharing guardian must approve it. After approval: the app cannot be removed from the child's device without guardian re-authentication; iCloud sign-out is blocked on the device; Network Extension web content filters bundled in the app are installed automatically.

**Family Activity Picker**
SwiftUI view modifier `.familyActivityPicker(isPresented:selection:)` presents a system UI listing apps, websites, and categories used by the family. Selections are returned as opaque `FamilyActivitySelection` containing `ActivityToken` values — the tokens represent apps and websites without revealing which specific apps or sites to the app.

**DeviceActivity Schedules**
A `DeviceActivitySchedule` defines a repeating time window (start/end date components). Calling `DeviceActivityCenter.shared.startMonitoring(_:during:events:)` registers a schedule. The app's `DeviceActivityMonitor` extension subclass receives `intervalDidStart` and `intervalDidEnd` callbacks the first time the device is used after each schedule boundary — even if the main app never launches.

**DeviceActivity Events**
`DeviceActivityEvent` specifies a set of app/website tokens and a usage `threshold`. When total usage of those tokens within the active schedule reaches the threshold, the extension's `eventDidReachThreshold(_:activity:)` is called. This enables the "unlock apps after completing homework" pattern.

**ManagedSettings Restrictions**
From a `DeviceActivityMonitor` extension or the main app (with Family Controls authorization), the app configures a `ManagedSettingsStore` to apply restrictions, including:
- `shield.applications` — set of `ApplicationToken` to block with a shield overlay
- `shield.webDomains` — shield specific web domains
- `account.lockAccounts` — prevent account creation/removal
- `appStore.denyAppInstallation` — prevent App Store installs
- Content rating filters for movies and TV (available to any app without Family Controls authorization)

**Custom Shields**
Two extension points in ManagedSettings customize shield appearance and behavior:
- `ShieldConfigurationProvider` — override `configuration(for:)` to return a `ShieldConfiguration` struct with background effect/color, icon, title, subtitle, and button labels.
- `ShieldActionHandler` — override `handle(_:for:completionHandler:)` to respond to primary/secondary button taps with `.close` (close the shielded app) or `.defer` (redraw shield — useful for "Ask for Access" flows).

## APIs & Frameworks

### FamilyControls Framework **[NEW]**
- `AuthorizationCenter.shared.requestAuthorization() async throws` — guardian authentication gate **[NEW]**
- `FamilyActivitySelection` — opaque selection containing sets of tokens **[NEW]**
  - `applicationTokens: Set<ApplicationToken>`
  - `webDomainTokens: Set<WebDomainToken>`
  - `categoryTokens: Set<ActivityCategoryToken>`
- `.familyActivityPicker(isPresented:selection:)` — SwiftUI view modifier **[NEW]**
- Xcode capability: "Family Controls" — add via Signing & Capabilities

### DeviceActivity Framework **[NEW]**
- `DeviceActivityName: RawRepresentable<String>` — identifier for a monitored activity **[NEW]**
- `DeviceActivitySchedule` — `intervalStart: DateComponents`, `intervalEnd: DateComponents`, `repeats: Bool` **[NEW]**
- `DeviceActivityEvent` — `applications`, `webDomains`, `categories`, `threshold: DateComponents` **[NEW]**
- `DeviceActivityCenter.shared.startMonitoring(_:during:events:) throws` **[NEW]**
- `DeviceActivityCenter.shared.stopMonitoring(_:)` **[NEW]**
- `DeviceActivityMonitor: NSExtensionPrincipalClass` — extension subclass **[NEW]**
  - `intervalDidStart(for:)` — schedule start callback **[NEW]**
  - `intervalDidEnd(for:)` — schedule end callback **[NEW]**
  - `eventDidReachThreshold(_:activity:)` — usage threshold callback **[NEW]**

### ManagedSettings Framework **[NEW]**
- `ManagedSettingsStore` — central settings object for applying restrictions **[NEW]**
  - `shield.applications: Set<ApplicationToken>?` — nil removes shield **[NEW]**
  - `shield.webDomains: Set<WebDomainToken>?` **[NEW]**
  - `account.lockAccounts: Bool` **[NEW]**
  - `appStore.denyAppInstallation: Bool` **[NEW]**
  - `media.denyExplicitContent: Bool` **[NEW]**
  - `media.denyMovies(above:)` / `media.denyTVShows(above:)` — content rating filters (no Family Controls authorization required) **[NEW]**
- `ShieldConfigurationProvider: NSExtensionPrincipalClass` **[NEW]**
  - `configuration(for: ApplicationToken) -> ShieldConfiguration` **[NEW]**
  - `ShieldConfiguration` — `backgroundEffect`, `backgroundColor`, `icon`, `title`, `subtitle`, `primaryButtonLabel`, `primaryButtonBackgroundColor`, `secondaryButtonLabel` **[NEW]**
- `ShieldActionHandler: NSExtensionPrincipalClass` **[NEW]**
  - `handle(_: ShieldAction, for: ApplicationToken, completionHandler: (ShieldActionResponse) -> Void)` **[NEW]**
  - `ShieldActionResponse` — `.close` or `.defer` **[NEW]**

## Code Highlights

Request Family Controls authorization:
```swift
import FamilyControls

try await AuthorizationCenter.shared.requestAuthorization()
```

Set up a daily Device Activity schedule and register encouraged-app event:
```swift
import DeviceActivity

let center = DeviceActivityCenter()
let schedule = DeviceActivitySchedule(
    intervalStart: DateComponents(hour: 0, minute: 0),
    intervalEnd: DateComponents(hour: 23, minute: 59),
    repeats: true
)
let encouragedEvent = DeviceActivityEvent(
    applications: selection.applicationTokens,
    threshold: DateComponents(hour: 1)
)
try center.startMonitoring(.daily, during: schedule, events: [.encouraged: encouragedEvent])
```

Shield apps from a DeviceActivityMonitor extension:
```swift
import ManagedSettings

class HomeworkMonitor: DeviceActivityMonitor {
    let store = ManagedSettingsStore()
    
    override func intervalDidStart(for activity: DeviceActivityName) {
        let selection = AppModel.shared.discouragedSelection
        store.shield.applications = selection.applicationTokens
    }
    
    override func intervalDidEnd(for activity: DeviceActivityName) {
        store.shield.applications = nil
    }
    
    override func eventDidReachThreshold(_ event: DeviceActivityEvent.Name,
                                         activity: DeviceActivityName) {
        store.shield.applications = nil  // remove shield when homework threshold met
    }
}
```

Custom shield configuration:
```swift
class HomeworkShieldProvider: ShieldConfigurationProvider {
    override func configuration(for application: ApplicationToken) -> ShieldConfiguration {
        ShieldConfiguration(
            backgroundColor: .systemIndigo,
            icon: UIImage(named: "HomeworkIcon"),
            title: ShieldConfiguration.Label(text: "Time to do homework!", color: .white),
            primaryButtonLabel: ShieldConfiguration.Label(text: "Ask for Access", color: .white),
            primaryButtonBackgroundColor: .systemPurple
        )
    }
}
```

## Takeaways
- The Screen Time API requires all three frameworks together: FamilyControls for authorization, DeviceActivity for background code execution, and ManagedSettings for applying restrictions.
- `DeviceActivityMonitor` extension callbacks fire even when the parent app has never been opened since device setup — this is the correct model for restrictions that need to persist on a child's device.
- Opaque `ActivityToken` values mean the app never learns which specific apps or sites users have on their device; all usage data stays on-device within the Family Sharing group.
- `ShieldActionHandler` with `.defer` response enables interactive shield flows (e.g., "Ask for Access" waiting on a guardian push notification) without closing or removing the shield.

---
_Source: WWDC21 Session 10123 page (abstract, chapter summaries, code samples, and resource links)._
