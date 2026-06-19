# Streaming Audio on watchOS 6
**WWDC19 · Session 716** · [Watch](https://developer.apple.com/videos/play/wwdc2019/716/)

_Platforms:_ watchOS 6, iOS 13

## Overview
watchOS 6 introduces audio streaming directly on Apple Watch — without requiring the paired iPhone nearby. This session explains the two supported streaming models (HLS via AVQueuePlayer, and custom protocols via URLSession/Network framework), the required project configuration, and the specific sequencing rules for audio session activation and network access that differ significantly from iOS.

The session also brings parity between the watchOS and iOS SDKs by adding AVFoundation, Core Media, URLSession streaming/WebSocket tasks, and the Network framework to watchOS 6.

## Key Topics

### Two Streaming Models
- **HLS Streaming:** Point `AVQueuePlayer` at an HLS audio feed. AVFoundation handles buffering, adaptive bitrate selection, and playback automatically. Simplest path for HLS content.
- **Custom/Proprietary Protocols:** Use `URLSession` (streaming task or WebSocket task) or the Network framework to fetch and decode audio data, then hand off PCM/compressed audio to AVFoundation for playback.

### Project Configuration
- Requires Apple Watch Series 3 or later.
- In Xcode, add the Background Modes capability to the WatchKit Extension target and enable the Audio mode.
- If the app already supports background audio playback, no additional project configuration is needed for streaming.

### Audio Session Activation Sequence (Critical Difference from iOS)
1. On app launch, use `URLSession` to fetch metadata (playlists, album art, stream options) from servers. At this stage, `URLSessionStreamingTask`, `URLSessionWebSocketTask`, and Network framework are NOT yet available.
2. Activate `AVAudioSession` only when the user is ready to begin streaming — not before. Activating the session early disrupts user experience.
3. On activation, watchOS automatically presents an audio route picker if no route is currently active (Bluetooth headphones, etc.). On iOS, a default audio route is always available; on watchOS it is not.
4. After audio session activation, all networking APIs become available: `URLSessionStreamingTask`, `URLSessionWebSocketTask`, Network framework.
5. Once audio data is buffered, start playback via AVFoundation.

### Route Selection
- If a Bluetooth device is already connected to the Watch, watchOS skips the route picker.
- For Apple W/H-chip Bluetooth devices connected to the paired iPhone, watchOS may temporarily borrow the connection (not guaranteed — e.g., active phone call prevents it).

### Networking Best Practices
- Always check `WKInterfaceDevice.supportsAudioStreaming` before streaming.
- Cache sufficient audio data locally at all times to handle network condition changes.
- Minimize the number and size of network requests — unnecessary requests can stall playback on Watch hardware.
- Start streaming at 64 kbps; monitor data arrival rate and upgrade bitrate only when network conditions permit. AVFoundation does this automatically for HLS.
- Do not rely on network reachability APIs — always assume connectivity and handle stalls and failures gracefully in real time.
- Apple Watch frequently transitions between Bluetooth, Wi-Fi, and cellular. Each transition can take several seconds; streaming code must tolerate these gaps.

### Deprecated APIs
- `WKAudioFilePlayer` and related `WKAudioFile*` APIs are deprecated in watchOS 6. Migrate to AVFoundation APIs.

## APIs & Frameworks

### AVFoundation (now on watchOS) **[NEW]**
- `AVPlayer` — audio playback **[NEW on watchOS]**
- `AVQueuePlayer` — queue-based playback, HLS streaming **[NEW on watchOS]**
- `AVAudioSession` — audio session management
- `AVAudioSession.activate(options:completionHandler:)` — activates session, triggers route picker if needed
- `AVAudioSession.Category.playback` — long-form audio category

### Core Media (now on watchOS) **[NEW]**
- `CMTime`, `CMTimeRange`, and related time types **[NEW on watchOS]**

### URLSession (expanded on watchOS) **[NEW/UPDATED]**
- `URLSessionStreamingTask` — available after audio session activated **[NEW on watchOS]**
- `URLSessionWebSocketTask` — WebSocket support **[NEW in iOS 13 and watchOS 6]**
- `.avStreaming` — new `URLRequest.networkServiceType` for streaming audio data **[NEW]**

### Network Framework (now on watchOS) **[NEW]**
- `NWConnection` — modern socket alternative, Swift-native **[NEW on watchOS]**
- `NWListener`, `NWPathMonitor` — network monitoring
- First introduced in iOS 12; now available on watchOS 6

### WatchKit
- `WKInterfaceDevice.supportsAudioStreaming` — capability check for streaming **[NEW]**
- `WKAudioFilePlayer`, `WKAudioFileQueuePlayer` — deprecated in watchOS 6

## Code Highlights

Checking streaming support:
```swift
guard WKInterfaceDevice.current().supportsAudioStreaming else {
    // Fall back to synced audio
    return
}
```

Using AVQueuePlayer for HLS streaming:
```swift
let item = AVPlayerItem(url: URL(string: "https://example.com/stream.m3u8")!)
let player = AVQueuePlayer(playerItem: item)
player.play()
```

Setting the AV streaming network service type:
```swift
var request = URLRequest(url: audioChunkURL)
request.networkServiceType = .avStreaming
let task = URLSession.shared.dataTask(with: request) { data, _, _ in
    // Buffer and play audio data
}
task.resume()
```

Activating audio session (triggers route picker):
```swift
do {
    try AVAudioSession.sharedInstance().setCategory(.playback, mode: .default)
    AVAudioSession.sharedInstance().activate(options: []) { success, error in
        if success {
            // Now safe to use URLSessionStreamingTask, Network framework
        }
    }
} catch { }
```

## Takeaways
- Activate `AVAudioSession` only immediately before streaming begins — watchOS presents a mandatory route picker at that moment, unlike iOS where a default route always exists.
- Use `URLSessionStreamingTask` and `URLSessionWebSocketTask` only after the audio session is active; earlier calls will fail.
- Start at 64 kbps and adapt upward based on measured throughput — Watch networks are variable and can transition without warning between Bluetooth, Wi-Fi, and cellular.
- `WKAudioFilePlayer` and related APIs are deprecated; migrate entirely to AVFoundation + URLSession.

---
_Source: WWDC19 Session 716 page (abstract, chapter summaries, code samples, and resource links)._
