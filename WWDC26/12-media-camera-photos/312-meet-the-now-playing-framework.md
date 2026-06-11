# Meet the Now Playing Framework
**WWDC26 · Session 312** · [Watch](https://developer.apple.com/videos/play/wwdc2026/312/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, watchOS, visionOS

## Overview
Now Playing is a new Swift framework introduced at WWDC26 that replaces the older `MPNowPlayingInfoCenter` / `MPRemoteCommandCenter` pattern with a modern, observable API. It connects an app's media playback directly to system surfaces — Lock Screen, Control Center, Dynamic Island, and CarPlay — without manual dictionary management.

The framework centers on two protocols: `MediaSessionRepresentable` for in-app playback, and `RemoteMediaSessionRepresentable` (inside an app extension) for representing playback on external devices such as smart speakers. A `MediaSession` object wraps a conforming model and drives all system UI automatically when the model's observable state changes.

A third feature, Media Sharing Extensions, lets apps route audio to third-party output devices via the system device picker without bundling external SDKs.

## Key Topics

### Media Sessions
Adopt `MediaSessionRepresentable` on an `@Observable` model class. The protocol requires an `id` string, a `content` property returning `any MediaContentRepresentable`, a `playbackSnapshot` returning a `MediaPlaybackSnapshot`, and a `commands` array of `MediaCommand` values. Instantiate a `MediaSession<YourModel>(model)` and the system takes care of the rest.

`GenericContent` is the concrete `MediaContentRepresentable` for audio and video with title, subtitle, type, duration, and an `Artwork` closure. `MediaPlaybackSnapshot` wraps the current playback state (`.playing()`, `.paused`). `MediaCommand` provides static factory methods for `.play`, `.pause`, `.previous`, `.next`, and custom commands.

### Remote Media Sessions
For playback on external devices, implement a `RemoteMediaSessionExtension` app extension (using `ExtensionFoundation`) and adopt `RemoteMediaSessionRepresentable` on the model inside it. The extension point identifier is `AppExtensionPoint.Identifier(host: "com.apple.nowplaying", name: "remote-media")`.

Remote sessions add a `devices` property returning `[MediaDevice]`, where each device describes its name, type, and capabilities — including `absoluteVolume` with a change handler. The server communicates state changes via APNs; the extension receives a `RemotePlayerState` value and updates its observable model.

### Media Sharing Extensions
Apps can surface the system audio route picker for third-party speakers and AV receivers without integrating vendor SDKs. The framework handles discovery and routing negotiation through the system.

## APIs & Frameworks

### NowPlaying (new framework — **[NEW]**)
- `MediaSessionRepresentable` — **[NEW]** protocol; adopt on `@Observable` model
  - `id: String`
  - `content: (any MediaContentRepresentable)?`
  - `playbackSnapshot: MediaPlaybackSnapshot?`
  - `commands: [MediaCommand]`
- `MediaSession<Model>` — **[NEW]** generic class; wraps a `MediaSessionRepresentable` model
  - `init(_ model: Model)`
- `MediaContentRepresentable` — **[NEW]** protocol
- `GenericContent` — **[NEW]** concrete content type
  - `id`, `title`, `subtitle`, `type` (`ContentType.audio` / `.video`), `duration` (`.live`, `.tracked`), `artwork`
- `Artwork` — **[NEW]** struct with async image-data closure
- `ArtworkRepresentation` — **[NEW]** wraps raw image data
- `MediaPlaybackSnapshot` — **[NEW]** struct
  - `init(state:)` — with `.playing()`, `.paused`, etc.
- `MediaCommand` — **[NEW]** enum-like; static factory methods: `.play`, `.pause`, `.previous`, `.next`
- `RemoteMediaSessionRepresentable` — **[NEW]** protocol for extension models
  - Adds `devices: [MediaDevice]`
  - `func update(_ state: RemotePlayerState)`
- `RemoteMediaSessionExtension` — **[NEW]** `@main` app extension base protocol
- `RemoteMediaSessionExtensionConfiguration` — **[NEW]** configuration wrapper
- `RemotePlayerState` — **[NEW]** value passed into the extension containing `sessionID`, `isPlaying`, `sound`, `devices`
- `MediaDevice` — **[NEW]** struct: `id`, `name`, `type` (`.speaker`, etc.), `capabilities`
- `MediaDevice.Capability` — **[NEW]** enum: `.absoluteVolume(volume) { newVolume in … }`

### ExtensionFoundation
- `AppExtensionPoint` — used to declare the `remote-media` extension point
- `AppExtensionConfiguration` — configuration protocol for the extension entry point

### Related Frameworks / Resources
- `AVSystemRouting` — [Routing media to third-party devices](https://developer.apple.com/documentation/AVSystemRouting/routing-media-to-third-party-devices)
- `UserNotifications` / APNs — used to push remote player state updates to the app extension
- [Publishing media sessions](https://developer.apple.com/documentation/NowPlaying/publishing-media-sessions)
- [Publishing remote media sessions](https://developer.apple.com/documentation/NowPlaying/publishing-remote-media-sessions)

## Code Highlights

Adopt `MediaSessionRepresentable`:
```swift
extension PlayerModel: MediaSessionRepresentable {
    var id: String { "ambient-sound-session" }
    var content: (any MediaContentRepresentable)? {
        GenericContent(id: sound.id, title: sound.name, subtitle: sound.description,
            type: .audio, duration: .live,
            artwork: Artwork(id: sound.id) { size in
                try ArtworkRepresentation(data: await self.artworkData(size: size))
            })
    }
    var playbackSnapshot: MediaPlaybackSnapshot? {
        MediaPlaybackSnapshot(state: player.isPlaying ? .playing() : .paused)
    }
    var commands: [MediaCommand] {[
        .play { self.player.play() },
        .pause { self.player.pause() },
        .previous { self.player.previous() },
        .next { self.player.next() }
    ]}
}
```

Instantiate the session:
```swift
let session = MediaSession(model)
```

Remote extension entry point:
```swift
@main
final class SampleAppExtension: @MainActor RemoteMediaSessionExtension {
    var configuration: some AppExtensionConfiguration {
        RemoteMediaSessionExtensionConfiguration(extension: self)
    }
    func session(_ state: RemotePlayerState) async throws -> RemotePlayerModel {
        RemotePlayerModel(state: state)
    }
}
```

## Takeaways
- Now Playing replaces `MPNowPlayingInfoCenter` and `MPRemoteCommandCenter` with a type-safe, observable Swift API; adopt `MediaSessionRepresentable` on any `@Observable` model to get Lock Screen, Dynamic Island, and CarPlay integration for free.
- Remote media sessions enable apps to represent playback on external speakers or other devices in the same system surfaces, pushed via APNs to an app extension.
- Media Sharing Extensions let apps participate in the system audio route picker without bundling third-party SDKs.
- The entire framework is Swift-first with `async`/`await` command handlers and Observation-compatible state publishing.

---
_Source: WWDC26 Session 312 page (abstract, chapter summaries, code samples, and resource links)._
