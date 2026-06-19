# Integrate MusicKit into Your App
**WWDC26 · Session 254** · [Watch](https://developer.apple.com/videos/play/wwdc2026/254/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, watchOS, visionOS

## Overview
This session is a comprehensive introduction to MusicKit, Apple's Swift framework for integrating Apple Music into third-party apps. It covers the complete flow from Xcode capability configuration and user authorization through music selection, playback control, and structured catalog requests — demonstrated through a workout app that plays Apple Music tracks during exercise sessions.

The session introduces the new Music Picker (a SwiftUI view modifier) that lets users browse both the Apple Music catalog and their personal library without the app needing to implement its own search UI. It also clarifies the distinction between `SystemMusicPlayer` (controls the system-wide music player, affecting the Music app) and `ApplicationMusicPlayer` (a private queue scoped to the app).

Cross-storefront song sharing and `MusicCatalogResourceRequest` with the `findEquivalents` option round out the catalog access coverage.

## Key Topics

### Project Setup and Authorization
Enable the "Media & Apple Music" capability in Xcode and add the `NSAppleMusicUsageDescription` key to `Info.plist`. Request authorization with `MusicAuthorization.request()`. Check `MusicSubscription.current` to gate paid features, and observe `MusicSubscription.subscriptionUpdates` (an `AsyncSequence`) for changes. Present a subscription offer UI via the `.musicSubscriptionOffer(isPresented:options:)` modifier.

### Music Items and Music Picker
MusicKit music items (`Song`, `Album`, `Artist`, `Playlist`, etc.) are typed value types with rich property and relationship APIs. The new `.musicPicker(isPresented:selection:)` SwiftUI modifier presents Apple's built-in picker UI for catalog and library browsing, writing the user's selection into a binding.

### Music Players and Playback
`SystemMusicPlayer.shared` controls the system queue (shared with Music.app). `ApplicationMusicPlayer.shared` manages a private queue. Set a queue by assigning `player.queue = ApplicationMusicPlayer.Queue(for: songs, startingAt: song)`. Observe `ApplicationMusicPlayer.shared.state` (an `@Observable`-compatible object) for `playbackStatus` changes. `ApplicationMusicPlayer.shared.queue` provides `currentEntry` with `title`, `subtitle`, and `artwork`.

Key playback methods: `player.play()`, `player.pause()`, `player.skipToNextEntry()`, `player.skipToPreviousEntry()` — all `async throws`.

Display artwork with `ArtworkImage(artwork, width:height:)`.

### Catalog Requests
`MusicCatalogResourceRequest<Song>` (and other resource types) queries the Apple Music catalog by matching a key path against a collection of values. The `.findEquivalents` option in `request.options` resolves song IDs across storefronts. Use `response.item(for: id)` to retrieve individual results from the response.

### Cross-Storefront Song Sharing
Pass storefront-specific `MusicItemID` values; use `findEquivalents` to resolve them to the user's local storefront automatically.

## APIs & Frameworks

### MusicKit
- `MusicAuthorization` — `request()` async → authorization status
- `MusicSubscription` — `current` async property, `subscriptionUpdates` AsyncSequence, `canBecomeSubscriber` Bool
- `MusicSubscriptionOffer.Options` — `messageIdentifier` (e.g., `.playMusic`)
- `.musicSubscriptionOffer(isPresented:options:)` — SwiftUI view modifier
- `Song`, `Album`, `Artist`, `Playlist`, `MusicVideo` — typed MusicKit item types
- `MusicItemID` — typed identifier for catalog items
- `.musicPicker(isPresented:selection:)` — **[NEW]** SwiftUI view modifier; presents system music picker
- `SystemMusicPlayer` — controls system-wide queue
- `ApplicationMusicPlayer` — app-private queue player
  - `.shared` — singleton
  - `queue` — `ApplicationMusicPlayer.Queue`; assign to set playback queue
  - `queue.currentEntry` — current `MusicPlayer.Queue.Entry`; `.title`, `.subtitle`, `.artwork`
  - `state` — `MusicPlayer.State`; `.playbackStatus` (`.playing`, `.paused`, etc.)
  - `play()` — async throws
  - `pause()`
  - `skipToNextEntry()` — async throws
  - `skipToPreviousEntry()` — async throws
- `ArtworkImage(_:width:height:)` — SwiftUI view for artwork
- `Artwork` — value type with URL and color info
- `MusicCatalogResourceRequest<MusicItemType>` — catalog query
  - `init(matching:memberOf:)` — filter by key path + value collection
  - `options` — `MusicDataRequest.Options`; supports `.findEquivalents`
  - `response()` async throws — returns `MusicCatalogResourceResponse`
  - `response.item(for:)` — look up individual result by `MusicItemID`
- `MusicCatalogSearchRequest` — free-text catalog search

### SwiftUI
- `.musicSubscriptionOffer(isPresented:options:)` modifier
- `.musicPicker(isPresented:selection:)` modifier — **[NEW]**
- `ArtworkImage` view

### Resources
- [MusicKit documentation](https://developer.apple.com/documentation/musickit)
- [Integrating MusicKit into your app](https://developer.apple.com/documentation/MusicKit/integrating-musickit-into-your-app)
- [Apple Services Performance Partner Program](https://performance-partners.apple.com/home)

## Code Highlights

Present subscription offer and observe status:
```swift
.task(id: isAuthorized) {
    self.subscription = try? await MusicSubscription.current
    for await subscription in MusicSubscription.subscriptionUpdates {
        self.subscription = subscription
    }
}
```

New music picker modifier:
```swift
Button("Pick some Music", systemImage: "music.note.list") { showMusicPicker = true }
    .musicPicker(isPresented: $showMusicPicker, selection: $selectedSong)
```

Cross-storefront catalog request:
```swift
var request = MusicCatalogResourceRequest<Song>(matching: \.id, memberOf: songIDs)
request.options = [.findEquivalents]
let response = try await request.response()
let featuredSong = response.item(for: songIDs[0])
```

## Takeaways
- The new `.musicPicker` modifier eliminates the need to build custom music browsing UI — Apple's system picker handles catalog search and library access in one view.
- `ApplicationMusicPlayer` is the right choice for apps that need an independent queue; `SystemMusicPlayer` is appropriate only when the app intentionally wants to control the system music experience.
- `MusicSubscription.subscriptionUpdates` is an `AsyncSequence` — use `for await` in a `.task` modifier to react to subscription status changes reactively.
- Use `MusicCatalogResourceRequest` with `.findEquivalents` for cross-storefront sharing scenarios to transparently resolve song IDs across regions.

---
_Source: WWDC26 Session 254 page (abstract, chapter summaries, code samples, and resource links)._
