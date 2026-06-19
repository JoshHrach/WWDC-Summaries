# Update Live Activities with Push Notifications
**WWDC23 · Session 10185** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10185/)

_Platforms:_ iOS 17, iPadOS 17

## Overview
This session explains how to remotely update Live Activities using Apple Push Notification service (APNs) and the ActivityKit framework. Rather than relying on the app running in the foreground, a server can push real-time content state changes directly to an active Live Activity, keeping the Lock Screen and Dynamic Island up to date without impacting battery life.

The session walks through every required step: enabling push notifications in the app target, requesting a Live Activity with `pushType: .token`, observing `pushTokenUpdates` to reliably receive and forward push tokens to a server, and constructing the correct APNs HTTP/2 request (headers + JSON payload). It also details how to test push updates locally from the terminal using curl before wiring up a real server.

The latter half covers production concerns: choosing between low-priority (opportunistic, unlimited) and high-priority (immediate, budgeted) updates, the new `NSSupportsLiveActivitiesFrequentUpdates` Info.plist key for high-frequency use cases, adding alert objects with localized titles and custom sounds, and payload enhancements such as `stale-date`, `dismissal-date`, and `relevance-score`.

## Key Topics

### Preparations
- Add the Push Notifications capability to the app target in Xcode.
- Pass `pushType: .token` to `Activity.request(attributes:content:pushType:)` so ActivityKit requests a push token from APNs.
- Observe `activity.pushTokenUpdates` (an `AsyncSequence`) to receive the initial token and any future token rotations.
- Convert the token `Data` to a hex string and send it to your server. Also handle subsequent token updates — the system may rotate the token during the activity's lifetime.

### First Push Update
- APNs push type: `liveactivity` (new push type introduced alongside ActivityKit push support).
- Required APNs headers: `apns-push-type: liveactivity`, `apns-topic: <bundleID>.push-type.liveactivity`, `apns-priority: 5 or 10`.
- Payload fields: `timestamp` (seconds since 1970), `event` (`"update"` or `"end"`), `content-state` (JSON-encoded ContentState).
- Encode ContentState with a default `JSONEncoder` (no custom strategies) to match the default `JSONDecoder` used on device.
- Test locally: use `curl` with an authentication token to send pushes directly to APNs sandbox without a server.

### Priority and Alerts
- **Low priority (`apns-priority: 5`)**: opportunistic delivery, no budget limit; use for the majority of updates.
- **High priority (`apns-priority: 10`)**: immediate delivery, subject to system budget; use for time-sensitive events.
- `NSSupportsLiveActivitiesFrequentUpdates` (Info.plist key, boolean) **[NEW]**: raises the high-priority budget for intensive Live Activities.
- `ActivityAuthorizationInfo.frequentPushesEnabled` **[NEW]**: property to check whether users have allowed frequent updates; send to server before starting updates.
- Alert object in payload: `title`, `body`, `sound` fields; title and body support `loc-key` / `loc-args` for on-device localization.
- Custom sounds: add audio files as app bundle resources; reference by filename in the `sound` field.

### Enhancements
- **`event: "end"`**: ends the Live Activity; combine with `dismissal-date` (optional, seconds since 1970) to control Lock Screen removal timing.
- **`stale-date`**: hints to the system when to mark the activity stale; read `ActivityViewContext.isStale` in SwiftUI views to show a stale-state UI.
- **`relevance-score`**: numeric value controlling ordering of multiple concurrent Live Activities on the Lock Screen and in the Dynamic Island (higher = more prominent).

## APIs & Frameworks

- `ActivityKit` — framework for managing Live Activities
- `Activity<Attributes>` — generic type representing a Live Activity instance
- `Activity.request(attributes:content:pushType:)` **[NEW `pushType` param]** — starts a Live Activity with optional push token support
- `Activity.pushTokenUpdates` **[NEW]** — `AsyncSequence` of `Data` values for push token changes
- `Activity.pushToken` — synchronous property (may be nil; prefer `pushTokenUpdates`)
- `ActivityAttributes` — protocol defining static and dynamic Live Activity data
- `ActivityContent` — wrapper around dynamic content state and stale date
- `ActivityViewContext` — context passed to SwiftUI Live Activity views
- `ActivityViewContext.isStale` **[NEW]** — `Bool` indicating the activity content may be outdated
- `ActivityAuthorizationInfo.frequentPushesEnabled` **[NEW]** — whether the user allows frequent high-priority updates
- `NSSupportsLiveActivitiesFrequentUpdates` **[NEW]** — Info.plist key (Boolean) to opt into higher update budget
- `Activity.request(attributes:content:pushType:)` `pushType` parameter — `.token` value **[NEW]**
- APNs push type `liveactivity` **[NEW]** — new APNs push type for Live Activity updates
- APNs header `apns-topic` with `.push-type.liveactivity` suffix **[NEW]**
- APNs payload field `timestamp` — Unix timestamp for ordering updates
- APNs payload field `event` — `"update"` or `"end"`
- APNs payload field `content-state` — JSON-encoded ContentState
- APNs payload field `alert` — object with `title`, `body`, `sound`; `title`/`body` support `loc-key`/`loc-args`
- APNs payload field `stale-date` **[NEW]** — Unix timestamp after which content is considered stale
- APNs payload field `dismissal-date` **[NEW]** — Unix timestamp for automatic Lock Screen removal after `end` event
- APNs payload field `relevance-score` **[NEW]** — ordering hint for concurrent Live Activities
- `JSONEncoder` / `JSONDecoder` (Foundation) — used to encode/decode ContentState; no custom strategies
- `WidgetKit` — used for `ActivityConfiguration` and rendering the Live Activity UI
- `ActivityConfiguration` — WidgetKit type defining the Live Activity widget configuration

## Code Highlights

```swift
// Request a Live Activity with push token support
let activity = try Activity.request(
    attributes: adventure,
    content: .init(state: initialState, staleDate: nil),
    pushType: .token
)

// Observe push token updates and forward to server
Task {
    for await pushToken in activity.pushTokenUpdates {
        let tokenString = pushToken.reduce("") { $0 + String(format: "%02x", $1) }
        try await self.sendPushToken(hero: hero, pushTokenString: tokenString)
    }
}
```

```json
// APNs payload for an update with alert and stale date
{
  "aps": {
    "timestamp": 1685952000,
    "event": "update",
    "stale-date": 1685959200,
    "relevance-score": 100,
    "content-state": {
      "currentHealthLevel": 0.0,
      "eventDescription": "Power Panda has been knocked down!"
    },
    "alert": {
      "title": { "loc-key": "%@ is knocked down!", "loc-args": ["Power Panda"] },
      "body":  { "loc-key": "Use a potion to heal %@!", "loc-args": ["Power Panda"] },
      "sound": "HeroDown.mp4"
    }
  }
}
```

## Takeaways
- Push tokens for Live Activities are per-activity and must be observed via `pushTokenUpdates` (not read synchronously), then forwarded to the server; token rotation must also be handled.
- Use low-priority pushes for the majority of updates to avoid hitting the high-priority budget; enable `NSSupportsLiveActivitiesFrequentUpdates` only when the use case demands sustained high-frequency updates.
- `stale-date` and `ActivityViewContext.isStale` give a clean way to warn users when the Live Activity content may be out of date due to missed pushes.
- `dismissal-date` and `relevance-score` in the end-event payload provide fine-grained control over Lock Screen cleanup and multi-activity ordering.

---
_Source: WWDC23 Session 10185 page (abstract, chapter summaries, code samples, and resource links)._
