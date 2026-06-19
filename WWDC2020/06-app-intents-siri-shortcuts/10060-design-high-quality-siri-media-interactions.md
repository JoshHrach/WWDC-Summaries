# Design High Quality Siri Media Interactions
**WWDC20 · Session 10060** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10060/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This session focuses on building high-quality Siri experiences for music and audio apps using SiriKit Media Intents. The core principle is simple: when someone asks Siri to play something, something must actually play. Reliability is paramount because the trust barrier for voice assistants is higher than traditional UIs, and a failed first interaction can permanently discourage users.

The session examines the most common natural-language patterns across SiriKit Media apps, showing how just four request types capture more than 90% of real-world traffic. It also covers how to teach Siri your app's custom vocabulary so that unusual playlist names, artist names, and other catalog entities are correctly recognized — rather than misinterpreted by the underlying ML model.

Now Playing controls via `MPRemoteCommandCenter` and metadata via `MPNowPlayingInfoCenter` are treated as first-class Siri interactions, so implementing them correctly ensures that commands like "next track," "skip forward," and "what song is this?" all work seamlessly.

## Key Topics

### Start Playback Reliably and Quickly
Timeouts are one of the biggest failure modes, especially in CarPlay where hands-free environments apply stricter limits. Robust playback stacks and fast intent handling are the first engineering priorities.

### High-Traffic Utterance Patterns
Four patterns cover 90%+ of SiriKit Media requests:
1. **Generic play** ("Play ControlAudio") — `mediaSearch` is nil or contains only `mediaType = .music` (~50%+ of traffic).
2. **Named item** ("Play Special Disaster Team") — only `mediaName` is populated (~30%).
3. **Artist + title compound** ("Play Maybe Sometime by Special Disaster Team") — both `mediaName` and `artistName` populated (~5%).
4. **Playlist** ("Play my WWDC playlist") — `mediaType = .playlist`, `mediaName` set (~5%).

### SiriKit Vocabulary APIs
Custom vocabulary corrects the ML model when catalog-specific phrases (e.g., "70s punk classics") would otherwise be parsed as generic attributes (decade + genre). Two mechanisms are available:
- **User vocabulary** (`INVocabulary`) — per-user data such as personal playlists.
- **Global vocabulary** (`.plist` bundled with the app) — catalog-wide terms available to all users.

### Now Playing Commands
Siri routes voice commands to `MPRemoteCommandCenter` handlers. Commands include `pauseCommand`, `playCommand`, `previousTrackCommand`, `nextTrackCommand`, `skipForwardCommand`, `skipBackwardCommand`, `changeRepeatModeCommand`, `changeShuffleModeCommand`, and `changePlaybackRateCommand`.

### Now Playing Metadata for Siri Questions
Setting properties on `MPNowPlayingInfoCenter` enables Siri to answer "What song is this?", "What band is this?", and "What album is this?" naturally.

### Educating Users About Siri
Siri engagement can increase up to 10x when apps include in-app education about available voice commands. Prompting users to try Siri dramatically increases adoption.

## APIs & Frameworks

### SiriKit / Intents
- `INPlayMediaIntent` **[NEW in iOS 13, extended in iOS 14]**
- `INMediaSearch` — carries `mediaName`, `artistName`, `mediaType`, `moodNames`, `genreNames`, `releaseDate`, `sortOrder`
- `INMediaType` — `.music`, `.playlist`, `.audioBook`, `.podcastShow`, etc.
- `INPlayMediaMediaItemResolutionResult` — `successes(with:)`
- `INVocabulary` — `shared()`, `setVocabularyStrings(_:of:)` **[NEW vocabulary types in iOS 14]**
- `INVocabularyStringType.mediaPlaylistTitle`
- `INVocabularyStringType.musicArtistName`
- `INVocabularyStringType.audiobookTitle`
- `INVocabularyStringType.audiobookAuthorName`
- `INVocabularyStringType.mediaShowTitle` (radio/podcasts)
- Global vocabulary `.plist` keys: `ParameterVocabularies`, `ParameterNames` (`INPlayMediaIntent.playlistTitle`), `ParameterVocabulary`, `VocabularyItemIdentifier`, `VocabularyItemSynonyms`, `VocabularyItemPhrase`

### MediaPlayer
- `MPRemoteCommandCenter` — `shared()`, `.pauseCommand`, `.playCommand`, `.previousTrackCommand`, `.nextTrackCommand`, `.skipForwardCommand`, `.skipBackwardCommand`, `.changeRepeatModeCommand`, `.changeShuffleModeCommand`, `.changePlaybackRateCommand`
- `MPRemoteCommandEvent` — `.interval` property on skip commands
- `MPNowPlayingInfoCenter` — `default()`, `nowPlayingInfo` dictionary
- `MPMediaItemPropertyTitle`
- `MPMediaItemPropertyArtist`
- `MPMediaItemPropertyAlbumTitle`

## Code Highlights

Resolving media items from a Siri intent:
```swift
func resolveMediaItems(for intent: INPlayMediaIntent,
                       with completion: @escaping ([INPlayMediaMediaItemResolutionResult]) -> Void) {
    let mediaSearch = intent.mediaSearch
    resolveMediaItems(for: mediaSearch) { optionalMediaItems in
        guard let mediaItems = optionalMediaItems else { return }
        completion(INPlayMediaMediaItemResolutionResult.successes(with: mediaItems))
    }
}
```

Setting user vocabulary for playlist names:
```swift
let vocabulary = INVocabulary.shared()
let playlistNames = NSOrderedSet(objects: "70s punk classics")
vocabulary.setVocabularyStrings(playlistNames, of: .mediaPlaylistTitle)
```

Global vocabulary `.plist` structure:
```xml
<key>ParameterVocabularies</key>
<array>
  <dict>
    <key>ParameterNames</key>
    <array><string>INPlayMediaIntent.playlistTitle</string></array>
    <key>ParameterVocabulary</key>
    <array>
      <dict>
        <key>VocabularyItemIdentifier</key><string>70s punk anthems</string>
        <key>VocabularyItemSynonyms</key>
        <array><dict><key>VocabularyItemPhrase</key><string>70s punk anthems</string></dict></array>
      </dict>
    </array>
  </dict>
</array>
```

## Takeaways
- Always play something when asked — even a "perfect-ish" generic result is far better than silence or an error.
- Sync `INVocabulary` user vocabulary (ordered by importance) for per-user catalog items; bundle a global vocabulary `.plist` for app-wide catalog entities.
- Implement `MPRemoteCommandCenter` handlers and `MPNowPlayingInfoCenter` metadata so Now Playing voice commands and questions work without extra code.
- Educate users about Siri capabilities in-app — engagement can increase up to 10x.

---
_Source: WWDC20 Session 10060 page (abstract, transcript, code samples, and resource links)._
