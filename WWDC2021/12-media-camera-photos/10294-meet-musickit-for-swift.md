# Meet MusicKit for Swift
**WWDC21 · Session 10294** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10294/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
MusicKit is a new Swift-native framework for integrating Apple Music into apps. It wraps the Apple Music API with strongly-typed Swift value types, async/await concurrency support, and deep SwiftUI integration. Developers can search and browse the Apple Music catalog, access music items with lazy-loaded relationships, play content through system or application-scoped players, observe subscription state, and present subscription offer sheets — all without managing developer or user tokens manually.

The framework removes two major barriers to Apple Music integration: automatic developer token generation (via an App ID entitlement in the developer portal) and automatic user token generation. Combined with a model layer designed for Swift's type system and async/await, MusicKit significantly reduces the boilerplate previously required to build music-integrated apps.

## Key Topics

**Music Item Model**
Value types like `Song`, `Album`, `Artist`, `Playlist`, `Track`, `Genre`, and `MusicVideo` with three property categories: attributes (title, artwork, isCompilation), relationships (tracks, artists — strongly typed collections), and associations (relatedAlbums — editorially driven, include optional title).

**Requests**
`MusicCatalogSearchRequest` — searches across multiple resource types. `MusicCatalogResourceRequest<T>` — fetches resources matching a filter (e.g., UPC barcode). `MusicDataRequest` — raw Apple Music API URL fetch with JSON decoding via `Codable`.

**Authorization**
`MusicAuthorization.request() async -> MusicAuthorization.Status` — requests user consent (single prompt per device/app pair). `Info.plist` key `NSAppleMusicUsageDescription` provides the purpose string shown in the consent dialog.

**Tokens (Automatic)**
Developer and user tokens are now automatically generated — no server-side token generation needed. Opt in by enabling the MusicKit capability on the App ID in the developer portal.

**Subscription State**
`MusicSubscription` with properties `canPlayCatalogContent`, `hasCloudLibraryEnabled`, `canBecomeSubscriber`. Observable via `MusicSubscription.subscriptionUpdates` async sequence.

**Playback**
`SystemMusicPlayer` — remote-controls the system Music app; Music app is the now-playing app. `ApplicationMusicPlayer` — app-owned independent playback queue; app is the now-playing app. Both support setting the queue, play/pause, now-playing info, and remote commands.

**Subscription Offers**
`MusicSubscriptionOffer` — presents a native Apple Music subscription offer sheet, optionally contextual (highlighting a specific song, album, or playlist). Affiliates earn credit via the Apple Services Performance Partners Program.

## APIs & Frameworks

### MusicKit Framework **[NEW]**

**Music Item Types**
- `Song` — individual track with attributes and relationships **[NEW]**
- `Album` — with `.tracks`, `.artists`, `.relatedAlbums` relationships/associations **[NEW]**
- `Artist` — with `.albums`, `.songs`, `.playlists` **[NEW]**
- `Playlist` — with `.tracks` **[NEW]**
- `Track` — enum-like type representing a song or music video **[NEW]**
- `Genre` — with `.name` **[NEW]**
- `MusicVideo` — music video type **[NEW]**
- `Artwork` — struct with `.url(width:height:)` and color information **[NEW]**
- `MusicItemCollection<T>` — paginated collection of music items; `.title` for associations **[NEW]**
  - `.nextBatch(limit:)` — loads the next page **[NEW]**

**Relationships / Associations Loading**
- `MusicItem.with(_:)` — async method to fetch a detailed version with specified relationships/associations **[NEW]**
  - `MusicItemRelationshipCodingKeys` — relationship names: `.artists`, `.tracks`, `.relatedAlbums`, etc.

**Requests**
- `MusicCatalogSearchRequest` — search across types **[NEW]**
  - `.term`, `.types` properties
  - `.response() async throws -> MusicCatalogSearchResponse`
