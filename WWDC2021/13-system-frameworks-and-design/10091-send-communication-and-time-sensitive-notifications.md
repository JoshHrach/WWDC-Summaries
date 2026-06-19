# Send Communication and Time Sensitive Notifications
**WWDC21 · Session 10091** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10091/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8

## Overview
iOS 15 introduces a major evolution of the notifications system, giving developers fine-grained control over how and when notifications interrupt users. Two new system-level features — Notification Summary and Focus — filter and schedule notifications, and this session explains how to work with them through four new interruption levels.

The session also introduces Communication Notifications, a richer notification type backed by SiriKit `INSendMessageIntent` and `INStartCallIntent`. Communication notifications get prominent avatars, standardized layouts, Siri Announce support, and the ability to break through Focus/Summary based on user-defined contact priorities — making apps first-class citizens in the Siri suggestions and Focus configuration experience.

## Key Topics

### Visual Updates
- Larger app icon in notification banners; rich notification expanded view updated to match system style.
- **Notification action icons** **[NEW]**: attach `UNNotificationActionIcon` to `UNNotificationAction` / `UNTextInputNotificationAction` using system image names or template image names.

### Notification Management: Summary and Focus
- **Notification Summary**: delivers notifications at user-scheduled times instead of immediately; include media attachments and set a `relevanceScore` on `UNMutableNotificationContent` to increase prominence in the summary.
- **Focus**: user-configured modes (Work, Sleep, Reading, etc.) that filter which apps and people can interrupt. Communication notifications and Time Sensitive notifications can break through if allowed.

### Interruption Levels (NEW)
Four levels in `UNNotificationInterruptionLevel`:
- **Passive** (`.passive`): delivered silently to the notification list; no sound, no screen wake, no breakthrough.
- **Active** (`.active`): default behavior — sound, vibration, screen wake; does not break through Focus/Summary.
- **Time Sensitive** (`.timeSensitive`): same as active but can break through Focus and Notification Summary if the user allows it. Requires the **Time Sensitive Notifications** capability in Xcode.
- **Critical** (`.critical`): existing Critical Alerts level; bypasses the ringer switch and always interrupts. Requires an approved entitlement.

Set via `UNMutableNotificationContent.interruptionLevel` for local notifications, or the `"interruption-level"` key in APNs push payload.

### Siri Announce
- In iOS 15, Siri can announce any notification on supported devices (AirPods Pro/Max, HomePod, CarPlay) — the category-level `allowAnnouncement` option from iOS 14 is deprecated.
- Communication and Time Sensitive notifications are announced by default.

### Communication Notifications (NEW)
- Associate `INSendMessageIntent` or `INStartCallIntent` with a notification via `UNNotificationContent.updating(from:)` (implementing `UNNotificationContentProviding`).
- Must be done in a `UNNotificationServiceExtension` for remote notifications (intents are device-local), or in the main app for local notifications.
- Results in: prominent avatar display, standardized title (sender) / subtitle (recipients), Siri Announce, Siri contact suggestions, Focus breakthrough eligibility.
- Requires the **Communication Notifications** capability in Xcode.
- Intent persons (`INPerson`) should be populated as fully as possible: `personHandle`, `nameComponents`, `displayName`, `image`, `contactIdentifier`, `customIdentifier`, `suggestionType`.

### Intent Donation
- Donate `INInteraction` with `.incoming` direction in the notification service extension when receiving messages/calls.
- Donate `INInteraction` with `.outgoing` direction from the main app when the user sends a message or places a call (feeds Siri suggestions and Focus priority).
- Only outgoing donations affect Contacts suggestions (avoids spam callers cluttering Contacts).

## APIs & Frameworks

- `UserNotifications` framework
- `UNNotificationActionIcon` **[NEW]** — icon for a notification action
- `UNNotificationActionIcon(systemImageName:)` **[NEW]**
- `UNNotificationActionIcon(templateImageName:)` **[NEW]**
- `UNNotificationAction(identifier:title:options:icon:)` **[NEW]**
- `UNTextInputNotificationAction(identifier:title:options:icon:textInputButtonTitle:textInputPlaceholder:)` **[NEW]**
- `UNNotificationInterruptionLevel` enum **[NEW]** — `.passive`, `.active`, `.timeSensitive`, `.critical`
- `UNMutableNotificationContent.interruptionLevel` **[NEW]**
- `UNMutableNotificationContent.relevanceScore` **[NEW]** — 0.0–1.0 float for Notification Summary prominence
- APNs `"interruption-level"` key **[NEW]** — `"passive"`, `"active"`, `"time-sensitive"`, `"critical"`
- `UNNotificationContentProviding` protocol **[NEW]** — marker protocol on `INSendMessageIntent` / `INStartCallIntent`
- `UNNotificationContent.updating(from:)` **[NEW]** — returns content upgraded with communication intent
- `UNNotificationServiceExtension` — required for remote communication notifications
- `SiriKit` / `Intents` framework
- `INSendMessageIntent` — represents a messaging event
- `INStartCallIntent` — represents a call event
- `INInteraction` — wraps an intent for donation
- `INInteraction.direction` — `.incoming` or `.outgoing`
- `INInteraction.donate(completion:)` — donates the interaction to Siri
- `INPerson` — represents a participant; fully populate for best Siri experience
- `INPersonHandle` — identifies a person (email, phone, etc.)
- Time Sensitive Notifications capability (Xcode) — required for `.timeSensitive` level
- Communication Notifications capability (Xcode) — required for communication notifications

## Code Highlights

Setting up notification action icons:
```swift
let likeIcon = UNNotificationActionIcon(systemImageName: "hand.thumbsup")
let likeAction = UNNotificationAction(identifier: "like-action", title: "Like",
                                      options: [], icon: likeIcon)
```

Time Sensitive local notification:
```swift
let content = UNMutableNotificationContent()
content.title = "Urgent"
content.body = "Your account requires attention."
content.interruptionLevel = .timeSensitive
```

Creating a communication notification in a service extension:
```swift
func didReceive(_ request: UNNotificationRequest,
                withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
    let incomingMessageIntent: INSendMessageIntent = // ...
    let interaction = INInteraction(intent: incomingMessageIntent, response: nil)
    interaction.direction = .incoming
    interaction.donate(completion: nil)
    let messageContent = try! request.content.updating(from: incomingMessageIntent)
    contentHandler(messageContent)
}
```

## Takeaways
- Adopt interruption levels to respect Notification Summary and Focus rather than always interrupting; only use `.timeSensitive` for genuinely urgent notifications.
- Use `UNNotificationActionIcon` to add SF Symbols or custom template icons to notification actions for visual clarity.
- Implement communication notifications via `INSendMessageIntent` / `INStartCallIntent` in a `UNNotificationServiceExtension` to get prominent avatars, Siri Announce, and Focus breakthrough.
- Donate both incoming and outgoing intents: incoming in the service extension, outgoing from the main app when the user acts, to build Siri's understanding of important contacts.

---
_Source: WWDC21 Session 10091 page (abstract, chapter summaries, code samples, and resource links)._
