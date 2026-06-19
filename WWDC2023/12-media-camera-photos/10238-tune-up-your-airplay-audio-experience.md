# Tune up your AirPlay audio experience
**WWDC23 · Session 10238** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10238/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17

## Overview
This session introduces AirPlay Enhanced Audio Buffering — a new underlying AirPlay protocol designed to make whole-home audio more robust and responsive. Enhanced audio buffering streams data faster than real-time playback so temporary Wi-Fi interruptions (e.g., walking out of range) no longer cause audible glitches. It is also newly supported in wireless CarPlay. The session covers the full stack an app must implement for a great AirPlay audio experience: `AVAudioSession` configuration, `AVInitialRouteSharingPolicy`, `AVRoutePickerView`, `MPNowPlayingInfoCenter`/`MPRemoteCommandCenter` integration, and — critically — adopting either `AVQueuePlayer` or the lower-level `AVSampleBufferAudioRenderer` + `AVSampleBufferRenderSynchronizer` pair, which automatically enables enhanced buffering when routing to AirPlay.

A separate new feature, intelligent AirPlay suggestions, uses on-device learning to proactively surface nearby AirPlay speakers in Control Center when the user opens a longform-audio app — enabled with a single Info.plist key.

## Key Topics

### AirPlay Enhanced Audio Buffering
The new protocol buffers audio faster than real-time so the AirPlay receiver (HomePod, smart TV, CarPlay head unit) has a large reserve of audio data:
- **Robust** — playback continues through Wi-Fi dead spots without skipping
- **Responsive** — tap on HomePod or iPhone remote controls take effect immediately
- **Multi-channel** — supports Dolby Atmos from Apple TV
- **Lossless** (**[NEW]**) — intelligent use of lossless audio on iOS 17

Enhanced buffering also provides the best HLS Interstitials support for ad insertion (see WWDC23 session "Explore AirPlay with Interstitials").

Wireless CarPlay now also benefits from enhanced buffering — apps that adopt the required APIs automatically get improved CarPlay audio at no extra cost.

### AVAudioSession Configuration
Apps must configure `AVAudioSession` correctly for AirPlay:
1. **Category:** `.playback` — continues audio when app goes to background
2. **Mode:** `.default` for music; `.spokenAudio` for podcasts/audiobooks
3. **Routing policy:** `.longFormAudio` — required for enhanced audio buffering

```swift
try AVAudioSession.sharedInstance().setCategory(.playback, mode: .default, policy: .longFormAudio)
```

### Intelligent AirPlay Suggestions (New in iOS 17)
On-device machine learning learns the user's AirPlay habits (e.g., plays music on the kitchen HomePod when cooking) and proactively surfaces those speakers in Control Center when the app is opened. To opt in:
1. Apply the `AVAudioSession` `.longFormAudio` policy (above)
2. Add `AVInitialRouteSharingPolicy` key to Info.plist with value `LongFormAudio` **[NEW]**
   - Xcode label: "AirPlay optimization policy"

### AVRoutePickerView
Add `AVRoutePickerView` to the view hierarchy to show a standard AirPlay device picker. No custom implementation required.

### Media Player Integration
Use `MPNowPlayingInfoCenter.default()` to provide now-playing metadata (title, artist, artwork, duration, elapsed time) and `MPRemoteCommandCenter.shared()` to receive remote commands (play, pause, next/previous track, seek).

### Player API Choice: Two Paths to Enhanced Buffering

**Path 1 — AVQueuePlayer (recommended for most apps)**
- Automatically enables enhanced audio buffering when routed to AirPlay
- Handles item management, playback control, seeking
- Simplest adoption

**Path 2 — AVSampleBufferAudioRenderer + AVSampleBufferRenderSynchronizer**
- For apps with custom DRM, preprocessing on media data, or formats `AVPlayer` cannot handle
- Manual: app is responsible for enqueueing `CMSampleBuffer` objects on demand
- `AVSampleBufferRenderSynchronizer` establishes the media timeline; `AVSampleBufferAudioRenderer` follows it
- Install `requestMediaDataWhenReady(on:using:)` callback; enqueue until `isReadyForMoreMediaData` is false; call `stopRequestingMediaData()` at end of stream

For non-AirPlay routing (local speaker, Bluetooth), both APIs also work correctly. Apps wanting different behavior per route can observe `AVAudioSession.routeChangeNotification`.

## APIs & Frameworks

