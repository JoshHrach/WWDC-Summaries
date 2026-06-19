# Enhance Voice Communication with Push to Talk
**WWDC22 · Session 10117** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10117/)

_Platforms:_ iOS 16

## Overview
Presented by Kevin Ferrell and Trevor Sheridan, this session introduces the PushToTalk framework — a new iOS 16 framework for building walkie-talkie style voice communication apps. The framework provides system-level UI (a blue status-bar pill, a system popover accessible from anywhere in the system including the Lock Screen), background audio recording and playback, a dedicated APNs push type for receiving audio notifications, and tight integration with CoreBluetooth accessories.

The framework is designed to be backend-agnostic: apps implement their own audio encoding and streaming. PushToTalk manages the system UI, audio session lifecycle, Bluetooth hardware integration, and power-efficient background execution.

## Key Topics

### Channel Lifecycle
- **Channel**: represents a Push to Talk session. Only one channel can be active system-wide at a time.
- Initialize `PTChannelManager` once (shared singleton) in `applicationDidFinishLaunching`; provide a `PTChannelManagerDelegate` and a `PTChannelRestorationDelegate`.
- `requestJoinChannel(channelUUID:descriptor:)` — join a channel (foreground only). Requires a `UUID` to identify the channel and a `PTChannelDescriptor` (name + image).
- On success: `didJoinChannel` delegate callback + `receivedEphemeralPushToken` with the APNs device token for this channel (variable-length; valid only for the channel's lifetime).
- `leaveChannel(channelUUID:)` — leave programmatically, or user taps "Leave" in system UI.
- Failures: `failedToJoinChannel` (e.g., `PTChannelError.channelLimitReached`), `failedToLeaveChannel`.

### Channel Restoration
- On app relaunch or device reboot, `PTChannelRestorationDelegate.channelDescriptor(restoredChannelUUID:)` is called — return a cached `PTChannelDescriptor` quickly (no network requests).
- Update descriptors and service status any time information changes:
  - `setChannelDescriptor(_:channelUUID:)` — update name/image.
  - `setServiceStatus(_:channelUUID:)` — `.ready`, `.connecting`, `.disconnected`. System UI shows status and disables transmission when not ready.

### Transmitting Audio
- `requestBeginTransmitting(channelUUID:)` — start transmission from foreground or in response to a Bluetooth peripheral's characteristic change.
- `stopTransmitting(channelUUID:)` — stop transmission.
- Failures: `failedToBeginTransmittingInChannel` (e.g., `PTChannelError.callIsActive` during a cellular or FaceTime call), `failedToStopTransmittingInChannel`.
- Delegate callbacks:
  - `didBeginTransmitting(from:)` — source is `PTChannelTransmitRequestSource` (`.systemUI`, `.programmatic`, `.hardware`).
  - `didActivateAudioSession(_:)` — audio session activated; start recording now.
  - `didEndTransmitting(from:)` — transmission ended.
  - `didDeactivateAudioSession(_:)` — stop recording, clean up.
- Do NOT manually activate or deactivate the audio session — PushToTalk manages it. Configure `AVAudioSession.category` to `.playAndRecord` at app launch.
- The system plays built-in chimes for mic activation/deactivation; do not add custom sounds.

### Receiving Audio (PTIncomingPushDelegate)
- New APNs push type for Push to Talk: `apns-push-type: pushtotalk`, `apns-topic: <bundle-id>.voip-ptt`, `apns-priority: 10`, `apns-expiration: 0`.
- On push receipt, `incomingPushResult(channelManager:channelUUID:pushPayload:)` is called.
- Return `PTPushResult`:
  - `.activeRemoteParticipant(PTParticipant)` — set the speaking participant (name + optional image); system activates audio session, calls `didActivateAudioSession`.
  - `.leaveChannel` — server instructs the device to leave the channel.
- If image is not available immediately: return participant with name only, then call `setActiveRemoteParticipant(_:channelUUID:)` once image is downloaded.
- When remote participant finishes: `setActiveRemoteParticipant(nil, channelUUID:)` — deactivates audio session, re-enables transmission in system UI.

### System UI
- Blue pill in status bar while channel is active; tap to open system popover from anywhere (including Lock Screen).
- Popover shows: channel name, channel image, active participant name/image, Talk button (hold to transmit), Leave button.
- All UI is system-provided; apps supply data (channel name, image, participant info).

### Xcode Project Requirements
- Background mode: "Push to Talk"
- Capability: Push to Talk
- Capability: Push Notifications (for APNs background wakeup)
- `NSMicrophoneUsageDescription` in Info.plist

### Best Practices
- Initialize `PTChannelManager` in `applicationDidFinishLaunching` for fast channel restoration after relaunch.
- Handle `AVAudioSession` interruptions, route changes, and failures (cellular calls take precedence over Push to Talk).
- When app is suspended (not transmitting/receiving), network connections drop — use Network.framework with QUIC for fast reconnection.
- Use the framework's background runtime; do not keep the app alive artificially.

## APIs & Frameworks

### PushToTalk Framework **[NEW]**
- `PTChannelManager` — primary interface for channel management **[NEW]**
- `PTChannelManager.channelManager(delegate:restorationDelegate:)` — async factory (singleton) **[NEW]**
- `PTChannelManager.requestJoinChannel(channelUUID:descriptor:)` **[NEW]**
- `PTChannelManager.leaveChannel(channelUUID:)` **[NEW]**
- `PTChannelManager.requestBeginTransmitting(channelUUID:)` **[NEW]**
- `PTChannelManager.stopTransmitting(channelUUID:)` **[NEW]**
- `PTChannelManager.setChannelDescriptor(_:channelUUID:)` **[NEW]**
- `PTChannelManager.setServiceStatus(_:channelUUID:)` **[NEW]**
- `PTChannelManager.setActiveRemoteParticipant(_:channelUUID:)` **[NEW]**
- `PTChannelManagerDelegate` — delegate for all channel events **[NEW]**
- `PTChannelRestorationDelegate` — return cached descriptor on relaunch **[NEW]**
- `PTChannelDescriptor(name:image:)` — channel metadata **[NEW]**
- `PTParticipant(name:image:)` — remote participant metadata **[NEW]**
- `PTChannelTransmitRequestSource` — `.systemUI`, `.programmatic`, `.hardware` **[NEW]**
- `PTPushResult` — `.activeRemoteParticipant(_:)`, `.leaveChannel` **[NEW]**
- `PTChannelError` — `.channelLimitReached`, `.callIsActive`, `.transmissionNotFound` **[NEW]**
- `PTChannelJoinReason`, `PTChannelLeaveReason` **[NEW]**

### AVFoundation
- `AVAudioSession` — configure `.playAndRecord` category at launch
- `AVAudioSession` interruption/route-change notifications — handle in app

## Code Highlights

```swift
// Initialize channel manager at launch
func setupChannelManager() async throws {
    channelManager = try await PTChannelManager.channelManager(
        delegate: self, restorationDelegate: self)
}

// Join a channel (foreground only)
func joinChannel(channelUUID: UUID) {
    let descriptor = PTChannelDescriptor(name: "Field Team Alpha",
                                         image: UIImage(named: "ChannelIcon"))
    channelManager.requestJoinChannel(channelUUID: channelUUID, descriptor: descriptor)
}

// Receive ephemeral push token — send to server
func channelManager(_ channelManager: PTChannelManager,
                    receivedEphemeralPushToken pushToken: Data) {
    sendTokenToServer(pushToken)  // variable length — do not hardcode
}

// Transmit audio
func startTransmitting() { channelManager.requestBeginTransmitting(channelUUID: channelUUID) }
func stopTransmitting()  { channelManager.stopTransmitting(channelUUID: channelUUID) }

// Audio session lifecycle
func channelManager(_ channelManager: PTChannelManager,
                    didActivate audioSession: AVAudioSession) {
    startAudioRecording()  // begin encoding and streaming
}
func channelManager(_ channelManager: PTChannelManager,
                    didDeactivate audioSession: AVAudioSession) {
    stopAudioRecording()
}

// Receive incoming push (new apns-push-type: pushtotalk)
func incomingPushResult(channelManager: PTChannelManager,
                        channelUUID: UUID,
                        pushPayload: [String: Any]) -> PTPushResult {
    guard let speaker = pushPayload["activeSpeaker"] as? String else {
        return .leaveChannel
    }
    let participant = PTParticipant(name: speaker, image: getSpeakerImage(speaker))
    return .activeRemoteParticipant(participant)
}

// Clear active participant when audio ends
func stopReceivingAudio() {
    channelManager.setActiveRemoteParticipant(nil, channelUUID: channelUUID)
}
```

## Takeaways
- PushToTalk provides the entire system UI layer for free; apps only need to supply the channel descriptor (name + image), participant info, and implement audio streaming — the framework handles background activation, the status-bar pill, and the Lock Screen popover.
- The ephemeral APNs push token received on channel join is the only token to use; it expires when the channel ends. Always send this token to your server to enable audio delivery notifications.
- Never manually activate or deactivate `AVAudioSession` — PushToTalk manages session priority within the system; doing so yourself will break transmission and may cause battery issues.
- For reliable background reconnection, use Network.framework with QUIC; the app will be suspended when idle and network connections will drop between sessions.

---
_Source: WWDC22 Session 10117 page (transcript, code samples, and resource links)._
