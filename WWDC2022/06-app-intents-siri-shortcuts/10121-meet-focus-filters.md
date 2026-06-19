# Meet Focus Filters
**WWDC22 · Session 10121** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10121/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
Focus filters, introduced in iOS 16 and macOS Ventura, let developers customize app behavior in response to the user's currently active Focus mode. Apps declare what can be configured per-Focus by implementing `SetFocusFilterIntent` from the App Intents framework. Users configure the filter values in Focus Settings; the system delivers those values to the app (or a dedicated app extension) whenever the Focus changes.

System apps like Calendar and Mail demonstrate the pattern: Calendar filters which calendars appear when a Focus is active, and Mail filters which mailboxes and notifications are shown. Third-party apps can use Focus filters to switch accounts, surface themed layouts, reduce badge counts, and silence unrelated notifications — any situation where an app can adapt its content to a user's current context.

## Key Topics
- **`SetFocusFilterIntent`** — the protocol an app implements to declare its Focus filter; contains `title`, `description`, `@Parameter` properties, `displayRepresentation`, and optionally `perform()` and `appContext`
- **Parameters** — declared as `@Parameter`-decorated properties on the intent; support standard Swift types (`Bool`, `String`, `Float`, etc.) and custom `AppEntity` types; can be required (with a default) or optional (`?`); users configure per-Focus values in Focus Settings
- **`displayRepresentation`** — dynamic property returning a `DisplayRepresentation` with primary and secondary strings reflecting the currently configured parameter values; shown in the Focus Settings grid
- **Delivery mechanism** — when a Focus activates or deactivates, the system calls `perform()` on the FocusFilterIntent if the app is running, or launches a dedicated app extension and calls `perform()` there; apps that only need to update their own views do not need an extension; apps that need to update widgets, badges, or notifications should implement the extension
- **`FocusFilterIntent.current`** — async property to imperatively read the currently active Focus filter parameters at any time, without waiting for `perform()` to be called
- **`FocusFilterAppContext`** — returned from `perform()` (or the `appContext` property) to give the system additional context: specifically a `notificationFilterPredicate` (`NSPredicate`) that silences notifications whose `filterCriteria` string does not match
- **`UNMutableNotificationContent.filterCriteria`** — new string property on local and remote notifications; the system compares this against the active Focus filter's predicate to determine whether to silence the notification
- **`UNUserNotificationCenter.setBadgeCount(_:)`** — new API to programmatically set the app badge count, allowing the app to surface only relevant badge counts for the active Focus

## APIs & Frameworks
**App Intents framework — Focus** **[NEW]**
- `SetFocusFilterIntent` protocol **[NEW]** — base protocol for Focus filter declarations; import `AppIntents`
- `SetFocusFilterIntent.title: LocalizedStringResource` — static; shown in Focus Settings before configuration
- `SetFocusFilterIntent.description: LocalizedStringResource?` — static; shown in the configuration detail view
- `SetFocusFilterIntent.displayRepresentation: DisplayRepresentation` — dynamic; primary + secondary text shown after configuration
- `@Parameter(title:default:)` — property wrapper **[NEW in Focus context]** — decorates a property as user-configurable per Focus; works with `Bool`, `String`, numeric types, and `AppEntity` conforming types; optional parameters use `?` type
- `SetFocusFilterIntent.perform() async throws -> some IntentResult` — called by the system on Focus transition; read `self.<parameter>` to access configured values; return `.result()` or `.result(appContext:)`
- `SetFocusFilterIntent.current` **[NEW]** — `static async throws -> Self` — returns the currently active filter's configured values
- `FocusFilterAppContext` **[NEW]** — object returned alongside an intent result to provide context to the system
  - `FocusFilterAppContext(notificationFilterPredicate: NSPredicate?)` — filter predicate for silencing notifications
- `SetFocusFilterIntent.appContext: FocusFilterAppContext` — property that can be returned at any time to push updated context; call `invalidate()` to force the system to re-read it

**User Notifications**
- `UNMutableNotificationContent.filterCriteria: String?` **[NEW]** — set on local notifications to declare which account/context the notification belongs to; silenced if it does not match the active Focus filter's predicate
- Remote notification JSON payload — `filter-criteria` key for the same purpose on push notifications
- `UNUserNotificationCenter.setBadgeCount(_ newBadgeCount: Int) async throws` **[NEW]** — replaces the badge with a Focus-appropriate count

## Code Highlights
Defining a Focus filter with parameters:
```swift
import AppIntents

struct ExampleChatAppFocusFilter: SetFocusFilterIntent {
    static var title: LocalizedStringResource = "Set account, status & look"
    static var description: LocalizedStringResource? = """
        Select an account, set your status, and configure
        the look of Example Chat App.
    """

    @Parameter(title: "Use Dark Mode", default: false)
    var alwaysUseDarkMode: Bool

    @Parameter(title: "Status Message")
    var status: String?

    @Parameter(title: "Selected Account")
    var account: AccountEntity?
}
```

Dynamic display representation:
```swift
var displayRepresentation: DisplayRepresentation {
    var titleList: [LocalizedStringResource] = []
    var subtitleList: [String] = []
    if let account { titleList.append("Account"); subtitleList.append(account.displayName) }
    if let status  { titleList.append("Status");  subtitleList.append(status) }
    titleList.append("Look")
    subtitleList.append(alwaysUseDarkMode ? "Dark" : "Dynamic")
    return DisplayRepresentation(
        title: "Set \(titleList, format: .list(type: .and))",
        subtitle: "\(subtitleList.formatted())"
    )
}
```

Responding to a Focus change:
```swift
func perform() async throws -> some IntentResult {
    let myData = AppData(alwaysUseDarkMode: self.alwaysUseDarkMode,
                        status: self.status,
                        account: self.account)
    myModel.shared.updateAppWithData(myData)
    return .result()
}
```

Imperatively reading the current filter:
```swift
let currentFilter = try await ExampleChatAppFocusFilter.current
myModel.shared.updateAppWithData(AppData(from: currentFilter))
```

Setting a notification filter predicate and tagging a notification:
```swift
// In the Focus filter — return a predicate matching only the active account
var appContext: FocusFilterAppContext {
    let predicate = NSPredicate(format: "SELF IN %@", [account?.identifier ?? ""])
    return FocusFilterAppContext(notificationFilterPredicate: predicate)
}

// When scheduling a notification for a specific account
let content = UNMutableNotificationContent()
content.title = "Curt Rothert"
content.body = "The run through today was great."
content.filterCriteria = "work-account-identifier"  // silenced if not in predicate
```

## Takeaways
- Focus filters let apps declare configurable per-Focus behaviors via `SetFocusFilterIntent`; users set them up once in Focus Settings, and the system delivers the values via `perform()` on every Focus transition.
- Apps that only update their own UI implement `perform()` in the app; apps that need to update widgets, notifications, or badges should also implement the extension target.
- `FocusFilterAppContext.notificationFilterPredicate` + `UNMutableNotificationContent.filterCriteria` provides a lightweight notification silencing mechanism that does not require the app to be running.
- `UNUserNotificationCenter.setBadgeCount(_:)` replaces the old badge API and enables Focus-aware badge counts without scheduling a dummy notification.

---
_Source: WWDC22 Session 10121 page (abstract, chapter summaries, code samples, and resource links)._
