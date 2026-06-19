# Integrate Your Media App with HomePod
**WWDC23 · Session 10104** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10104/)

_Platforms:_ iOS 17, iPadOS 17

## Overview
Starting with iOS 17, any app that already implements SiriKit Media Intents can automatically play content on HomePod via AirPlay when a user asks Siri on the speaker. No code changes are required for existing implementations — HomePod processes the voice request, sends a SiriKit intent to the user's primary iPhone over Wi-Fi, and the iPhone AirPlays audio back to the speaker.

This session covers the full integration path: how intent routing works, which intents are supported, how to provide specific error responses to improve Siri's feedback, how to correctly configure AVAudioSession for background AirPlay playback, and how to support affinity requests (like/add) and personal app vocabulary. It also covers HomePod-specific requirements such as voice recognition setup and primary device selection.

## Key Topics

### How HomePod Integration Works
- HomePod processes the request and sends a SiriKit Media intent to the user's primary iPhone over Wi-Fi.
- Devices must be on the same Wi-Fi network; physical proximity is not required.
- Any app supporting `INPlayMediaIntent` today works on HomePod with no additional changes.
- Siri uses voice recognition to route the request to the device of the recognized Home user.

### Supported Intents
- `INPlayMediaIntent` — primary playback intent; handles music, podcasts, audiobooks, radio, meditation, etc.
- `INAddMediaIntent` — adds content to a queue or library.
- `INUpdateMediaAffinityIntent` — like/dislike/favorite content (works even for AirPlayed content not started via Siri).
- Find/search requests on HomePod result in playback (not a pure search result).
- `playbackQueueLocation` field: `.next` for add-to-queue requests.

### Intent Resolution and Error Handling
- Return specific failure reasons instead of generic `.unsupported`:
  - `INPlayMediaIntentResponseCode.failureRequiringAppLaunch` — missing login.
  - `INPlayMediaIntentResponseCode.failureUnknownMediaType` — unsupported media type (improves Siri's spoken response).
- Siri abandons callbacks after 10 seconds; respond as fast as possible.

### Handle Responses: Background vs. Foreground
- `handleInApp` — system starts app in background and plays audio; best for HomePod (phone may be in another room).
- `continueInApp` — system starts app in foreground; requires device to be unlocked.
- Background audio playback is only possible when the app is in a Siri request context.

### AVAudioSession Configuration
- Set category to `.playback` before activating the session for background audio to continue when the app goes into the background.
- `AVAudioSession.Mode.spokenAudio` — pauses playback on interruptions (ideal for podcasts/audiobooks) rather than ducking.
- `AVAudioSession.RouteSharingPolicy.longFormAudio` — appropriate for AirPlay long-form audio.
- Incorrect category setup causes playback to stop when the app backgrounds.

### AirPlay Integration Requirements
- Use `MPNowPlayingInfoCenter` to provide now-playing metadata.
- Use `MPRemoteCommandCenter` to handle remote commands (play, pause, skip).
- Use buffered playback APIs to eliminate resumption delays caused by AirPlay buffering.

### HomePod-Specific Setup Requirements
- User must be registered in the Home and have "Recognize My Voice" enabled in the Home app.
- "Personal Requests" setting is not required.
- Primary iPhone: the device set up for location sharing under Apple ID / Find My settings.
- First Siri request from HomePod prompts the user to grant Siri access to the app on their iPhone.

### Personal App Vocabulary
- Use `INVocabulary` (personal app vocabulary) to inform the system of user-specific entities: playlists, audiobooks, podcast names, favorite artists.
- Improves Siri's ability to resolve entity names in requests.

## APIs & Frameworks
- `SiriKit` framework — core intent handling
- `INPlayMediaIntent` — play media via Siri; works on HomePod **[NEW HomePod routing, iOS 17]**
- `INAddMediaIntent` — add content to queue/library
- `INUpdateMediaAffinityIntent` — like/dislike/favorite
- `INMediaItemType` — enum for media type: `.song`, `.album`, `.artist`, `.genre`, `.podcast`, `.audioBook`, `.radioStation`, etc.
- `INPlayMediaMediaItemResolutionResult` — resolution result for resolving media entities
- `INPlayMediaIntentResponseCode.failureRequiringAppLaunch` — specific failure code for login-required scenarios
- `INPlayMediaIntentResponseCode.failureUnknownMediaType` — specific failure code for unsupported media types
- `INPlayMediaIntentResponse.handleInApp` — background playback response
- `INPlayMediaIntentResponse.continueInApp` — foreground playback response
- `INMediaSortOrder` — sort order for results (e.g., `.popular`)
- `INPlaybackQueueLocation.next` — add to next in queue
- `AVAudioSession` — audio session management
- `AVAudioSession.Category.playback` — required for background audio
- `AVAudioSession.Mode.spokenAudio` — pause-on-interruption mode for podcasts/audiobooks
- `AVAudioSession.RouteSharingPolicy.longFormAudio` — AirPlay long-form audio routing
- `MPNowPlayingInfoCenter` — provides now-playing metadata to system
- `MPRemoteCommandCenter` — handles remote playback commands
- `INVocabulary` — personal app vocabulary for user-specific entity names
- AirPlay — audio streaming protocol; used to route app audio to HomePod

## Code Highlights

Proper AVAudioSession configuration for background AirPlay:
```swift
let audioSession = AVAudioSession.sharedInstance()
try audioSession.setCategory(
    .playback,
    mode: .spokenAudio,
    routeSharingPolicy: .longFormAudio
)
try audioSession.setActive(true)
```

Returning a specific error response when a media type is unsupported:
```swift
func resolveMediaItems(for intent: INPlayMediaIntent) async -> [INPlayMediaMediaItemResolutionResult] {
    guard intent.mediaType != .musicGenre else {
        return [.failure(with: .failureUnknownMediaType)]
    }
    // ... search and return results
}
```

## Takeaways
- Existing SiriKit Media Intent implementations work on HomePod automatically in iOS 17 — no code changes required.
- Set `AVAudioSession.Category.playback` before session activation for audio to continue in the background when the app is launched by a Siri request on HomePod.
- Return specific `INPlayMediaIntentResponseCode` failure cases (e.g., `.failureUnknownMediaType`) instead of generic `.unsupported` to enable Siri to give users actionable feedback.
- Users must have "Recognize My Voice" enabled in the Home app; personal vocabulary improves entity resolution for app-specific content names.

---
_Source: WWDC23 Session 10104 page (abstract, chapter summaries, code samples, and resource links)._
