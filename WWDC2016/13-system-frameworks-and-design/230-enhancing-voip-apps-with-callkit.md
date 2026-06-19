# Enhancing VoIP Apps with CallKit
**WWDC16 · Session 230** · [Watch](https://developer.apple.com/videos/play/wwdc2016/230/)

_Platforms:_ iOS 10

## Overview
CallKit is a brand-new framework introduced in iOS 10 that elevates third-party VoIP apps to a first-party calling experience. Before CallKit, VoIP calls arrived as local notifications — easily missed banners that required the user to unlock the device and navigate to the app. With CallKit, incoming calls show the same full-screen native UI used by the Phone app, with a custom ringtone and slide-to-answer. Outgoing calls can be initiated from the Phone app's Recents, Favorites, and Contacts tabs, from Siri, and from CarPlay and Bluetooth accessories.

The framework introduces two primary classes: `CXProvider`, which the app uses to tell the system about external events (incoming calls, call state changes); and `CXCallController`, which the app uses to request system approval for user-initiated actions (start call, end call, hold, etc.). This separation ensures that the system can properly interleave VoIP calls with native phone calls, FaceTime, and other VoIP apps.

## Key Topics

### Architecture
- `CXProvider` — app notifies system of out-of-band events (incoming call, remote party connected/ended).
- `CXCallController` — app requests user-initiated actions; system approves and may interleave with other active calls (e.g., holding a phone call to let the VoIP call begin).
- `CXCallUpdate` — metadata object used to describe a call (handle, display name, capabilities).
- `CXTransaction` — wraps one or more `CXAction` instances; submitted via `CXCallController`.
- `CXAction` subclasses — `CXAnswerCallAction`, `CXEndCallAction`, `CXStartCallAction`, `CXHoldCallAction`, `CXSetGroupCallAction`, `CXPlayDTMFCallAction`, `CXSetMutedCallAction`; each must be fulfilled or failed.

### Incoming Call Flow
1. PushKit delivers push notification to the app.
2. App creates a `CXCallUpdate` with call metadata and calls `provider.reportNewIncomingCall(with:update:completion:)`.
3. System shows full-screen native incoming call UI with custom ringtone.
4. If user answers, `CXProvider` delegate receives `provider(_:perform:)` with a `CXAnswerCallAction`.
5. App configures (but does not activate) its `AVAudioSession`, starts call audio pipeline, calls `action.fulfill()`.
6. System activates the audio session at boosted priority and calls `providerDidActivate(_:audioSession:)`.
7. App begins processing call audio in the `providerDidActivate` callback.

### Outgoing Call Flow
1. App receives an `INStartAudioCallIntent` (via `NSUserActivity`) from Phone app Recents/Contacts, Siri, CarPlay, or Bluetooth.
2. App creates a `CXStartCallAction` from the intent handle.
3. App wraps action in `CXTransaction` and calls `callController.request(_:completion:)`.
4. System approves, holds any conflicting calls, then sends action back via `CXProvider` delegate.
5. App performs the call, calls `action.fulfill()`, then reports state changes to provider (`reportOutgoingCall(with:startedConnectingAt:)`, `reportOutgoingCall(with:connectedAt:)`).
6. Active call shows the green double-height status bar (previously reserved for Phone/FaceTime).

### Provider Authorization
- `CXProvider` requires user authorization (similar to Contacts/CoreLocation).
- Check `CXProvider.authorizationStatus` at launch; request authorization if undetermined.
- Include an informative `NSVoIPUsageDescription` in `Info.plist`.
- Observe authorization status changes while running.

### Provider Configuration (`CXProviderConfiguration`)
- `localizedName` — display name shown in native call UI.
- `supportsVideo` — whether the app supports video calls.
- `maximumCallGroups` / `maximumCallsPerCallGroup` — call concurrency limits.
- `iconTemplateImageData` — masked icon image shown in the end-call UI button that deep-links to the app (available in a future seed at time of the session).
- `includesCallsInRecents` — whether to show calls in the Phone app's Recents.
- `ringtoneSound` — custom ringtone name.

### Action Errors and Timeouts
- Each action must be fulfilled (`action.fulfill()`) or failed (`action.fail()`) within a system-defined timeout.
- Failing an action allows the system to show appropriate error UI to the user.
- If an action times out, the provider delegate receives `provider(_:timedOutPerforming:)`.

### System Restrictions on Incoming Calls
- Incoming calls may be filtered by the system for several reasons; the `reportNewIncomingCall` completion handler receives an error with a `CXErrorCode`:
  - `.filteredByDoNotDisturb` — user has Do Not Disturb enabled.
  - `.filteredByBlockList` — caller is in the user's blocked list.
  - `.notAuthorized` / `.unknownError` — app not authorized or disabled.

### Audio Integration
- App should configure (not activate) its `AVAudioSession` when receiving or starting a call.
- System activates the audio session at boosted priority after `action.fulfill()`.
- `CXProviderDelegate.provider(_:didActivate:)` — start call audio processing here.
- `CXProviderDelegate.provider(_:didDeactivate:)` — stop call audio processing here.
- Audio is on par with native Phone and FaceTime; other apps cannot interrupt it.
- CallKit handles audio routing for Bluetooth accessories and user accessibility settings automatically.

## APIs & Frameworks

- **CallKit** **[NEW framework in iOS 10]**
- `CXProvider` **[NEW]** — reports out-of-band call events to system
- `CXProviderConfiguration` **[NEW]** — customizes call UI appearance and capabilities
- `CXCallController` **[NEW]** — requests user-initiated actions from system
- `CXCallUpdate` **[NEW]** — metadata for a call (handle, display name, hasVideo, etc.)
- `CXHandle` **[NEW]** — typed call handle (phone number, email, generic)
- `CXTransaction` **[NEW]** — container for one or more actions
- `CXAction` **[NEW]** — base action class; subclasses:
  - `CXStartCallAction` **[NEW]**
  - `CXAnswerCallAction` **[NEW]**
  - `CXEndCallAction` **[NEW]**
  - `CXHoldCallAction` **[NEW]**
  - `CXSetGroupCallAction` **[NEW]**
  - `CXPlayDTMFCallAction` **[NEW]**
  - `CXSetMutedCallAction` **[NEW]**
- `CXProviderDelegate` **[NEW]** — protocol for receiving actions and system events
  - `provider(_:perform:)` — overloaded for each `CXAction` subclass
  - `provider(_:didActivate:audioSession:)` — audio session activated by system
  - `provider(_:didDeactivate:audioSession:)` — audio session deactivated
  - `provider(_:timedOutPerforming:)` — action timed out
- `CXProvider.reportNewIncomingCall(with:update:completion:)` **[NEW]**
- `CXProvider.reportOutgoingCall(with:startedConnectingAt:)` **[NEW]**
- `CXProvider.reportOutgoingCall(with:connectedAt:)` **[NEW]**
- `CXProvider.reportCall(with:endedAt:reason:)` **[NEW]**
- `CXCallController.request(_:completion:)` **[NEW]**
- `CXProvider.authorizationStatus` **[NEW]**
- `CXProvider.requestAuthorization()` **[NEW]**
- `CXErrorCode` **[NEW]** — `.filteredByDoNotDisturb`, `.filteredByBlockList`, `.notAuthorized`
- **PushKit** (`PKPushRegistry`) — used to receive VoIP push notifications that trigger incoming call reports
- **SiriKit** (`INStartAudioCallIntent`) — provides handle for outgoing calls initiated from Siri or Phone app
- `AVAudioSession` — configured (not activated) by app; activated/deactivated by system via CallKit

## Code Highlights

Reporting an incoming call:
```swift
func reportIncomingCall(uuid: UUID, handle: String) {
    let update = CXCallUpdate()
    update.remoteHandle = CXHandle(type: .phoneNumber, value: handle)
    provider.reportNewIncomingCall(with: uuid, update: update) { error in
        guard error == nil else { return }
        let call = SpeakerboxCall(uuid: uuid)
        callManager.addCall(call)
    }
}
```

Answering a call in the provider delegate:
```swift
func provider(_ provider: CXProvider, perform action: CXAnswerCallAction) {
    guard let call = callManager.call(with: action.callUUID) else {
        action.fail(); return
    }
    configureAudioSession()   // configure only, system activates
    call.answer()
    action.fulfill()
}
```

Requesting an outgoing call:
```swift
let handle = CXHandle(type: .phoneNumber, value: phoneNumber)
let startCallAction = CXStartCallAction(call: uuid, handle: handle)
let transaction = CXTransaction(action: startCallAction)
callController.request(transaction) { error in ... }
```

## Takeaways
- CallKit gives third-party VoIP apps full-screen incoming call UI, Recents/Favorites/Contacts integration, Siri, CarPlay, and Bluetooth accessory support — all previously exclusive to the native Phone app.
- Never activate your `AVAudioSession` directly; configure it in the action callback and let the system activate it at boosted priority. Start audio processing only in `provider(_:didActivate:audioSession:)`.
- Always fulfill or fail every action within the system timeout; failure provides user-facing error feedback and prevents the system from entering inconsistent state.
- Use `CXCallController` even for calls started from within your own app UI; the system needs to coordinate with other active calls (phone, FaceTime, other VoIP apps) before granting your call audio priority.

---
_Source: WWDC16 Session 230 page (abstract, transcript, and resource links)._
