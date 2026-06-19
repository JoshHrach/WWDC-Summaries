# Error Handling Best Practices for HTTP Live Streaming
**WWDC17 · Session 514** · [Watch](https://developer.apple.com/videos/play/wwdc2017/514/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, tvOS 11

## Overview
This session addresses one of the most common HLS developer questions: what is the correct behavior when something goes wrong during streaming? It covers both the server side (which HTTP error codes to return for which failure conditions) and the client/app side (which AVFoundation APIs to observe, how to interpret error codes, and how to communicate errors to users).

The session distinguishes between temporary failures (where the server should continue serving playlists with `EXT-X-GAP`-tagged segments and return `404` for stale playlists) and permanent failures (where `410 Gone` should be returned and AVPlayer will not retry). On the app side, `AVPlayer.status`, `AVPlayerItem.status`, `AVPlayerItem.error`, and `AVPlayerItem.errorLog` form the complete picture of what went wrong, and specific notification names allow apps to react in real time rather than polling.

## Key Topics

- **Server error code mapping** — use HTTP status codes to communicate failure cause precisely:
  - `401` — content is protected and client lacks authentication
  - `403` — client is not authorized for this content
  - `404` — temporary resource unavailability (preferred over returning stale content)
  - `410` — permanent resource unavailability (no retries by AVPlayer)
  - `500` — unexpected server condition
  - `501` — unsupported feature (e.g., BYTE-RANGE not supported)
  - `502` — invalid response from upstream gateway
  - `503` — server unavailable (maintenance, overloaded)
  - `504` — gateway timeout

- **EXT-X-GAP tag for temporary failures** — new in iOS 11; mark failed segments in the media playlist rather than returning error HTTP codes; tells AVPlayer the failure is temporary and it should try switching variants rather than treating playback as unrecoverable. For `404` and `503` cases on live streams: use `EXT-X-GAP`. For backward compatibility, also send `404` on actual segment requests for GAP-tagged segments.

- **Stale live playlist handling** — if the live playlist cannot be updated by its target duration: return `404` for the playlist request (faster notification than returning the old playlist and letting AVPlayer detect staleness); also alerts new joining clients immediately.

- **Failover with redundant variants** — host same-bitrate variants on different servers in the master playlist; AVPlayer tries backup alternates before switching down; server can trigger explicit failover by returning `404` on a playlist request.

- **AVPlayer retry behavior** — for temporary errors AVPlayer retries and switches variants; after reasonable time it may switch back up to the previously failed variant if network allows; for `410` (permanent), no retry is attempted, only variant switching.

- **Client-side error detection** — observe:
  - `AVPlayer.status` → `.failed`
  - `AVPlayerItem.status` → `.failed`
  - `AVPlayerItem.error` — root cause of failure
  - `AVPlayerItemFailedToPlayToEndTimeNotification` + `AVPlayerItemFailedToPlayToEndTimeErrorKey`
  - `AVPlayerItemNewErrorLogEntryNotification` — immediate notification of new error log events
  - `AVPlayerItem.errorLog` — full snapshot of all errors during session

- **Error categories and AVFoundation error codes**:
  - Network errors (4xx, 5xx, TCP/IP, DNS) → `AVErrorContentIsUnavailable` or `AVErrorNoLongerPlayable`
  - `AVErrorContentIsUnavailable` — content was never playable (auth failure, 401/403)
  - `AVErrorNoLongerPlayable` — was playable, then became unplayable (multiple errors accumulated)
  - `AVErrorFailedToParse` — malformed playlist, key, or session data
  - `AVErrorContentNotUpdated` — live playlist not updated in time

- **Session data errors** — not fatal but not silently ignored; app should still handle them.

## APIs & Frameworks

**AVFoundation**
- `AVPlayer.status` — observe for `.failed`
- `AVPlayerItem.status` — observe for `.failed`
- `AVPlayerItem.error` — `NSError` describing the failure; check `userInfo` for nested underlying errors
- `AVPlayerItem.errorLog` — `AVPlayerItemErrorLog` with all error events during session
- `AVPlayerItemErrorLogEvent` — individual error log entry; contains `errorStatusCode`, `errorDomain`, `errorComment`, `serverAddress`, `playbackSessionID`
- `AVPlayerItemFailedToPlayToEndTimeNotification` — posted when item cannot play to completion
- `AVPlayerItemFailedToPlayToEndTimeErrorKey` — key to extract error from notification userInfo
- `AVPlayerItemNewErrorLogEntryNotification` — posted each time a new error log event is added
- `AVErrorContentIsUnavailable` — error code for content never playable
- `AVErrorNoLongerPlayable` — error code for content that became unplayable
- `AVErrorFailedToParse` — error code for format/parse failures
- `AVErrorContentNotUpdated` — error code for stale live playlist

**HLS Playlist Tags (server-side)**
- `EXT-X-GAP` **[NEW iOS 11]** — marks segments as unavailable due to temporary failure; tells AVPlayer to switch variants rather than stall

## Code Highlights

Observing errors:
```swift
// KVO on player and item status
player.addObserver(self, forKeyPath: #keyPath(AVPlayer.status), options: .new, context: nil)
playerItem.addObserver(self, forKeyPath: #keyPath(AVPlayerItem.status), options: .new, context: nil)

// Notification for failed-to-play-to-end
NotificationCenter.default.addObserver(
    self, selector: #selector(itemFailedToPlayToEnd(_:)),
    name: .AVPlayerItemFailedToPlayToEndTime, object: playerItem)

// In KVO handler:
override func observeValue(forKeyPath keyPath: String?, ...) {
    if keyPath == #keyPath(AVPlayerItem.status),
       playerItem.status == .failed {
        print("Item failed: \(playerItem.error?.localizedDescription ?? "unknown")")
        // Display meaningful error UI to user
    }
}

@objc func itemFailedToPlayToEnd(_ notification: Notification) {
    let error = notification.userInfo?[AVPlayerItemFailedToPlayToEndTimeErrorKey] as? Error
    print("Failed to play to end: \(error?.localizedDescription ?? "unknown")")
}
```

## Takeaways

- Use precise HTTP status codes — `401`/`403` for auth issues, `404`/`503` for temporary unavailability, `410` for permanent removal — so AVPlayer can make the right retry/switch decision.
- Use `EXT-X-GAP` in the live playlist for temporary encoder/packager outages rather than dropping segments or returning errors; it lets AVPlayer switch variants smoothly and avoid stalls.
- Return `404` for a stale live playlist rather than the old content — it signals the problem faster and immediately notifies newly joining players.
- On the app side, always observe `AVPlayerItem.status`, listen to `AVPlayerItemFailedToPlayToEndTimeNotification`, and display a meaningful error message to users; never silently swallow AVPlayer failures.

---
_Source: WWDC17 Session 514 page (abstract, transcript, and resource links)._
