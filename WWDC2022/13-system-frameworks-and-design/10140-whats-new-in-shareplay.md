# What's New in SharePlay
**WWDC22 · Session 10140** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10140/)

_Platforms:_ iOS 15.4+, iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
SharePlay (the Group Activities framework) ships three new capabilities in iOS 15.4 and iOS 16. First, apps can now start a SharePlay session directly — without a pre-existing FaceTime call — using a new `GroupActivitySharingController` API and share sheet integration. Second, `GroupSessionMessenger` doubles down on real-time messaging with a 4× larger payload limit (256 KB) and a new unreliable (UDP-based) messaging mode for low-latency communication. Third, the session covers best practices around "Staged GroupActivity," ownerless session design, and FaceTime handoff support introduced in iOS 16.

## Key Topics

### Starting SharePlay from Your App (iOS 15.4+)
Previously, SharePlay required an active FaceTime call before an app could begin a session. Now apps can initiate SharePlay in three ways:

1. **Share sheet (zero-adoption)** — If the app is entitled for SharePlay, a SharePlay button automatically appears in the system share sheet. Register the `GroupActivity` on `NSItemProvider` and pass it to `UIActivityViewController` for the optimal experience.
2. **Prominent vs. non-prominent** — `UIActivityViewController.allowsProminentActivity = false` shows the SharePlay button less prominently; set `excludedActivityTypes = [.sharePlay]` to suppress it entirely.
3. **In-app button** — Create a `GroupActivitySharingController` and present it as a `UIViewController`. The user picks contacts via the system people picker, starts a FaceTime call, and can then activate the staged `GroupActivity`.

Upon activation of the staged GroupActivity, the app receives a `GroupSession` as normal.

### Staged GroupActivity and Ownerless Sessions
A "Staged GroupActivity" is a group activity that has been proposed but not yet activated. When the session activates, each participant's device may have different knowledge of the current state (e.g., playback position). Best practice: each device that joins should contribute its state knowledge to all others so they can converge on a source of truth. Sessions are peer-to-peer and explicitly ownerless — avoid any "owner" logic because:
- iOS 16 introduces **FaceTime handoff**: a user can transfer the session from iPhone to Apple TV or iPad; the originating device leaves the session
- System UI (the FaceTime HUD's "End SharePlay for Everyone" button) can call `.end()` on the `GroupSession` on your app's behalf at any time

Design the experience so any participant leaving or rejoining does not break the session.

### GroupSessionMessenger Improvements
**Payload size increase** — Maximum message payload increased from 64 KB to **256 KB** (4×). Apps no longer need to fragment large messages manually.

**Unreliable messaging (UDP) [NEW]** — A new `MessageReliability` parameter on `GroupSessionMessenger` allows choosing between:
- `.reliable` (default) — guaranteed delivery, TCP-like; message arrives eventually but may be delayed
- `.unreliable` — UDP-based; low latency, no delivery guarantee; ideal for high-frequency real-time data

Use `.unreliable` for position/gesture updates during an active interaction (where a missed frame is acceptable) and `.reliable` for finalized state (e.g., the completed stroke with all points).

## APIs & Frameworks

**Group Activities (GroupActivities framework)**
- `GroupActivitySharingController` **[NEW]** — UIViewController that presents system people picker to start a FaceTime + SharePlay session from within the app
- `NSItemProvider.registerGroupActivity(_:)` **[NEW]** — register a `GroupActivity` for share sheet participation
- `UIActivityViewController.allowsProminentActivity` **[NEW]** — control SharePlay button prominence in share sheet
- `UIActivityViewController.excludedActivityTypes` — use `.sharePlay` to exclude SharePlay from share sheet
- `GroupSessionMessenger(session:deliveryMode:)` **[NEW]** — initializer accepting `MessageDeliveryMode`
- `MessageDeliveryMode.reliable` — guaranteed delivery (existing behavior)
- `MessageDeliveryMode.unreliable` **[NEW]** — UDP-based low-latency delivery
- `GroupSessionMessenger` max payload — increased from 64 KB to **256 KB** **[UPDATED]**
- `GroupSession.end()` — end session; may be called by system on app's behalf
- `GroupActivity` — existing protocol for defining shareable activities

## Code Highlights

```swift
// Start SharePlay from the share sheet
let itemProvider = NSItemProvider()
itemProvider.registerGroupActivity(WatchTogether())
let configuration = UIActivityItemsConfiguration(itemProviders: [itemProvider])
let shareSheet = UIActivityViewController(activityItemsConfiguration: configuration)
// Optionally suppress prominence or exclude SharePlay:
// shareSheet.allowsProminentActivity = false
// shareSheet.excludedActivityTypes = [.sharePlay]
present(shareSheet, animated: true)

// Start SharePlay with an in-app button (no pre-existing FaceTime call needed)
let controller = GroupActivitySharingController(WatchTogetherActivity())
present(controller, animated: true)
```

```swift
// Unreliable messenger for real-time drawing gestures
let unreliableMessenger = GroupSessionMessenger(session: session, deliveryMode: .unreliable)
let reliableMessenger   = GroupSessionMessenger(session: session)

// Send each drag point via unreliable channel (low latency, ok to drop)
try await unreliableMessenger.send(DrawingPoint(location: location))

// Send complete stroke via reliable channel when gesture ends
try await reliableMessenger.send(CompletedStroke(points: allPoints))
```

## Takeaways

- Add a SharePlay entry point directly in your app using `GroupActivitySharingController` — users no longer need to start a FaceTime call first; this significantly lowers the barrier to starting group sessions.
- Design your session as ownerless: every participant contributes its state on join and any participant (including system UI) can end the session — never gate behavior on "who started it."
- Use `.unreliable` delivery for high-frequency real-time data (cursor positions, drawing gestures, game state) and `.reliable` for finalized or critical state — this combination reduces latency while preserving data integrity.
- The 256 KB payload limit eliminates most manual message fragmentation; still design message payloads efficiently for sessions with many participants.

---
_Source: WWDC22 Session 10140 page (abstract, chapter summaries, code samples, and resource links)._
