# Broadcast Updates to Your Live Activities
**WWDC24 · Session 10069** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10069/)

_Platforms:_ iOS 18, iPadOS 18 (server-side: APNs; client-side: ActivityKit)

## Overview
iOS 18 introduces Broadcast Push Notifications for Live Activities—a new APNs channel type that lets a single push notification update many users' Live Activities simultaneously. This is designed for high-fan-out scenarios like sports scores, stock tickers, transit delays, or any event where a single state change needs to reach all subscribers at once, without the sender knowing individual device push tokens.

Previously, servers had to maintain per-device push tokens and send individual pushes—impractical at scale. Broadcast channels solve this: clients subscribe to a named channel, and the server pushes to the channel. APNs handles fan-out to all subscribers. This session covers the full lifecycle: channel creation, client subscription, server push, and channel management, along with best practices for channel naming, rate limits, and handling conflicts when a user's device has stale state.

## Key Topics
- **Broadcast push channels** — named APNs channels; server pushes once to a channel; APNs fans out to all subscribers
- **Channel creation** — server-side HTTP/2 request to APNs creates a channel and returns a channel ID
- **Client subscription** — `ActivityKit` `Activity.subscribe(to:)` method; client subscribes using a channel ID (not a device token)
- **Server push** — same APNs HTTP/2 format as regular push but targeting the channel ID; `apns-topic` is your app's bundle ID + `.push-type.liveactivity`
- **Channel lifecycle** — channels expire after a configurable TTL; server can delete channels early; clients unsubscribe automatically when activity ends
- **Update ordering** — APNs delivers updates in order within a channel; stale updates are discarded by the OS if a newer update has already been applied
- **Rate limits** — broadcast channels have different rate limits than device-targeted push; plan for burst scenarios (e.g., sports scoring moments)
- **Conflict resolution** — handle cases where a device missed updates; `timestamp` field in the push payload used by the OS to order updates correctly

## APIs & Frameworks
### ActivityKit (Client)
- `Activity<Attributes>` — unchanged; start/update/end remain the same
- **[NEW] `Activity.subscribe(to channelID: String)`** — subscribe the activity to a broadcast channel; call after activity creation
- **[NEW] `Activity.pushType`** — `.token` (existing device-targeted push) or **`.channel`** (new broadcast channel subscription)
- `Activity.pushTokenUpdates` — async sequence for device token updates (unchanged for token-based push)
- **[NEW] `Activity.channelID`** — the channel ID once subscribed; not a secret; safe to share

### APNs (Server-Side)
- **[NEW] Broadcast channel endpoint** — `POST /3/device/<channelID>` with new channel-targeted headers
- `apns-push-type: liveactivity` — required header (same as device-targeted Live Activity push)
- `apns-topic: <bundleID>.push-type.liveactivity` — required header
- **[NEW] Channel creation** — `POST /3/channels` with JSON body: `{ "bundle-id": "...", "expiry": 3600 }`; response contains `channel-id`
- **[NEW] Channel deletion** — `DELETE /3/channels/<channelID>`
- Payload structure — same `aps` dictionary as Live Activity device push; `content-state` encodes `ActivityAttributes.ContentState`
- `timestamp` — Unix timestamp in payload; required for update ordering on client

## Code Highlights
```swift
// Client: start a Live Activity and subscribe to broadcast channel
import ActivityKit

let attributes = SportsAttributes(matchID: "match-123")
let initialState = SportsAttributes.ContentState(homeScore: 0, awayScore: 0)

let activity = try Activity.request(
    attributes: attributes,
    content: .init(state: initialState, staleDate: nil)
)

// Subscribe to broadcast channel (channelID from your server)
let channelID = try await fetchChannelID(for: "match-123")
try await activity.subscribe(to: channelID)
```

```python
# Server: create a broadcast channel (Python example)
import httpx, json, jwt, time

# Create channel
response = httpx.post(
    "https://api.push.apple.com/3/channels",
    headers={
        "authorization": f"bearer {jwt_token}",
        "content-type": "application/json",
    },
    json={"bundle-id": "com.example.sportsapp", "expiry": 7200}
)
channel_id = response.json()["channel-id"]

# Push update to channel
payload = {
    "aps": {
        "timestamp": int(time.time()),
        "event": "update",
        "content-state": {"homeScore": 2, "awayScore": 1}
    }
}
httpx.post(
    f"https://api.push.apple.com/3/device/{channel_id}",
    headers={
        "authorization": f"bearer {jwt_token}",
        "apns-push-type": "liveactivity",
        "apns-topic": "com.example.sportsapp.push-type.liveactivity",
    },
    json=payload
)
```

## Takeaways
- Broadcast push channels eliminate the per-device token management bottleneck for high-fan-out Live Activity updates; one server push reaches all subscribers
- Always include a `timestamp` in your push payload; the OS uses it to discard stale updates when the device receives them out of order
- Broadcast channels suit scenarios with many subscribers per event (sports, elections, transit); for 1:1 updates like delivery tracking, the existing device-targeted push model is still appropriate
- Plan channel TTLs carefully—channels automatically expire, and clients must handle graceful fallback when a channel expires mid-activity

---
_Source: WWDC24 Session 10069 page (abstract, chapter summaries, code samples, and resource links)._
