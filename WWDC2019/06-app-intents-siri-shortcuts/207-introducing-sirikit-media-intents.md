# Introducing SiriKit Media Intents
**WWDC19 · Session 207** · [Watch](https://developer.apple.com/videos/play/wwdc2019/207/)

_Platforms:_ iOS 13, iPadOS 13, watchOS 6 (CarPlay, HomePod, AirPods via Siri)

## Overview
iOS 13 adds a new SiriKit Media domain, enabling audio apps (music, podcasts, audiobooks, radio) to respond to voice commands for playback control, playlist management, taste-preference signals, and catalog search. Unlike Shortcuts (which operates on donated intents), the SiriKit Media domain uses Siri's NLU pipeline to parse rich natural-language queries — "Play the newest Stuff You Should Know podcast in my app at double speed" — and deliver a structured `INMediaSearch` object to the app's Intents extension for resolution.

The four new intents handle the full content control lifecycle: play, add, update affinity (like/dislike), and search. Resolution happens in the Intents app extension; playback itself is performed via a background app launch on iOS (foreground on watchOS). The session covers the new intents, the resolve/handle flow, best practices for flexible search matching, error reporting, and how existing Shortcuts implementations can be extended with a single resolve method to gain full SiriKit support.

## Key Topics

**Four SiriKit Media Intents**
- `INPlayMediaIntent` — "Play Khalid on my app" / "Play Outer Peace next in my app."
- `INAddMediaIntent` — "Add this song to my road trip playlist in my app."
- `INUpdateMediaAffinityIntent` — "I like this song" / "I hate this song."
- `INSearchForMediaIntent` — "Find Billie Eilish in my app."

**Supported Media Types and Search Properties**
`INMediaSearch` carries a structured parse of the user's utterance with properties for: `mediaName`, `albumName`, `artistName`, `genreName`, `playlistName`, `moodNames`, `mediaType` (`.music`, `.podcast`, `.audioBook`, `.radioStation`, `.musicStation`, `.algorithmicRadioStation`, `.podcastShow`, `.podcastEpisode`, `.podcastPlaylist`), `sortOrder` (`INMediaSortOrder.newest`, `.oldest`, `.best`, `.worst`, `.popular`, `.unpopular`, `.recommended`), `reference` (`INMediaReference.currentlyPlaying`), and `identifier` (from `MPNowPlayingInfoCenter.externalContentIdentifier`).

If the user's utterance type does not match a strongly parsed type, the `mediaName` still carries the search string for generic lookup.

**Request Processing: Resolve → Handle (skip Confirm)**
- Resolve: implement `resolveMediaItems(for intent:with completion:)`. Search the app catalog against `intent.mediaSearch`. Return `INPlayMediaMediaItemResolutionResult.success(with:)` with one or more `INMediaItem` objects, or `.unsupported(forReason:)`.
- Confirm: skip in the media domain — adding dialog before playback significantly reduces completion rates.
- Handle (`INPlayMediaIntent`): return `.handleInApp(successOrFailure)` — triggers a background app launch. The app delegate's `application(_:handle:completionHandler:)` reads the resolved media items and starts playback. For `INAddMediaIntent` and `INUpdateMediaAffinityIntent`, perform the operation in the extension itself without app launch.

**watchOS Differences**
Return `INPlayMediaIntentResponseCode.continueInApp` (not `.handleInApp`) from handle. This triggers a foreground app launch. Read the intent from `WKExtensionDelegate.handleUserActivity(_:)` via the `interaction.intent` property. Use on-device cache in resolve; avoid network calls.

**Search Flexibility (Critical Best Practice)**
Direct string comparison against `INMediaSearch.mediaName` fails in common cases. Implement case-insensitive, punctuation-insensitive matching. Strip common suffixes/modifiers that users never say: "Deluxe Edition," "From the Motion Picture," "feat." / "featuring", "Podcast" (from podcast titles), "Audio" / "Video" (edition variants). Handle speech-recognizer variants: numeric vs. hyphenated ordinals (81st vs. eighty-first), homophones (son/sun).

**Error Cases**
`INPlayMediaMediaItemUnsupportedReason`: `.loginRequired`, `.subscriptionRequired`, `.unsupportedMediaType`, `.notExplicitlyBeingSought`, `.regionRestriction`, `.cellularDataSettings`, `.explicitContentSettings`. Parallel reasons exist for `INAddMediaIntent`, `INUpdateMediaAffinityIntent`, and `INSearchForMediaIntent`. Returning an appropriate unsupported reason lets Siri speak a meaningful error to the user.

**"Play my app" — No Search Criteria**
When `INMediaSearch` has no search criteria set, the user said "Play my app" without specifying content. The recommended behavior: resume the current queue, play a recommended playlist, or surface trending content. Do not prompt the user for what to play — this breaks the hands-free experience.

**Returned INMediaItem Influences Siri Dialogue**
Populate `title`, `artist`, and `type` on every returned `INMediaItem`. Siri uses these fields to speak confirmation. If multiple items are returned, Siri speaks the first one. Include `identifier` to allow cross-referencing with `MPNowPlayingInfoCenter.externalContentIdentifier`.

**User Vocabulary**
Donate user-specific vocabulary to help Siri recognize custom catalog names: music apps donate playlist titles and music artist names; audiobook apps donate titles and author names; podcast apps donate show titles. Vocabulary is ordered — most important items first. See `INVocabulary` and global vocabulary for app-wide terms.

**Migrating from Shortcuts**
Shortcuts (iOS 12) for media playback used `INPlayMediaIntent` without a resolve method. To add full SiriKit support: add the resolve method, update the Info.plist supported intents and media types in the Intents extension. The handle method and background app launch are the same between both implementations.

## APIs & Frameworks

**SiriKit / Intents** (iOS 13) **[NEW]**

Intents:
- `INPlayMediaIntent` **[NEW — extended from iOS 12 Shortcuts]**
  - `INPlayMediaIntentHandling` protocol: `resolveMediaItems(for:with:)`, `handle(intent:completion:)` **[NEW methods]**
  - `INPlayMediaMediaItemResolutionResult` **[NEW]**
  - `INPlayMediaIntentResponseCode.handleInApp` **[NEW]**
  - `INPlayMediaIntentResponseCode.continueInApp` (watchOS) **[NEW]**
  - `INPlayMediaMediaItemUnsupportedReason` **[NEW]** — full enum of error reasons
- `INAddMediaIntent` **[NEW]**
  - `INAddMediaIntentHandling`: `resolveMediaItems(for:with:)`, `resolveMediaDestination(for:with:)`, `handle(intent:completion:)` **[NEW]**
  - `INAddMediaMediaDestinationResolutionResult` **[NEW]**
  - `INAddMediaMediaItemUnsupportedReason` **[NEW]**
- `INUpdateMediaAffinityIntent` **[NEW]**
  - `INUpdateMediaAffinityIntentHandling` **[NEW]**
  - `INMediaAffinityType`: `.like`, `.dislike` **[NEW]**
- `INSearchForMediaIntent` **[NEW]**
  - `INSearchForMediaIntentHandling` **[NEW]**

Media search and items:
- `INMediaSearch` **[NEW]**
  - `.mediaName: String?` **[NEW]**
  - `.albumName: String?` **[NEW]**
  - `.artistName: String?` **[NEW]**
  - `.genreName: String?` **[NEW]**
  - `.playlistName: String?` **[NEW]**
  - `.moodNames: [String]?` **[NEW]**
  - `.mediaType: INMediaItemType` **[NEW]**
  - `.sortOrder: INMediaSortOrder` **[NEW]**
  - `.reference: INMediaReference` **[NEW]**
  - `.identifier: String?` **[NEW]**
- `INMediaItem` **[NEW]**
  - `.identifier: String?` **[NEW]**
  - `.title: String?` **[NEW]**
  - `.type: INMediaItemType` **[NEW]**
  - `.artwork: INImage?` **[NEW]**
  - `.artist: String?` **[NEW]**
- `INMediaItemType`: `.music`, `.podcast`, `.audioBook`, `.radioStation`, `.musicStation`, `.algorithmicRadioStation`, `.podcastShow`, `.podcastEpisode`, `.podcastPlaylist`, `.unknown` **[NEW]**
- `INMediaSortOrder`: `.newest`, `.oldest`, `.best`, `.worst`, `.popular`, `.unpopular`, `.recommended` **[NEW]**
- `INMediaReference`: `.currentlyPlaying` **[NEW]**

Playback controls on INPlayMediaIntent:
- `INPlaybackRepeatMode`: `.none`, `.one`, `.all` **[NEW]**
- `INPlaybackQueueLocation`: `.now`, `.next`, `.later` **[NEW]**
- `INPlayShuffleMode`: `.on`, `.off` **[NEW]**

App delegate background launch (existing since iOS 12):
- `UIApplicationDelegate.application(_:handle:completionHandler:)` — now used for media intents

User vocabulary:
- `INVocabulary.shared().setVocabularyStrings(_:of:)` — donate catalog-specific terms **[NEW media types]**
- `INVocabularyStringType.mediaPlaylistTitle`, `.musicArtistName`, `.audiobookTitle`, `.audiobookAuthorName`, `.showTitle` **[NEW]**

Intents extension setup:
- Info.plist `NSExtension > NSExtensionAttributes > IntentsSupported`: add `INPlayMediaIntent`, `INAddMediaIntent`, etc.
- Info.plist media types: `INPlayMediaIntent > SupportedMediaCategories` **[NEW]**

## Code Highlights

Resolve method for play intent:
```swift
func resolveMediaItems(for intent: INPlayMediaIntent,
                       with completion: @escaping ([INPlayMediaMediaItemResolutionResult]) -> Void) {
    guard let search = intent.mediaSearch else {
        completion([.unsupported(forReason: .notExplicitlyBeingSought)])
        return
    }
    
    let query = search.mediaName ?? ""
    // Case-insensitive, punctuation-stripped fuzzy search
    let items = catalog.search(query: query, type: search.mediaType)
    
    if let item = items.first {
        let mediaItem = INMediaItem(identifier: item.id,
                                   title: item.title,
                                   type: .music,
                                   artwork: nil,
                                   artist: item.artist)
        completion([.success(with: mediaItem)])
    } else {
        completion([.unsupported(forReason: .notExplicitlyBeingSought)])
    }
}
```

Handle method (background app launch on iOS):
```swift
func handle(intent: INPlayMediaIntent, completion: @escaping (INPlayMediaIntentResponse) -> Void) {
    completion(INPlayMediaIntentResponse(code: .handleInApp, userActivity: nil))
}
```

App delegate receiving background launch:
```swift
func application(_ application: UIApplication,
                 handle intent: INIntent,
                 completionHandler: @escaping (INIntentResponse) -> Void) {
    guard let playIntent = intent as? INPlayMediaIntent,
          let mediaItem = playIntent.mediaItems?.first else {
        completionHandler(INPlayMediaIntentResponse(code: .failure, userActivity: nil))
        return
    }
    player.play(identifier: mediaItem.identifier)
    completionHandler(INPlayMediaIntentResponse(code: .success, userActivity: nil))
}
```

Handle add intent in extension (no app launch needed):
```swift
func handle(intent: INAddMediaIntent, completion: @escaping (INAddMediaIntentResponse) -> Void) {
    guard let item = intent.mediaItems?.first,
          let destination = intent.mediaDestination else {
        completion(INAddMediaIntentResponse(code: .failure, userActivity: nil))
        return
    }
    library.add(identifier: item.identifier!, to: destination)
    completion(INAddMediaIntentResponse(code: .success, userActivity: nil))
}
```

## Takeaways
- The resolve step is the most important part of SiriKit Media integration: implement flexible, case-insensitive, punctuation-stripping, suffix-aware search to account for the gap between what users say and what catalog titles actually contain.
- Skip the confirm step entirely — adding any dialogue prompt before playback significantly reduces the likelihood users will complete the interaction.
- For `INAddMediaIntent` and `INUpdateMediaAffinityIntent`, handle the operation in the extension itself without a background app launch; only `INPlayMediaIntent` needs to launch the app.
- Migrating from a Shortcuts (iOS 12) media implementation to full SiriKit requires adding only the `resolveMediaItems` method — the handle method and background app launch code are identical.

---
_Source: WWDC19 Session 207 page (transcript, chapter summaries, and resource links)._
