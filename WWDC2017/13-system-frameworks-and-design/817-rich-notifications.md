# Rich Notifications
**WWDC17 · Session 817** · [Watch](https://developer.apple.com/videos/play/wwdc2017/817/)

_Platforms:_ iOS 11

## Overview
This design-focused session lays out the principles for creating notifications that users look forward to receiving rather than dismissing or disabling. The core argument is that notifications should be self-contained packages of information — not just hooks to drive app opens — and that rich notifications (Short Look, Long Look with custom content, Quick Actions) should allow users to complete a task without ever launching the app, then return seamlessly to what they were doing.

The session walks through a progression of Long Look capabilities using real app examples: displaying additional text beyond the Short Look truncation (Mail), adding a thumbnail image (Photos), using custom app UI with typography and branding (Calendar, Castro), showing video playback inline (Tips, Kuna smart camera), playing audio with a transcript (Phone/Voicemail), displaying a live-updating map (Find My Friends), and building a fully interactive experience with a live-updating view that responds to typing indicators and allows replies (Messages). Throughout, the emphasis is on bringing an app's design language into the notification rather than using generic system UI.

## Key Topics
- **Notification philosophy** — send only when genuinely relevant; not for traffic/engagement boosting; ask for permission with clear explanation of value; notifications as a self-contained task-completion surface, not an app entry point
- **Short Look** — answers "what is this notification about?"; populate all fields with clear, direct, informative language; follow current iOS 11 APIs
- **Long Look** — provides additional context; activated by 3D Touch on the notification; use custom `UNNotificationContentExtension` to show: expanded text, custom UI, images, GIFs/animations, video, audio, live-updating maps, live conversation threads
- **Quick Actions** — `UNNotificationAction` buttons at the bottom of Long Look; should allow completing the relevant task without launching the app (e.g., reply, accept invite, start playback, speak to camera intercom)
- **Content types in Long Look** — static images, custom UI matching app's design language, data visualizations (charts, weather conditions, stock prices), video (tips, highlights, camera feeds), audio with transcription, live-updating maps, live conversation threads
- **Live updates** — Long Look content can update in real time (typing indicators, location changes, score updates, stock prices, social feeds)
- **Design language** — notifications should use the same typography, layout, and visual style as the app itself (Castro podcast app example showing app-matched typesetting)
- **Branding in notifications** — app artwork, custom fonts, colors carried through from the app

## APIs & Frameworks

### UserNotifications / UserNotificationsUI
- **`UNNotificationContentExtension`** — protocol for implementing a custom Long Look view; adopted by a notification content extension target
- **`UNNotificationAction`** — defines a Quick Action button; `identifier`, `title`, `options`
- **`UNTextInputNotificationAction`** — Quick Action with a text input field (for reply functionality)
- **`UNNotificationCategory`** — groups related notification actions; registered with `UNUserNotificationCenter`
- **`UNNotificationContent`** — `title`, `subtitle`, `body`, `badge`, `sound`, `attachments`, `categoryIdentifier`, `userInfo`
- **`UNNotificationAttachment`** — wraps image, audio, or video file for display in notification; `UNNotificationAttachmentOptions` for thumbnail clipping rect
- **`UNMutableNotificationContent`** — mutable version for constructing notification content locally
- **`UNUserNotificationCenter.current()`** — singleton; `requestAuthorization(options:completionHandler:)`, `setNotificationCategories(_:)`
- **`UNNotificationServiceExtension`** — server-side push modification (e.g., download image before delivery)
- **`UNNotificationExtensionInitialContentSizeRatio`** (Info.plist key) — initial aspect ratio of Long Look custom view
- **`UNNotificationExtensionDefaultContentHidden`** (Info.plist key) — hide system-provided title/body when custom view provides its own layout
- **`UNNotificationExtensionCategory`** (Info.plist key) — maps the extension to a `UNNotificationCategory` identifier
- **`didReceive(_:)` (UNNotificationContentExtension)** — called to configure the custom view with notification content
- **`didReceive(_:completionHandler:)` (UNNotificationContentExtension)** — called when a Quick Action is tapped; allows handling the response within the extension before (optionally) launching the app

### UIKit
- **`UIViewController`** subclass as the Long Look container — full UIKit view hierarchy available: `UIImageView`, `UILabel`, `MKMapView`, `AVPlayerLayer`, etc.
- **`MKMapView`** — used for live-updating location notifications (Find My Friends example)
- **`AVPlayerLayer`** / **`AVPlayer`** — used for inline video playback in Long Look (Tips, Kuna examples)

## Code Highlights
This session is design-principles focused. Key implementation pattern for a content extension:

```swift
// Notification Content Extension principal class
class NotificationViewController: UIViewController, UNNotificationContentExtension {
    @IBOutlet var imageView: UIImageView!
    @IBOutlet var bodyLabel: UILabel!

    func didReceive(_ notification: UNNotification) {
        let content = notification.request.content
        bodyLabel.text = content.body

        // Display attachment image
        if let attachment = content.attachments.first,
           attachment.url.startAccessingSecurityScopedResource() {
            imageView.image = UIImage(contentsOfFile: attachment.url.path)
            attachment.url.stopAccessingSecurityScopedResource()
        }
    }

    // Handle quick action tap within the extension
    func didReceive(_ response: UNNotificationResponse,
                    completionHandler: @escaping (UNNotificationContentExtensionResponseOption) -> Void) {
        if response.actionIdentifier == "ACCEPT_ACTION" {
            // handle acceptance inline
            completionHandler(.dismiss)
        } else {
            completionHandler(.dismissAndForwardAction)
        }
    }
}
```

```swift
// Registering categories with actions
let acceptAction = UNNotificationAction(identifier: "ACCEPT_ACTION",
                                         title: "Accept",
                                         options: [])
let declineAction = UNNotificationAction(identifier: "DECLINE_ACTION",
                                          title: "Decline",
                                          options: [.destructive])
let inviteCategory = UNNotificationCategory(identifier: "INVITE",
                                             actions: [acceptAction, declineAction],
                                             intentIdentifiers: [],
                                             options: [])
UNUserNotificationCenter.current().setNotificationCategories([inviteCategory])
```

## Takeaways
- The Long Look is where rich notifications earn their value: use it to show enough context that users can take action without launching the app, using the same visual design language as the app itself.
- Live-updating Long Look views (maps, conversation threads, stock prices, scores) can update in real time — the user does not need to open the app to see fresh data.
- Quick Actions should complete the task that the notification represents; a user who taps "Accept" on a meeting invite should never need to open Calendar.
- Send notifications only when genuinely relevant; excessive or low-value notifications are the leading cause of users disabling all notifications from an app.

---
_Source: WWDC17 Session 817 page (abstract, chapter summaries, code samples, and resource links)._
