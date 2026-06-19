# Explore more content with MusicKit
**WWDC22 · Session 110347** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110347/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
MusicKit received major expansions in 2022 across four areas: new catalog content types and search enhancements, personalized content (recently played items and personal recommendations), user library access, and library mutation (adding items and creating/editing playlists). Together these upgrades allow apps to deliver a fully integrated music experience — browsing and playing catalog content, surfacing personalized recommendations, and reading/writing the user's own music library — entirely through the Swift MusicKit API.

## Key Topics

**New catalog types** — `Curator` (editorial and external curators like Nike, Shazam) and `RadioShow` (e.g., "New Music Daily") are new first-class `MusicItem` types with `name`, `url`, `artwork`, `kind`, and a `playlists` relationship. `Playlist` gains `curator` and `radioShow` reverse relationships.

**Search enhancements** — `MusicCatalogSearchRequest.includeTopResults = true` populates a new `topResults` property returning a type-agnostic, relevancy-ordered `MusicItemCollection`. `MusicCatalogSearchSuggestionsRequest` provides autocomplete `Suggestion` objects with `displayTerm` (for UI) and `searchTerm` (for the follow-up search request).

**Catalog charts** — `MusicCatalogChartsRequest` fetches top songs, albums, playlists, and music videos by kind: `.mostPlayed`, `.dailyGlobalTop`, `.cityTop`. Supports genre filtering. Replaces custom `MusicDataRequest` usage.

**Audio variants** — `Song` and `Album` gain an `audioVariants` property (loaded via `.with(.audioVariants)`) returning an array of `AudioVariant` values such as `.dolbyAtmos`, `.lossless`. `ApplicationMusicPlayer.shared.state.audioVariant` exposes the currently playing variant for live UI updates.

**Personalized content** — `MusicRecentlyPlayedContainerRequest` fetches recently played albums, playlists, and stations. `MusicRecentlyPlayedRequest<T>` fetches recently played items of a specific type (e.g., `Song`). `MusicPersonalRecommendationsRequest` returns themed recommendation groups with `title`, `nextRefreshDate`, and mixed-type item collections.

**User library requests** — `MusicLibraryRequest<T>` fetches library items by type with `filter(matching:equalTo:)`, `filter(matching:contains:)`, and `includeDownloadedContentOnly` for offline-only results. `MusicLibrarySectionedRequest<Section, Item>` groups items by a section type (e.g., `Genre`). `MusicLibrarySearchRequest` searches within the library. The `with(_:preferredSource:)` method accepts `.library` to load relationships from the library rather than the catalog.

**Library mutations** — `MusicLibrary.shared.add(_:to:)` adds an item to a playlist. `MusicLibrary.shared.add(_:)` adds items to the library. New APIs for creating playlists, editing playlist metadata, and editing the track list of library-owned playlists.

## APIs & Frameworks

### MusicKit — Catalog
- `Curator` **[NEW]** — `.name`, `.url`, `.artwork`, `.kind` (`.editorial` / `.external`), `.playlists`
- `RadioShow` **[NEW]** — `.name`, `.url`, `.artwork`, `.playlists`
- `Playlist.curator` relationship **[NEW]**
- `Playlist.radioShow` relationship **[NEW]**
- `MusicCatalogSearchRequest` — existing catalog search
- `MusicCatalogSearchRequest.includeTopResults` **[NEW]** — enables `topResults` in response
- `MusicCatalogSearchResponse.topResults` **[NEW]** — type-agnostic `MusicItemCollection`
- `MusicCatalogSearchSuggestionsRequest` **[NEW]** — autocomplete suggestions
- `MusicCatalogSearchSuggestionsResponse.suggestions` — array of `Suggestion`
- `Suggestion.displayTerm` — string for UI
- `Suggestion.searchTerm` — string for follow-up `MusicCatalogSearchRequest`
- `MusicCatalogChartsRequest` **[NEW]** — top charts request
- `MusicCatalogChartsRequest.init(kinds:types:)` — takes `MusicCatalogChartKind` set and type array
- `MusicCatalogChartKind` — `.mostPlayed`, `.dailyGlobalTop`, `.cityTop`
- `MusicCatalogChartsResponse` — `.songCharts`, `.albumCharts`, `.playlistCharts`, etc.

