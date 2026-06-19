# Using Grouped Notifications
**WWDC18 · Session 711** · [Watch](https://developer.apple.com/videos/play/wwdc2018/711/)

_Platforms:_ iOS 12

## Overview
A detailed implementation guide for Notification Grouping — the iOS 12 feature that stacks multiple notifications from the same app into a single group in Notification Center. The session covers the full API surface: default app grouping, custom thread-based grouping via `threadIdentifier`, custom summary format strings per category, summary arguments for dynamic content (names, counts), bundled-notification counting with `summaryArgumentCount`, and internationalization of plural summary strings using `.stringsdict` files.

Three real iOS 12 apps are used as design case studies — Calendar, Messages, and Mail — each demonstrating a different grouping strategy aligned to how users consume that app's notifications. The session also covers how grouped notifications interact with content extensions (rich notifications) and the correct way to clean up additional notifications when an extension dismisses.

## Key Topics

### Default App Grouping
- Without any code changes, iOS 12 groups all notifications from an app into a single group (the "app group")
- The group displays: leading notification (most recent) at top; brief summary below; notification count
- Users can swipe to clear the entire group, or tap to expand and manage individual notifications
- App grouping is sufficient for apps with homogeneous notifications (e.g., Podcasts episode alerts)

### Custom Groups via `threadIdentifier` **[NEW in iOS 12 — note: API existed earlier for other features]**
- Set `UNMutableNotificationContent.threadIdentifier` to any unique string; all notifications with the same string go to the same group
- `threadIdentifier` previously existed for Watch complications and notification privacy; existing adopters should review their grouping since the behavior is now visible in iOS 12's Notification Center
- Design patterns from first-party apps:
  - **Calendar**: one app group for bulk informational updates (no thread identifier); separate thread for urgent actional alerts (event reminders, time-to-leave). Lesson: separate important/actionable from informative updates.
  - **Messages**: one thread per conversation (1:1 and group chats get separate identifiers). Lesson: create groups for meaningful personal communications that are short-lived.
  - **Mail**: one group per account; VIP contacts' emails extracted to a separate group; thread-subscribed emails extracted to their own group. Lesson: respect the user's explicit priorities and organization choices.

### Custom Summary Strings
- Each notification group shows a summary (e.g., "9 more notifications") — customize it per notification category
- Set `UNNotificationCategory.categorySummaryFormat` with a format string containing `%u` (numeric placeholder)
- Example: `"%u more messages"` → "9 more messages"
- The `categorySummaryFormat` is on the category (type), not the content, because a category represents a type of notification — different categories can have different summaries

### Summary Arguments (Dynamic Name Lists)
- For summaries that include people's names (e.g., "5 more messages from Alice, Bob"), use a format with `%u` and `%@` placeholders: `"%u more messages from %@"`
- Set `UNMutableNotificationContent.summaryArgument` to the sender's name on each notification
- iOS collects all `summaryArgument` values from notifications in the group and de-duplicates them before interpolating into the `%@` placeholder

### Bundled Notification Counts (`summaryArgumentCount`)
- Some apps bundle multiple items into a single notification (e.g., "3 new episodes available")
- Set `UNMutableNotificationContent.summaryArgumentCount` to the number of items that notification represents
- iOS sums all `summaryArgumentCount` values in the group for the `%u` placeholder in the summary
- Defaults to 1 if not set; only set it when a single notification represents multiple items

### Plural Localization with `.stringsdict`
- Summary format strings contain numbers, requiring correct singular/plural forms ("1 more message" vs. "5 more messages")
- Use `NSLocalizedString` variant for notification summaries (stored for later delivery when system locale may change)
- Define plural forms in a `.stringsdict` (property list) file; Foundation automatically selects the correct form based on the number
- Different languages have different plural rule counts (English: 2, Hebrew: 3, Russian: 3 with different rules) — `.stringsdict` handles all of them without per-language code changes
- Supported format string patterns: (1) `%u` only — numeric placeholder; (2) `%u` + `%@` — numeric + argument list; these are the only two formats supported

### Notification Groups + Content Extensions
- When a user long-presses a group to view the rich notification, the content extension receives the **leading notification** (most recent in group) via the standard `didReceive` method
- Extensions can load additional notifications from the delivered set (using `UNUserNotificationCenter.getDeliveredNotifications`) or from the extension's own API
- While the extension is open, new notifications delivered to the same thread are forwarded to the extension via the same `didReceive` method
- When dismissing, extensions should remove any additional notifications they displayed from Notification Center to keep it organized

## APIs & Frameworks

**UserNotifications (new/updated in iOS 12)**
- `UNMutableNotificationContent.threadIdentifier` — **[existed pre-iOS 12, now affects Notification Center grouping]** string that determines which group a notification belongs to
- `UNMutableNotificationContent.summaryArgument` **[NEW]** — string value (e.g., sender name) contributed to the group's summary `%@` placeholder
- `UNMutableNotificationContent.summaryArgumentCount` **[NEW]** — number of items this notification represents for group summary counting; defaults to 1
- `UNNotificationCategory(identifier:actions:intentIdentifiers:hiddenPreviewsBodyPlaceholder:categorySummaryFormat:options:)` **[NEW `categorySummaryFormat` parameter]** — format string with `%u` (and optionally `%@`) for the group summary
- `UNNotificationCategory.categorySummaryFormat` **[NEW]** — the format string used to describe the notification group summary
- `UNNotificationCategory.hiddenPreviewsBodyPlaceholder` — (existing iOS 11 API) text shown when notifications are set to private; similar to summary format but different context

**UserNotificationsUI**
- `UNNotificationContentExtension` — receives `didReceive(_ notification:)` with the leading notification when group is long-pressed; additional notifications delivered to open extension via same method

**Foundation (Localization)**
- `.stringsdict` file — property list defining plural rules for format strings; Foundation selects correct plural form automatically based on count and locale
- `NSLocalizedString` variants for notification content — stores localization keys rather than resolved strings, allowing correct resolution at delivery time if locale changes

**Push Notification Payload (APNs)**
- `thread-id` key — equivalent to `threadIdentifier`
- `summary-arg` key — equivalent to `summaryArgument`
- `summary-arg-count` key — equivalent to `summaryArgumentCount`

## Code Highlights

Setting a thread identifier (custom group) on a local notification:
```swift
let content = UNMutableNotificationContent()
content.title = "New message from Alice"
content.body = "Hey, are you coming to lunch?"
content.threadIdentifier = "conversation-alice-123"  // all messages in this thread group together
```

Setting a category with a custom summary format:
```swift
let category = UNNotificationCategory(
    identifier: "MESSAGE",
    actions: [],
    intentIdentifiers: [],
    hiddenPreviewsBodyPlaceholder: NSLocalizedString("%u more messages", comment: ""),
    categorySummaryFormat: NSLocalizedString("%u more messages from %@", comment: ""),
    options: []
)
UNUserNotificationCenter.current().setNotificationCategories([category])
```

Setting summary argument and count for a bundled notification:
```swift
let content = UNMutableNotificationContent()
content.categoryIdentifier = "PODCAST_EPISODES"
content.title = "Daily Tech News"
content.body = "3 new episodes available"
content.summaryArgument = "Daily Tech News"   // show in summary as sender/source name
content.summaryArgumentCount = 3              // this notification counts as 3 items
```

APNs push payload for grouped notification:
```json
{
  "aps": {
    "alert": { "title": "New message", "body": "Hey!" },
    "thread-id": "conversation-alice-123",
    "summary-arg": "Alice",
    "summary-arg-count": 1
  }
}
```

## Takeaways
- Set `threadIdentifier` thoughtfully — it existed before iOS 12 and existing adopters should review whether their grouping makes sense in the context of Notification Center stacking.
- Match grouping granularity to notification lifetime and volume: short-lived personal messages warrant many fine-grained groups; high-volume informational updates are better collapsed into one or a few coarse groups.
- Always provide `categorySummaryFormat` — the default "N more notifications" is generic and less useful than "N more messages from Alice, Bob" for your specific content type.
- Use `.stringsdict` for all notification summary strings that contain numbers — it handles all locale plural rules automatically without per-language code changes.

---
_Source: WWDC18 Session 711 page (abstract, full transcript, and resource links)._
