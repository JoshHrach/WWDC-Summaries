# Create Live Communication Experiences
**WWDC26 · Session 226** · [Watch](https://developer.apple.com/videos/play/wwdc2026/226/)

_Platforms:_ iOS, iPadOS

## Overview
LiveCommunicationKit is a new framework introduced at WWDC26 that gives VoIP and real-time communication apps a native, system-integrated call UI. It renders full-screen on the Lock Screen, appears in Control Center, surfaces in the Dynamic Island, and integrates with multitasking — all without apps implementing any custom call chrome.

The framework is action-based: the system delivers `ConversationAction` objects to the app's `ConversationManagerDelegate`, and the app fulfills or fails them after performing the associated media operations (connect, end, merge). This unified model handles both system-initiated actions (user taps "Answer" on Lock Screen) and app-initiated actions (user taps a call button in-app) through the same delegate path.

The session walks through all three lifecycle scenarios — incoming conversations via PushKit, outgoing conversations, and group calls with merge/unmerge.

## Key Topics

### Introduction and Core Concepts
A `Conversation` is built from `Handle` objects (phone number or email address), display names, and a `Capabilities` set. The `ConversationManager` singleton manages the full lifecycle. All configuration happens in `ConversationManager.Configuration` — ringtone, icon, group limits, supported handle types, video support.

### Incoming Conversations
Incoming calls are delivered via PushKit (`PKPushRegistry`). In the `PKPushRegistryDelegate` method, call `manager.reportNewIncomingConversation(uuid:update:)` with a `Conversation.Update` that includes members and capabilities. The system displays the incoming call UI. When the user answers, the delegate receives a `JoinConversationAction`; the app sets up its media stream, reports `.conversationStartedConnecting` and `.conversationConnected`, then calls `action.fulfill(dateConnected:)`.

### Outgoing Conversations
Create a `StartConversationAction` with a UUID, handles, and `isVideo`. Call `manager.perform([startAction])`. The delegate receives the action and routes it through the same `handleStartAction` path, maintaining a single unified action-handling switch.

### Groups
Pass multiple handles to `StartConversationAction` for group calls. Use `Conversation.Update` with `localMember`, `members` (all invited), and `activeRemoteMembers` (currently connected). Report updates via `manager.reportConversationEvent(.conversationUpdated(update), for: conversation)`. Merge/unmerge is handled via `MergeConversationAction` (with `conversationUUID` and `conversationUUIDToMergeWith`).

## APIs & Frameworks

### LiveCommunicationKit (new framework — **[NEW]**)
- `ConversationManager` — **[NEW]** central manager class
  - `init(configuration:)`
  - `delegate: ConversationManagerDelegate`
  - `conversations` — current active conversations
  - `reportNewIncomingConversation(uuid:update:)` — async throws
  - `reportConversationEvent(_:for:)` — send lifecycle events
  - `perform(_:)` — async throws; execute an array of actions
- `ConversationManager.Configuration` — **[NEW]** struct
  - `ringtoneName`, `iconTemplateImageData`, `maximumConversationGroups`, `maximumConversationsPerConversationGroup`, `includesConversationInRecents`, `supportsVideo`, `supportedHandleTypes`
- `ConversationManagerDelegate` — **[NEW]** protocol
  - `conversationManager(_:perform:)` — primary action routing method
- `Conversation` — **[NEW]** value type; `uuid`, `members`, `capabilities`
- `Conversation.Update` — **[NEW]** value type for reporting state changes
  - `init(members:capabilities:)` — for incoming
  - `init(localMember:members:activeRemoteMembers:capabilities:)` — for group updates
- `ConversationAction` — **[NEW]** base class for all actions
  - `fulfill()` / `fail()`
- `JoinConversationAction: ConversationAction` — **[NEW]**
  - `conversationUUID`
  - `fulfill(dateConnected:)`
- `EndConversationAction: ConversationAction` — **[NEW]**
- `StartConversationAction: ConversationAction` — **[NEW]**
  - `init(conversationUUID:handles:isVideo:)`
- `MergeConversationAction: ConversationAction` — **[NEW]**
  - `conversationUUID` — source conversation
  - `conversationUUIDToMergeWith` — target conversation
- `ConversationEvent` — **[NEW]** enum: `.conversationStartedConnecting(_:)`, `.conversationConnected(_:)`, `.conversationUpdated(_:)`
- `Handle` — **[NEW]** struct: `type` (`HandleType.phoneNumber`, `.emailAddress`), `value`, `displayName`
- `Capabilities` — **[NEW]** option set: `.video`, `.pausing`, `.merging`, `.unmerging`

### PushKit
- `PKPushRegistry` / `PKPushRegistryDelegate` — used to receive incoming VoIP push payloads
- `PKPushPayload`, `PKVoIPPushMetadata`
- `pushRegistry(_:didReceiveIncomingVoIPPushWith:metadata:)` async — entry point for incoming call reporting

### Resources
- [LiveCommunicationKit documentation](https://developer.apple.com/documentation/LiveCommunicationKit)
- [Initiating VoIP conversations with LiveCommunicationKit](https://developer.apple.com/documentation/LiveCommunicationKit/initiating-voip-conversations-with-livecommunicationkit)
- [Responding to VoIP Notifications from PushKit](https://developer.apple.com/documentation/PushKit/responding-to-voip-notifications-from-pushkit)

## Code Highlights

Set up the manager:
```swift
let configuration = ConversationManager.Configuration(
    ringtoneName: "SampleRingtone.caf",
    iconTemplateImageData: UIImage(named: "SampleIcon")?.pngData(),
    maximumConversationGroups: 1,
    maximumConversationsPerConversationGroup: 2,
    includesConversationInRecents: true,
    supportsVideo: true,
    supportedHandleTypes: [.phoneNumber, .emailAddress]
)
let manager = ConversationManager(configuration: configuration)
manager.delegate = self
```

Report incoming conversation from PushKit:
```swift
let capabilities = [.video, .pausing, .merging]
let update = Conversation.Update(members: [handle], capabilities: capabilities)
try? await manager.reportNewIncomingConversation(uuid: uuid, update: update)
```

Fulfill join action:
```swift
manager.reportConversationEvent(.conversationStartedConnecting(.now), for: conversation)
try await setupMediaStream(with: action.conversationUUID)
manager.reportConversationEvent(.conversationConnected(.now), for: conversation)
action.fulfill(dateConnected: .now)
```

Start an outgoing call:
```swift
let startAction = StartConversationAction(
    conversationUUID: UUID(),
    handles: [Handle(type: .phoneNumber, value: "+1-650-555-0199", displayName: "Ryan Notch")],
    isVideo: false
)
try await manager.perform([startAction])
```

## Takeaways
- LiveCommunicationKit replaces CallKit for new VoIP apps and provides a richer, more integrated system UI including Dynamic Island and Lock Screen — without apps building any call chrome.
- All actions (incoming answer, outgoing start, end, merge) flow through a single delegate method; the action object carries context and the app calls `fulfill()` or `fail()` after completing media setup.
- Group call state is tracked through `Conversation.Update` with separate `members` (invited) and `activeRemoteMembers` (connected) arrays — keep both in sync via `reportConversationEvent(.conversationUpdated)`.
- Merge/unmerge for conference calls is handled by `MergeConversationAction` with source and target conversation UUIDs; report an updated `Conversation.Update` to the target after combining streams.

---
_Source: WWDC26 Session 226 page (abstract, chapter summaries, code samples, and resource links)._
