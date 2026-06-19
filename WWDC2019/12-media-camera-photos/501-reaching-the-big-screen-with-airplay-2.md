# Reaching the Big Screen with AirPlay 2
**WWDC19 · Session 501** · [Watch](https://developer.apple.com/videos/play/wwdc2019/501/)

_Platforms:_ iOS 13, macOS Catalina 10.15, tvOS 13

## Overview
AirPlay 2 in iOS 13 brings major improvements for video apps, including support for built-in AirPlay on third-party smart TVs (up to 4K HDR with surround sound), intelligent AirPlay Suggestions powered by on-device learning, and enhanced multitasking so playback continues while users do other things on their device.

This session walks through five key integration areas: delivering high-quality HLS video suitable for TV playback, adding an in-app AirPlay route picker, implementing full remote control and Now Playing metadata support, enabling background media for multitasking, and integrating with the new long-form video routing APIs for AirPlay Suggestions.

## Key Topics

### High-Quality HLS Video for AirPlay
- Playlists must include variants for 4K, HDR, and surround sound — not just phone/tablet resolutions.
- Include variants for every video codec (e.g., H.264 and HEVC) to enable smooth codec-matching transitions.
- Add an I-frame variant (`EXT-X-I-FRAMES-ONLY`) to support fast-forward frame previews during seeking.
- FairPlay Streaming (FPS) is fully supported on Apple TV and all AirPlay-enabled smart TVs.
- For ad insertion or serial content, use HLS discontinuity tags or `AVQueuePlayer`. Never destroy and recreate an `AVPlayer` instance while AirPlaying — the system may tear down the AirPlay engine.

### In-App AirPlay Route Picker
- Use `AVRoutePickerView` (available since iOS 11, now on macOS) to present the AirPlay destination picker.
- **[NEW in iOS 13]** Set `prioritizesVideoDevices = true` to show the AirPlay video icon and sort TV destinations to the top.
- `MPVolumeView`'s route button properties are deprecated in iOS 13. Migrate to `AVRoutePickerView`.

### Remote Controls and Now Playing Metadata
- Become the Now Playing app by: (1) activating an `AVAudioSession` with a non-mixable category (e.g., `.playback`), and (2) registering at least one `MPRemoteCommandCenter` command handler.
- On macOS, set `MPNowPlayingInfoCenter.default().playbackState` explicitly.
- Update `MPNowPlayingInfoCenter.default().nowPlayingInfo` dictionary with:
  - Static metadata: `MPMediaItemPropertyTitle`, `MPMediaItemPropertyArtwork`, etc.
  - Playback state: `MPNowPlayingInfoPropertyElapsedPlaybackTime`, `MPMediaItemPropertyPlaybackDuration`, `MPNowPlayingInfoPropertyPlaybackRate`.
  - The system infers progress between explicit updates — only update on rate changes or seeks.
- **[NEW in iOS 13]** Remote control UIs support audio language and subtitle selection via `MPRemoteCommandCenter.enableLanguageOptionCommand` and `.disableLanguageOptionCommand`.
- Temporarily disable commands with `command.isEnabled = false` (no need to remove the handler).
- Use `AVAsset`'s media selection groups and map them to `MPNowPlayingInfoLanguageOptionGroup` for language/subtitle support.

### Background Media (Multitasking)
- Add `audio` background mode to `Info.plist`.
- Activate `AVAudioSession` with `.playback` category only when the user initiates playback on primary content.
- Listen to `AVAudioSession.interruptionNotification` — only auto-resume if `shouldResume` key is `true`.
- Set `AVPlayer.allowsExternalPlayback = false` for promotional/launch videos that should never AirPlay.
- Never auto-play primary content on app launch or foreground — always require explicit user action.

### Long-Form Video Integration and AirPlay Suggestions **[NEW in iOS 13]**
- Long-form video apps (movies, TV shows, sports) qualify for AirPlay Suggestions and advanced routing behaviors.
- Register by adding `AVInitialRouteSharingPolicy` = `longFormVideo` to `Info.plist`.
- Call `AVAudioSession.sharedInstance().prepareRouteSelectionForPlayback(completionHandler:)` when the user taps Play. The system may auto-route or prompt the user.
- Completion handler receives a `AVAudioSessionRouteSelection`: `.local` (play on device), `.external` (play via AirPlay), or `.none` (user cancelled).
- **[NEW]** Long-form video apps can play short content locally on the device while long-form content continues on the TV — without interruption.
- Test with the Developer Settings > AirPlay Suggestions toggle (Default / Always Prompt / Always Automatic).

## APIs & Frameworks

### AVFoundation / AVKit
- `AVRoutePickerView` — in-app AirPlay destination picker **[now on macOS]**
- `AVRoutePickerView.prioritizesVideoDevices` — sort TV destinations to top **[NEW]**
- `AVAudioSession` — audio session configuration
- `AVAudioSession.Category.playback` — non-mixable background-capable category
- `AVAudioSession.prepareRouteSelectionForPlayback(completionHandler:)` — intelligent route selection **[NEW]**
- `AVAudioSessionRouteSelection` — `.local`, `.external`, `.none` **[NEW]**
- `AVInitialRouteSharingPolicy` — Info.plist key for long-form video registration **[NEW]**
- `AVPlayer.allowsExternalPlayback` — prevent specific content from AirPlaying
- `AVQueuePlayer` — preferred for serial content when AirPlaying
- `AVPlayerViewController` — handles remote controls automatically
- `AVAudioSession.interruptionNotification` — media interruption notifications

### MediaPlayer Framework
- `MPRemoteCommandCenter` — registers handlers for remote control commands
  - `.playCommand`, `.pauseCommand`, `.togglePlayPauseCommand`
  - `.skipForwardCommand`, `.skipBackwardCommand`
  - `.changePlaybackPositionCommand`
  - `.enableLanguageOptionCommand` **[NEW]**
  - `.disableLanguageOptionCommand` **[NEW]**
  - `MPSkipIntervalCommand.preferredIntervals`
  - `MPRemoteCommand.isEnabled`
- `MPNowPlayingInfoCenter` — provides Now Playing metadata to system UI
  - `MPMediaItemPropertyTitle`, `MPMediaItemPropertyArtwork`
  - `MPMediaItemPropertyPlaybackDuration`
  - `MPNowPlayingInfoPropertyElapsedPlaybackTime`
  - `MPNowPlayingInfoPropertyPlaybackRate`
- `MPNowPlayingInfoLanguageOptionGroup` — audio/subtitle language options **[NEW]**
- `MPNowPlayingInfoCenter.default().playbackState` — macOS playback state
- `MPVolumeView` — volume UI (route button deprecated in iOS 13)

### HTTP Live Streaming
- `EXT-X-I-FRAMES-ONLY` — I-frame variant for seek previews
- HLS discontinuity tags — for ad insertion / serial content stitching
- FairPlay Streaming (FPS) — DRM, supported on all AirPlay destinations

## Code Highlights

Adding a video-prioritized route picker:
```swift
let picker = AVRoutePickerView()
picker.prioritizesVideoDevices = true
view.addSubview(picker)
```

Registering remote control commands:
```swift
let center = MPRemoteCommandCenter.shared()
center.playCommand.addTarget { _ in
    player.play(); return .success
}
center.skipForwardCommand.preferredIntervals = [15]
center.skipForwardCommand.addTarget { event in
    let e = event as! MPSkipIntervalCommandEvent
    player.seek(to: player.currentTime() + CMTime(seconds: e.interval, preferredTimescale: 1))
    return .success
}
```

Updating Now Playing playback info:
```swift
var info = MPNowPlayingInfoCenter.default().nowPlayingInfo ?? [:]
info[MPMediaItemPropertyPlaybackDuration] = duration
info[MPNowPlayingInfoPropertyElapsedPlaybackTime] = currentTime
info[MPNowPlayingInfoPropertyPlaybackRate] = player.rate
MPNowPlayingInfoCenter.default().nowPlayingInfo = info
```

Long-form video routing:
```swift
// Info.plist: AVInitialRouteSharingPolicy = longFormVideo
AVAudioSession.sharedInstance().prepareRouteSelectionForPlayback { routeSelection, _ in
    switch routeSelection {
    case .local: presentPlayerUI()
    case .external: presentRemotePlaybackUI()
    case .none: break
    }
}
```

## Takeaways
- Ensure HLS playlists include 4K/HDR/surround variants and I-frame tracks — users can AirPlay to TVs at any time.
- Migrate from `MPVolumeView` route buttons to `AVRoutePickerView` with `prioritizesVideoDevices = true`.
- Implement `MPRemoteCommandCenter` and `MPNowPlayingInfoCenter` for full remote control support including language/subtitle switching (new in iOS 13).
- Register long-form video apps via `AVInitialRouteSharingPolicy` and use `prepareRouteSelectionForPlayback` to enable intelligent AirPlay Suggestions.

---
_Source: WWDC19 Session 501 page (abstract, chapter summaries, code samples, and resource links)._
