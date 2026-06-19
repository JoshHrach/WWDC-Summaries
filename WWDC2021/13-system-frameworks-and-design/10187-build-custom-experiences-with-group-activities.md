# Build Custom Experiences with Group Activities
**WWDC21 · Session 10187** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10187/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session goes beyond media playback to show how to build fully custom SharePlay experiences using the Group Activities framework. Using a collaborative drawing app ("DrawTogether") as the example, the session covers the entire lifecycle: defining a generic (non-media) group activity, creating a `GroupSessionMessenger` to send and receive typed Codable messages between participants, handling late joiners with catch-up data, transitioning to new sessions, and surfacing UI controls conditionally using `GroupStateObserver`.

The two key distinctions from media experiences: set `metadata.type = .generic` on the activity metadata, and replace AVFoundation playback synchronization with `GroupSessionMessenger` for custom real-time message exchange.

## Key Topics

### Defining a Custom (Generic) Group Activity
- Conform a struct to `GroupActivity` protocol
- In `metadata` computed property, set `metadata.type = .generic` — this is the only difference from a media activity configuration
- `activityIdentifier` can use the default implementation (bundle ID + type name)
- Add the Group Activities entitlement in Xcode Signing & Capabilities

### Activating the Activity
- Call `DrawTogether().activate()` — triggers the system SharePlay UI to invite participants via FaceTime
- Subscribe to the `sessions` async sequence on the activity type to receive `GroupSession<DrawTogether>` objects when the session starts
- Call `groupSession.join()` to join the session

### GroupSessionMessenger — Send and Receive Custom Messages
Three-step pattern:
1. **Define**: a `Codable` struct for each message type (e.g., `UpsertStrokeMessage` with `id`, `color`, `point`)
2. **Receive**: iterate the async sequence from `messenger.messages(of: MessageType.self)` — yields `(message, MessageContext)` tuples; `MessageContext.source` identifies the sending participant
3. **Send**: `try await messenger.send(message)` sends to all active participants; `try await messenger.send(message, to: participants)` sends to a subset

- `GroupSessionMessenger` provides **reliable, FIFO-ordered delivery** to all active participants
- Message size limits apply — do not use for large assets (images, video, files); use a different transport for those
- **Flow control**: avoid burst sending (rapid loops) — may throw errors; space out sends or batch messages
- **Versioning**: include a version field in message structs to enable interop with older app versions

### Handling Late Joiners (Catch-Up)
- `GroupSession.activeParticipants` fires when participants join
- Observe with `groupSession.$activeParticipants.sink { ... }` — compute the delta (new - old) to identify newly joined participants
- Send a catch-up message (e.g., full canvas state) **only to the new participants** using `messenger.send(canvasMessage, to: newParticipants)`
- Use a `pointCount` or timestamp heuristic in the catch-up message to accept only the most up-to-date version if multiple arrive

### Transitioning to a New Session
- **Preferred (clean break)**: call `DrawTogether().activate()` again — creates a new `GroupSession` with a clean state; all participants receive the new session via the `sessions` async sequence; old session ends
- **Lightweight update**: set `groupSession.activity = newActivity` — updates the activity for all participants without ending the session; participants observe `groupSession.$activity` for changes; system ensures convergence
- Choose "new session" for experiences needing a clean slate; choose "update activity" for playlist-style progressions

### GroupStateObserver — Conditional UI
- `GroupStateObserver` publishes `isEligibleForGroupSession: Bool` — `true` when a FaceTime call is active and SharePlay is available
- Use it to conditionally show/hide the SharePlay invitation button so it only appears when useful
- Check `groupSession != nil` to additionally hide the button when already in a session

## APIs & Frameworks

### Group Activities Framework **[NEW in iOS 15]**
- `GroupActivity` protocol **[NEW]**
  - `metadata: GroupActivityMetadata`
  - `activityIdentifier: String` (default implementation available)
  - `func activate() async throws -> GroupActivityActivationResult`
  - `static var sessions: AsyncStream<GroupSession<Self>>`