- **AVFoundation**
  - `AVAudioSession`
    - `.setCategory(_:mode:policy:)` — configure playback category, mode, and routing policy
    - `AVAudioSession.Category.playback` — keep audio alive in background
    - `AVAudioSession.Mode.default` / `.spokenAudio` — general vs. podcast/audiobook mode
    - `AVAudioSession.RouteSharingPolicy.longFormAudio` — required for enhanced AirPlay buffering
    - `AVAudioSession.routeChangeNotification` — observe to detect AirPlay vs. local route changes
  - `AVQueuePlayer` — **recommended** player for enhanced audio buffering **[now supports enhanced AirPlay buffering]**
    - `insert(_:after:)` — enqueue `AVPlayerItem`
    - `play()` / `pause()` / `seek(to:)` — standard playback controls
  - `AVPlayerItem` — wraps `AVAsset` for playback
  - `AVAsset(url:)` — asset from local or remote URL
  - `AVSampleBufferAudioRenderer` — low-level audio renderer for custom player pipelines **[supports enhanced AirPlay buffering]**
    - `requestMediaDataWhenReady(on:using:)` — callback for filling the buffer
    - `enqueue(_:)` — enqueue a `CMSampleBuffer`
    - `isReadyForMoreMediaData: Bool` — check before enqueuing
    - `stopRequestingMediaData()` — call at end of stream
  - `AVSampleBufferRenderSynchronizer` — establishes media timeline for sample buffer rendering
    - `addRenderer(_:)` — attach audio renderer
    - `rate: Float` — set to `1.0` to start playback at natural speed
  - `AVRoutePickerView` — standard AirPlay device picker UI control
  - `AVInitialRouteSharingPolicy` Info.plist key **[NEW]** — value `LongFormAudio` enables intelligent AirPlay suggestions
- **MediaPlayer**
  - `MPNowPlayingInfoCenter.default()` — provide now-playing metadata dictionary
    - Keys: `MPMediaItemPropertyTitle`, `MPMediaItemPropertyArtist`, `MPNowPlayingInfoPropertyElapsedPlaybackTime`, etc.
  - `MPRemoteCommandCenter.shared()` — receive remote control commands
    - `.playCommand`, `.pauseCommand`, `.nextTrackCommand`, `.previousTrackCommand`, `.changePlaybackPositionCommand`

## Code Highlights

Full audio session setup:
```swift
let session = AVAudioSession.sharedInstance()
try session.setCategory(.playback, mode: .default, policy: .longFormAudio)
try session.setActive(true)
```

AVQueuePlayer — simplest path to enhanced AirPlay buffering:
```swift
let player = AVQueuePlayer()
let url = URL(string: "https://example.com/audio.m4a")!
let asset = AVAsset(url: url)
let item = AVPlayerItem(asset: asset)
player.insert(item, after: nil)
player.play()
// Enhanced audio buffering is automatic when AirPlay is the output route
```

AVSampleBufferAudioRenderer — custom player pipeline:
```swift
let serializationQueue = DispatchQueue(label: "audio.serialization")
let audioRenderer = AVSampleBufferAudioRenderer()
let renderSynchronizer = AVSampleBufferRenderSynchronizer()
renderSynchronizer.addRenderer(audioRenderer)

serializationQueue.async {
    self.audioRenderer.requestMediaDataWhenReady(on: serializationQueue) {
        while self.audioRenderer.isReadyForMoreMediaData {
            if let buffer = self.nextSampleBuffer() {
                self.audioRenderer.enqueue(buffer)
            } else {
                self.audioRenderer.stopRequestingMediaData()
                break
            }
        }
    }
    self.renderSynchronizer.rate = 1.0
}
```

Info.plist key for intelligent AirPlay suggestions:
```xml
<key>AVInitialRouteSharingPolicy</key>
<string>LongFormAudio</string>
```

## Takeaways

- Adopt `AVQueuePlayer` (or `AVSampleBufferAudioRenderer` + `AVSampleBufferRenderSynchronizer` for custom pipelines) to automatically enable AirPlay Enhanced Audio Buffering — no other AirPlay-specific code is needed in the player itself.
- Set `AVAudioSession` category to `.playback`, mode to `.default` (or `.spokenAudio` for podcasts/audiobooks), and routing policy to `.longFormAudio` — these three settings are prerequisites for enhanced AirPlay buffering.
- Add the `AVInitialRouteSharingPolicy = LongFormAudio` Info.plist key to opt into iOS 17's intelligent AirPlay suggestions, which surface nearby speakers proactively based on the user's listening habits.
- Enhanced audio buffering also works automatically for wireless CarPlay when either player API is adopted — no additional code is required for the CarPlay path.

---
_Source: WWDC23 Session 10238 page (abstract, chapters, transcript, and code samples)._