### MusicKit — Audio & Playback
- `Song.audioVariants` / `Album.audioVariants` — `[AudioVariant]` **[NEW]**
- `AudioVariant` — `.dolbyAtmos`, `.lossless`, `.lossy`, `.highResolutionLossless`, etc.
- `MusicItem.isAppleDigitalMaster` **[NEW]** — Bool indicating highest-quality master
- `ApplicationMusicPlayer.shared.state.audioVariant` — currently playing `AudioVariant` **[NEW]**
- `ApplicationMusicPlayer.shared.queue` — `@ObservedObject` for current entry

### MusicKit — Personalized Content
- `MusicRecentlyPlayedContainerRequest` **[NEW]** — recently played albums, playlists, stations
- `MusicRecentlyPlayedRequest<T>` **[NEW]** — recently played items of specific type
- `MusicPersonalRecommendationsRequest` **[NEW]** — personal recommendations
- `MusicPersonalRecommendation` — `.id`, `.title`, `.nextRefreshDate`, `.playlists`, `.albums`, `.items`

### MusicKit — Library
- `MusicLibraryRequest<T>` **[NEW]** — fetch library items by type
- `MusicLibraryRequest.filter(matching:equalTo:)` — type-safe filter on key path
- `MusicLibraryRequest.filter(matching:contains:)` — relationship filter
- `MusicLibraryRequest.includeDownloadedContentOnly` — offline-only filter
- `MusicLibraryResponse<T>` — `.items: MusicItemCollection<T>`
- `MusicLibrarySectionedRequest<Section, Item>` **[NEW]** — sectioned fetch
- `MusicLibrarySectionedRequest.sortItems(by:ascending:)` — sort items within sections
- `MusicLibrarySectionedRequest.sortSections(by:ascending:)` — sort sections
- `MusicLibrarySectionedResponse` — `.sections: [Section]`, each section has `.items`
- `MusicLibrarySearchRequest` **[NEW]** — search within library
- `MusicItem.with(_:preferredSource:)` **[NEW]** — load relationship from `.library` or `.catalog`

### MusicKit — Library Mutations
- `MusicLibrary.shared` — shared library instance **[NEW]**
- `MusicLibrary.shared.add(_:to:)` **[NEW]** — add item to a playlist
- `MusicLibrary.shared.add(_:)` **[NEW]** — add item to the library
- Playlist creation and metadata/track editing **[NEW]**

## Code Highlights

Loading catalog top results and suggestions:
```swift
var searchRequest = MusicCatalogSearchRequest(term: "Hello", types: [Artist.self, Album.self, Song.self])
searchRequest.includeTopResults = true
let searchResponse = try await searchRequest.response()
print(searchResponse.topResults)  // type-agnostic, relevancy-ordered

let suggestionsRequest = MusicCatalogSearchSuggestionsRequest(term: "shaz")
let suggestions = try await suggestionsRequest.response()
```

Fetching library albums with filters:
```swift
var request = MusicLibraryRequest<Album>()
request.filter(matching: \.isCompilation, equalTo: true)
request.filter(matching: \.genres, contains: danceGenre)
request.includeDownloadedContentOnly = true
let response = try await request.response()
```

Adding a track to a playlist:
```swift
try await MusicLibrary.shared.add(selectedTrack, to: playlist)
```

Showing active audio variant in a SwiftUI view:
```swift
@ObservedObject var musicPlayerState = ApplicationMusicPlayer.shared.state
// ...
if musicPlayerState.audioVariant == .dolbyAtmos {
    Image("dolby-atmos-badge")
}
```

## Takeaways
- `MusicCatalogChartsRequest` replaces custom `MusicDataRequest` chart calls and includes built-in pagination support.
- Library requests (`MusicLibraryRequest`, `MusicLibrarySectionedRequest`) load from the on-device library copy — no network needed — and support offline-only filtering with `includeDownloadedContentOnly`.
- `MusicPersonalRecommendationsRequest` and `MusicRecentlyPlayedRequest` handle user-token authentication automatically; no manual token management required.
- The new `MusicLibrary.shared` mutation API enables apps to add songs, create playlists, and edit track lists without ever leaving your app's context.

---
_Source: WWDC22 Session 110347 page (abstract, chapter summaries, code samples, and resource links)._