- `MusicCatalogResourceRequest<MusicItem>` — filter-based resource fetch **[NEW]**
  - `init(matching:equalTo:)` — e.g., matching `.upc`, equalTo barcode string
  - `.response() async throws -> MusicCatalogResourceResponse<T>`
- `MusicDataRequest` — raw URL request returning JSON data **[NEW]**
  - `init(urlRequest:)`
  - `.response() async throws -> MusicDataResponse`
  - `MusicDataRequest.currentCountryCode` — async property for localized catalog access **[NEW]**

**Authorization**
- `MusicAuthorization.request() async -> MusicAuthorization.Status` **[NEW]**
- `MusicAuthorization.Status` — `.authorized`, `.denied`, `.notDetermined`, `.restricted`
- `NSAppleMusicUsageDescription` — Info.plist key

**Subscription**
- `MusicSubscription` — struct with `canPlayCatalogContent`, `hasCloudLibraryEnabled`, `canBecomeSubscriber` **[NEW]**
- `MusicSubscription.subscriptionUpdates` — `AsyncSequence` of subscription changes **[NEW]**

**Playback**
- `SystemMusicPlayer` — singleton, controls system Music app **[NEW]**
  - `.shared` property
- `ApplicationMusicPlayer` — singleton, app-owned queue **[NEW]**
  - `.shared` property
  - `.queue` — `ApplicationMusicPlayer.Queue` with insert/remove operations **[NEW]**
- Both players: `.play()`, `.pause()`, `.skipToNextEntry()`, `.skipToPreviousEntry()` **[NEW]**
- `ApplicationMusicPlayer.Queue.Entry` — wraps a music item in the queue **[NEW]**

**Subscription Offer UI**
- `MusicSubscriptionOffer` — SwiftUI view modifier namespace **[NEW]**
- `MusicSubscriptionOffer.Options` — `itemID`, `affiliateToken`, `campaignToken`, `messageIdentifier` **[NEW]**
- `.musicSubscriptionOffer(isPresented:options:)` — SwiftUI view modifier **[NEW]**

## Code Highlights

Fetch album with tracks and related albums:
```swift
let detailedAlbum = try await album.with([.artists, .tracks, .relatedAlbums])
if let tracks = detailedAlbum.tracks {
    tracks.prefix(2).forEach { print($0) }
}
```

Find album by UPC barcode:
```swift
var albumsRequest = MusicCatalogResourceRequest<Album>(matching: \.upc, equalTo: detectedBarcode)
let response = try await albumsRequest.response()
let firstAlbum = response.items.first
```

Drive play button from subscription state:
```swift
Button(action: handlePlayButtonSelected) {
    Image(systemName: "play.fill")
}
.disabled(!(musicSubscription?.canPlayCatalogContent ?? false))
.task {
    for await subscription in MusicSubscription.subscriptionUpdates {
        musicSubscription = subscription
    }
}
```

Show contextual subscription offer:
```swift
var offerOptions = MusicSubscriptionOffer.Options()
offerOptions.itemID = album.id
Button("Join Apple Music") { isShowingOffer = true }
    .disabled(!(musicSubscription?.canBecomeSubscriber ?? false))
    .musicSubscriptionOffer(isPresented: $isShowingOffer, options: offerOptions)
```

## Takeaways
- MusicKit eliminates the need for server-side token generation — enabling the MusicKit App ID capability makes both developer and user tokens automatic.
- The `MusicCatalogResourceRequest<T>` filter API enables powerful catalog lookups like barcode/UPC matching in just a few lines.
- `MusicSubscription.subscriptionUpdates` is an `AsyncSequence` that works directly with SwiftUI's `.task` modifier, enabling reactive subscription-state-driven UI with minimal boilerplate.
- `ApplicationMusicPlayer` keeps playback state fully isolated from the system Music app, making it the right choice for fitness, game, or social apps that manage their own music experience.

---
_Source: WWDC21 Session 10294 page (abstract, chapter summaries, code samples, and resource links)._
