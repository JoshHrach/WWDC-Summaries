# Enhance Your App's Audio Experience with AirPods
**WWDC23 · Session 10233** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10233/)

_Platforms:_ iOS 17, macOS Sonoma 14, tvOS 17

## Overview
This session covers three AirPods features introduced in iOS 17 and macOS 14: AirPods Automatic Switching for macOS, Press to Mute and Unmute via AirPods stem, and expanded Spatial Audio platform support. Together these create a more seamless, hands-free audio experience throughout the day across the Apple ecosystem.

The session introduces `AVAudioApplication`, a new API sibling to `AVAudioSession` for configuring application-wide audio behaviors. On iOS, CallKit apps get Press to Mute and Unmute for free; non-CallKit communication apps can adopt the new API with only a few lines of code. On macOS, apps additionally receive a handler to intercept the gesture and mute uplink audio themselves.

Spatial Audio support expands to macOS 14 (via `AVPlayer` and `AVSampleBufferAudioRenderer`) and continues on iOS 17 and tvOS 17 (additionally via `AURemoteIO` and `AudioQueue`), with automatic activation for apps registered for Now Playing.

## Key Topics

- **AirPods Automatic Switching for macOS** — macOS 14 joins the automatic switching ecosystem; sandbox and App Store apps participate automatically with no code changes required. Best practices: register for Now Playing for media apps, avoid registering for conferencing/gaming apps, avoid playing silence longer than two seconds, use Audio Services for notification sounds.
- **Press to Mute and Unmute** — New AirPods stem gesture that mutes/unmutes the microphone and provides a system tone and banner. CallKit apps get this free on iOS 17; non-CallKit apps use the new `AVAudioApplication` API. macOS requires additional handler and CoreAudio property adoption.
- **Spatial Audio with AirPods** — Expanded platform support for Spatial Audio and Personalized Spatial Audio; automatic activation for Now Playing-registered apps; Control Center configuration.

## APIs & Frameworks

**AVFAudio / AVFoundation**
- `AVAudioApplication` **[NEW]** — application-wide audio behavior configuration; singleton class
- `AVAudioApplication.shared` **[NEW]** — shared instance
- `AVAudioApplication.inputMuteStateChangeNotification` **[NEW]** — `Notification.Name` posted when mute state changes via gesture
- `AVAudioApplication.muteStateKey` **[NEW]** — key in notification `userInfo` for the new mute state
- `AVAudioApplication.setInputMuted(_:)` **[NEW]** — programmatically mute/unmute app's input
- `AVAudioApplication.isInputMuted` **[NEW]** — read current mute state
- `AVAudioApplication.setInputMuteStateChangeHandler(_:)` **[NEW, macOS only]** — handler called when gesture occurs; responsible for muting uplink audio; returns Bool to accept/reject
- `AVAudioSession` — existing; `AVAudioApplication` is a sibling, not a replacement
- `AVPlayer` — Spatial Audio support on macOS 14 **[NEW]**
- `AVSampleBufferAudioRenderer` — Spatial Audio support on macOS 14 **[NEW]**
- `AURemoteIO` — Spatial Audio support on iOS 17 / tvOS 17 (automatic via Now Playing)
- `AudioQueue` — Spatial Audio support on iOS 17 / tvOS 17 (automatic via Now Playing)

**CoreAudio**
- `kAudioHardwarePropertyProcessInputMute` **[NEW, macOS only]** — `AudioObjectPropertyAddress` selector to silence input audio to the process while keeping IO running; set via `AudioObjectSetPropertyData` on `kAudioObjectSystemObject`
- `AudioObjectPropertyAddress` — existing struct for CoreAudio property addressing
- `AudioObjectSetPropertyData` — existing; used to toggle process input mute

**CallKit**
- `CXCallController` / CallKit stack — existing; all CallKit apps receive Press to Mute and Unmute on iOS 17 with no additional changes

**Now Playing / MPNowPlayingInfoCenter**
- Registering for Now Playing enables automatic AirPods switching prioritization and automatic Spatial Audio activation for `AURemoteIO` / `AudioQueue` apps

## Code Highlights

Adopting Press to Mute and Unmute on iOS (non-CallKit):
```swift
import AVFAudio

let instance = AVAudioApplication.shared

NotificationCenter.default.addObserver(
    forName: AVAudioApplication.inputMuteStateChangeNotification,
    object: nil, queue: .main
) { notification in
    let isMuted = notification.userInfo?[AVAudioApplication.muteStateKey] as? Bool ?? false
    // Update internal state and UI
    instance.setInputMuted(isMuted)
}
```

Adopting Press to Mute and Unmute on macOS (additional handler):
```swift
instance.setInputMuteStateChangeHandler { isMuted in
    // Mute uplink audio here
    return true // return false to reject the gesture
}

// Optionally use CoreAudio to mute process input
var address = AudioObjectPropertyAddress(
    mSelector: kAudioHardwarePropertyProcessInputMute,
    mScope: kAudioObjectPropertyScopeInput,
    mElement: kAudioObjectPropertyElementMain)
var isMuted: UInt32 = 1
AudioObjectSetPropertyData(kAudioObjectSystemObject, &address, 0, nil,
    UInt32(MemoryLayout.size(ofValue: isMuted)), &isMuted)
```

## Takeaways

- Most App Store and sandboxed macOS apps get AirPods Automatic Switching for free in macOS 14 with no code changes.
- Non-CallKit iOS communication apps need only a few lines using `AVAudioApplication` to add Press to Mute and Unmute.
- Register for Now Playing to enable AirPods routing prioritization and automatic Spatial Audio for `AURemoteIO`/`AudioQueue` on iOS and tvOS.
- Avoid playing silence longer than two seconds after a user pauses playback to prevent unexpected AirPods switching behavior.

---
_Source: WWDC23 Session 10233 page (abstract, chapter summaries, code samples, and resource links)._
