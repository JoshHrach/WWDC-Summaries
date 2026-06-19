# Designing Notifications
**WWDC18 · Session 806** · [Watch](https://developer.apple.com/videos/play/wwdc2018/806/)

_Platforms:_ iOS 12, watchOS 5

## Overview
This design-focused session covers how to create notifications that users will want to receive rather than disable. Presenters from Apple's Human Interface team walk through the notification design principles underlying iOS 12's major notification overhaul: grouped notifications by thread or app, provisional authorization for a "try before you ask" permission model, critical alerts for high-priority health/safety notifications, and practical design guidance on notification content, frequency, and timing.

The session pairs directly with the engineering session "What's New in User Notifications" and addresses the human motivations behind notification design choices — why users disable notifications and how to earn their trust.

## Key Topics

### Why Notifications Fail

- The number one reason users disable notifications is receiving too many irrelevant ones.
- A notification should be timely, personal, and actionable — if it fails any of these tests, reconsider sending it.
- Irrelevant, repeated, or marketing-only notifications train users to disable all notifications from an app.
- Notifications should always communicate something the user wants to know right now, not just when it is convenient for the app to send.

### iOS 12 Notification Grouping

- iOS 12 groups notifications into stacks by default — per app, or per thread identifier when `threadIdentifier` is set on `UNMutableNotificationContent`.
- **Thread grouping** — set `threadIdentifier` to a conversation ID, topic, or entity; all notifications with the same identifier collapse into one group in Notification Center.
- **Summary text** — set `summaryArgument` and `summaryArgumentCount` on the content to customize the group summary line (e.g., "3 more messages from Sarah").
- `summaryFormat` on `UNNotificationCategory` — defines the per-category summary template string using `%u` (count) and `%@` (argument) placeholders.
- Groups are collapsible and expandable; the most recent notification in a group appears on top.
- Design notifications in a series to read coherently when stacked — the group summary should make sense when individual notifications are collapsed.

### Provisional Authorization (iOS 12 New)

- Apps can request `.provisional` authorization without displaying a permission prompt.
- Provisional notifications are delivered **quietly** — no sound, no banner, no badge — and appear only in Notification Center.
- Users see "Keep" / "Turn Off" options on provisional notifications in Notification Center; tapping "Keep" upgrades to full authorization.
- Use provisionally to demonstrate notification value before asking for full permission. Do not immediately escalate by requesting full authorization after provisional.
- Provisional authorization is appropriate for apps that have not yet established a relationship with the user — let the notifications speak for themselves.

### Critical Alerts (iOS 12 New)

- Critical alerts play sound and appear even when Do Not Disturb is enabled or the device is silenced — reserved for health, safety, and public-safety use cases.
- Requires an **entitlement** from Apple; not available to all apps.
- Set `UNNotificationContent.sound` to `UNNotificationSound.defaultCritical` or `UNNotificationSound.criticalSoundNamed(_:withAudioVolume:)`.
- Use extremely sparingly — one false-positive critical alert damages user trust permanently.

### Notification Content Design

- **Title** — the most prominent element; state the source and key fact (e.g., "Sarah: Your package is out for delivery").
- **Subtitle** — secondary context; optional. Avoid repeating the title.
- **Body** — full details; should stand alone if title is the only text read.
- **Attachment** — images, audio, video via `UNNotificationAttachment`; makes notifications more recognizable and skimmable in a group.
- **Actions** — up to 4 actions via `UNNotificationAction` registered on `UNNotificationCategory`; place the most expected action first so it appears without expanding.
- Destructive actions (`UNNotificationAction.Options.destructive`) are displayed in red.
- Text input actions (`UNTextInputNotificationAction`) allow reply without opening the app.
- Custom notification UI via `UNNotificationContentExtension` — always include a default content fallback in case the extension fails.

### Permission Request Design

- Ask for notification permission only when the user is in a context where notifications make obvious sense — not at first launch.
- Explain concretely what notifications will contain before displaying the system prompt.
- If the user declines, respect the decision. Surface an in-app setting to re-enable later rather than prompting again.
- Use `.provisional` first for discovery; escalate to full authorization only after demonstrating value.

### Notification Management (iOS 12)

- Users can now manage notifications per-app from individual notifications via long-press: Deliver Quietly, Turn Off, Manage.
- "Deliver Quietly" routes to Notification Center only — no banner, sound, or badge — without full disable.
- Notification Center on iOS 12 shows recent + missed notifications together; design groups to degrade gracefully when older items are collapsed.

## APIs & Frameworks

**UserNotifications (iOS 12 additions)**
- `UNAuthorizationOptions.provisional` **[NEW]** — request silent notification delivery without a prompt
- `UNAuthorizationOptions.criticalAlert` **[NEW]** — request permission for critical alerts (entitlement required)
- `UNMutableNotificationContent.threadIdentifier` — group related notifications into a thread stack
- `UNMutableNotificationContent.summaryArgument` **[NEW]** — the "who" in the group summary (e.g., a sender name)
- `UNMutableNotificationContent.summaryArgumentCount` **[NEW]** — numeric count represented by this notification in group summary
- `UNNotificationCategory.init(identifier:actions:intentIdentifiers:hiddenPreviewsBodyPlaceholder:categorySummaryFormat:options:)` **[NEW]** — `categorySummaryFormat` parameter controls group summary template
- `UNNotificationSound.defaultCritical` **[NEW]** — system critical alert sound
- `UNNotificationSound.criticalSoundNamed(_:withAudioVolume:)` **[NEW]** — custom critical sound with explicit volume
- `UNNotificationAction` — `title`, `options` (`.authenticationRequired`, `.destructive`, `.foreground`)
- `UNTextInputNotificationAction` — inline text reply action
- `UNNotificationAttachment` — `init(identifier:url:options:)` — attach media to notification content
- `UNNotificationContentExtension` — custom notification UI in a Notification Content extension target

## Code Highlights

Requesting provisional authorization at first launch (no prompt shown):
```swift
UNUserNotificationCenter.current().requestAuthorization(
    options: [.provisional, .alert, .sound, .badge]) { granted, error in
    // Notifications will be delivered quietly until user decides
}
```

Setting thread grouping and summary metadata:
```swift
let content = UNMutableNotificationContent()
content.title = "Sarah"
content.body = "Lunch tomorrow?"
content.threadIdentifier = "conversation-sarah-42"
content.summaryArgument = "Sarah"       // used in "X more from Sarah"
content.summaryArgumentCount = 1
```

Registering a category with a custom summary format:
```swift
let replyAction = UNTextInputNotificationAction(
    identifier: "REPLY",
    title: "Reply",
    options: [],
    textInputButtonTitle: "Send",
    textInputPlaceholder: "Type a message…")

let category = UNNotificationCategory(
    identifier: "MESSAGE",
    actions: [replyAction],
    intentIdentifiers: [],
    hiddenPreviewsBodyPlaceholder: "Message",
    categorySummaryFormat: "%u more messages from %@",
    options: [])

UNUserNotificationCenter.current().setNotificationCategories([category])
```

## Takeaways
- iOS 12's provisional authorization lets apps deliver quiet notifications without a permission prompt — use this to earn trust before requesting full authorization.
- Set `threadIdentifier` on all notifications that belong to a logical group (conversation, topic, order); design the group summary text carefully using `summaryArgument` and `categorySummaryFormat`.
- Critical alerts require a special entitlement and should be reserved for genuine health/safety scenarios — overuse destroys trust.
- The single most effective way to keep notifications enabled is to send fewer, higher-value ones; every unnecessary notification is a vote to turn them off.

---
_Source: WWDC18 Session 806 page (abstract, full transcript, and resource links)._