- `GroupActivityMetadata` **[NEW]**
  - `title: String?`
  - `type: GroupActivityMetadata.ActivityType` — `.generic` for custom activities, `.watchTogether` for media
- `GroupSession<Activity>` **[NEW]**
  - `func join()`
  - `func leave()`
  - `var activity: Activity` — settable to broadcast activity changes
  - `var activeParticipants: Set<Participant>` (published)
  - `var state: GroupSession<Activity>.State` (published)
- `GroupSessionMessenger` **[NEW]**
  - `init(session: GroupSession<Activity>)`
  - `func messages<T: Decodable>(of type: T.Type) -> AsyncStream<(T, MessageContext)>`
  - `func send<T: Encodable>(_ value: T) async throws`
  - `func send<T: Encodable>(_ value: T, to participants: Set<Participant>) async throws`
  - `MessageContext.source: Participant` — who sent the message
- `GroupStateObserver` **[NEW]**
  - `var isEligibleForGroupSession: Bool` (published)

## Code Highlights

Defining a generic custom Group Activity:
```swift
struct DrawTogether: GroupActivity {
    var metadata: GroupActivityMetadata {
        var metadata = GroupActivityMetadata()
        metadata.title = NSLocalizedString("Draw Together", comment: "Title of group activity")
        metadata.type = .generic  // Required for custom (non-media) activities
        return metadata
    }
}
```

Activating and receiving sessions:
```swift
// Activate (trigger SharePlay invite UI)
Button { Task { try? await DrawTogether().activate() } } label: {
    Image(systemName: "person.2")
}

// Receive incoming sessions
Task {
    for await session in DrawTogether.sessions {
        canvas.configureGroupSession(session)
    }
}
```

Defining, receiving, and sending custom messages:
```swift
// 1. Define
struct UpsertStrokeMessage: Codable {
    let id: UUID
    let color: Color
    let point: CGPoint
}

// 2. Receive
let messenger = GroupSessionMessenger(session: groupSession)
Task.detached {
    for await (message, context) in messenger.messages(of: UpsertStrokeMessage.self) {
        await canvas.handle(message, from: context.source)
    }
}

// 3. Send
do {
    try await messenger.send(UpsertStrokeMessage(id: stroke.id, color: stroke.color, point: point))
} catch { print("Send error: \(error)") }
```

Late-joiner catch-up:
```swift
var currentParticipants: Set<Participant> = []
groupSession.$activeParticipants.sink { newParticipants in
    let newJoiners = newParticipants.subtracting(currentParticipants)
    currentParticipants = newParticipants
    if !newJoiners.isEmpty {
        let catchUp = CanvasMessage(strokes: canvas.strokes, pointCount: canvas.totalPoints)
        Task { try? await messenger.send(catchUp, to: newJoiners) }
    }
}
```

Conditional SharePlay button using GroupStateObserver:
```swift
@StateObject var groupStateObserver = GroupStateObserver()

var body: some View {
    if groupStateObserver.isEligibleForGroupSession && groupSession == nil {
        Button { Task { try? await DrawTogether().activate() } } label: {
            Image(systemName: "person.2.circle")
        }
    }
}
```

## Takeaways
- Setting `metadata.type = .generic` is the only configuration change needed to turn a media Group Activity into a custom one — all other session lifecycle APIs are identical.
- `GroupSessionMessenger` with a Codable message type provides FIFO-reliable delivery with minimal code — the three-step pattern (define → receive → send) covers the full bidirectional messaging lifecycle.
- Late joiner handling is an app responsibility: observe `$activeParticipants`, compute the delta, and send catch-up state only to the new participants.
- `GroupStateObserver` eliminates guesswork about when to show SharePlay controls — use it to keep invitation UI visible only when a FaceTime call is actually active.

---
_Source: WWDC21 Session 10187 page (abstract, transcript, and code samples)._
