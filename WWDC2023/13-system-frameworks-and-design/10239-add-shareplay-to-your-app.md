# Add SharePlay to your app
**WWDC23 · Session 10239** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10239/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17

## Overview
This session provides a comprehensive overview of SharePlay and the GroupActivities framework, highlighting new capabilities introduced in iOS 17 and beyond. SharePlay turns any app activity into a shared group experience — whether participants are far apart on a FaceTime call, chatting over Messages, or physically next to each other using AirDrop.

New in iOS 17, SharePlay can now be initiated directly via AirDrop by simply bringing two devices close together, expanding the three group contexts (FaceTime, Messages, and now AirDrop). The session also covers new activity type metadata, the redesigned Share menu in FaceTime calls, and practical guidance for adopting GroupActivities in your app.

The talk emphasizes that SharePlay handles all the infrastructure concerns — group management UI, privacy-preserving end-to-end encrypted data channels, and participant state — leaving developers free to build the activity itself.

## Key Topics

### New to SharePlay
- **SharePlay over AirDrop** — In iOS 17, bringing two devices together starts a SharePlay session instantly without accounts or usernames. Sessions can migrate from AirDrop to Messages to FaceTime as the group moves apart.
- **tvOS 17 FaceTime** — FaceTime calls can now be initiated on Apple TV, and SharePlay from tvOS is just an app away.
- **New activity types** — Added `workout`, `shop`, `read`, `learn`, and `create` activity types alongside the existing `watchTogether`, `listenTogether`, `playTogether`, and `generic`.
- **Redesigned Share menu in FaceTime** — New card UI in iOS 17 surfaces apps that support SharePlay and collaboration features directly from active FaceTime calls.

### Use Cases
Education (interactive synced diagrams), meal prep coordination, social feeds, shopping together (furniture, rideshare), food ordering (group restaurant orders), and more.

### Add SharePlay to Your App
1. Import `GroupActivities` and create a type conforming to `GroupActivity`.
2. Set a unique `activityIdentifier`.
3. Provide `GroupActivityMetadata` with a title, optional subtitle, preview image, and activity type.
4. Use `GroupSessionMessenger` for data sync; adopt `NSItemProvider` to support starting sessions from the share sheet and AirDrop.

### Best Practices
- Design for the group experience as if there were no devices — preserve the feeling of togetherness in UI.
- Support SharePlay across all platforms your app targets (iOS, iPadOS, macOS, tvOS).
- Add a dedicated SharePlay button in your app's UI to improve discoverability.
- Adopt `NSItemProvider` so AirDrop can start sessions.
- Make metadata titles activity-specific (not just the app name).
- Test with two or more physical devices.

## APIs & Frameworks

### GroupActivities
- `GroupActivities` (framework) **[NEW features]**
- `GroupActivity` protocol — conform to define a shareable activity
- `GroupActivity.activityIdentifier` — unique string identifier for the activity type
- `GroupActivityMetadata` — struct holding user-facing activity metadata
  - `metadata.title` — display name of the activity
  - `metadata.subtitle` — additional context shown in UI **[NEW]**
  - `metadata.previewImage` — `CGImage` shown in the activity card
  - `metadata.type` — `GroupActivityMetadata.ActivityType`
- `GroupActivityMetadata.ActivityType`
  - `.generic`
  - `.watchTogether`
  - `.listenTogether`
  - `.playTogether` (Game Center)
  - `.workout` **[NEW]**
  - `.shopTogether` **[NEW]**
  - `.read` **[NEW]**
  - `.learn` **[NEW]**
  - `.create` **[NEW]**
- `GroupSession` — manages the lifecycle of a SharePlay session
- `GroupSessionMessenger` — low-latency, end-to-end encrypted data channel for syncing app state
- `NSItemProvider` — adopt to enable starting sessions from share sheet and AirDrop **[NEW for AirDrop]**

### Related
- AirDrop SharePlay initiation (proximity-based, iOS 17) **[NEW]**
- FaceTime SharePlay on tvOS 17 **[NEW]**
- Redesigned FaceTime Share menu card (iOS 17) **[NEW]**

## Code Highlights

Defining a `GroupActivity` with metadata:

```swift
import GroupActivities

struct OrderTogether: GroupActivity {
    static let activityIdentifier = "com.example.apple-samplecode.TacoTruck.OrderTogether"

    let orderUUID: UUID
    let truckName: String

    var metadata: GroupActivityMetadata {
        var metadata = GroupActivityMetadata()
        metadata.title = "Order Tacos Together"
        metadata.subtitle = truckName
        metadata.previewImage = UIImage(named: "ActivityImage")?.cgImage
        metadata.type = .shopTogether
        return metadata
    }
}
```

## Takeaways
- iOS 17 adds SharePlay via AirDrop, enabling instant group sessions when devices are physically close — no accounts needed.
- Five new activity type values (`.workout`, `.shopTogether`, `.read`, `.learn`, `.create`) let the system surface the right icon and context.
- Adopting `NSItemProvider` is now essential to support AirDrop-initiated SharePlay sessions.
- SharePlay handles group UI, privacy (E2E encryption), and participant management — focus your effort on the shared activity itself.

---
_Source: WWDC23 Session 10239 page (abstract, chapter summaries, code samples, and resource links)._
