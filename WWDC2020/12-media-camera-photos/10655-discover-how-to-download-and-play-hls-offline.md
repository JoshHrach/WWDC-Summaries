# Discover How to Download and Play HLS Offline
**WWDC20 · Session 10655** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10655/)

_Platforms:_ iOS 14, iPadOS 14 (AVFoundation offline HLS)

## Overview
This session covers the complete lifecycle of downloading HLS content for offline playback using AVFoundation. The capability has existed since 2016 but iOS 14 adds new quality-selection options. The session covers two download task variants, delegate-based progress monitoring, background download support, FairPlay Streaming content protection for offline keys, and storage management best practices.

Offline HLS is appropriate when users explicitly request downloads — for airplane flights, constrained cellular data plans, or any scenario where offline access is desired. The same HLS streams used for online streaming can be used for downloads, and FairPlay Streaming keys can be persisted on-device with server-controlled expiration dates and rental dual-expiry support.

## Key Topics

### Download Task Types
Two variants of download task are available, both created via `AVAssetDownloadURLSession`:

1. **`AVAssetDownloadTask`** — downloads one combination of video, audio, and subtitle renditions using automatic media selection (device region/locale determines which renditions are chosen).
2. **`AVAggregateAssetDownloadTask`** — developer explicitly specifies which audio and subtitle renditions to download; supports multiple language combinations.

Both task types are backed by `URLSession` infrastructure: downloads are scheduled according to system resource availability, automatically retried on timeout, and continue running in the background.

### Monitoring Progress
- Implement `AVAssetDownloadDelegate` (conforms to `URLSessionTaskDelegate`).
- `didLoad(_:totalTimeRangesLoaded:timeRangeExpectedToLoad:)` — provides downloaded time ranges (not bytes) for progress calculation.
- `didCompleteWithError:` — signals download completion or failure.
- For `AVAggregateAssetDownloadTask`, an additional `didCompleteFor:` callback fires per media selection. Audio renditions fire twice: once for stereo, once for multi-channel.
- Recommended weight distribution for aggregate progress: ~70% to video, remainder split between audio and subtitle tracks.

### Background Downloads and Task Restoration
- Downloads continue when the app is backgrounded.
- On relaunch, restore existing tasks by re-creating `URLSession` with the same configuration identifier and calling `getAllTasks(completionHandler:)`.

### Playback of Downloaded Content
- While download is in progress: reuse the same `AVURLAsset` from the download task for `AVPlayerItem` — AVFoundation shares resources between download and playback, avoiding duplicate fetches.
- After download completes: save the file URL from the delegate, recreate `AVURLAsset(url: fileURL)` for later playback.
- `AVAssetCache` (from `AVURLAsset.assetCache`) reveals which renditions are playable offline.

### FairPlay Streaming for Offline Content
- Create an offline key (persistent content key) during download via `AVContentKeySession`.
- Store the offline key locally; replay it during offline playback when the key server is unreachable.
- Key expiration is server-controlled; if a key expires during playback, the session continues to completion rather than stopping abruptly.
- Create a new offline key before expiration.
- Delete offline keys via `invalidatePersistableContentKey(_:options:completionHandler:)` or `invalidateAllPersistableContentKeys(forApp:options:completionHandler:)` (since iOS 12).
- Rental dual-expiry: FairPlay supports two expiration dates (e.g., rent for 30 days, but once started must finish within 48 hours).

### Quality and Rendition Selection Options (iOS 14)
- `AVAssetDownloadTaskMinimumRequiredMediaBitrateKey` — minimum bitrate for video rendition selection.
- `AVAssetDownloadTaskMinimumRequiredPresentationSizeKey` **[NEW in iOS 14]** — minimum presentation size.
- `AVAssetDownloadTaskPrefersHDRKey` **[NEW in iOS 14]** — whether to prefer HDR renditions (default: true if available).
- `AVAssetDownloadTaskPrefersMultichannelKey` — whether to download multi-channel audio in addition to stereo (default: true).

### Storage Management
- Let the OS manage download storage via `AVAssetDownloadStorageManager`.
- Create an `AVMutableAssetDownloadStorageManagementPolicy` with a priority and expiration date.
- The OS purges assets by expiration date, then priority, when storage is low — including during software updates.
- Users can delete content through Settings. The asset title and artwork provided at download creation appear in Settings.

## APIs & Frameworks

### AVFoundation
- `AVAssetDownloadURLSession` — custom `URLSession` subclass for HLS downloads; init with `URLSessionConfiguration.background(withIdentifier:)` and `AVAssetDownloadDelegate`
- `AVAssetDownloadTask` — single-rendition download task; created via `AVAssetDownloadURLSession.makeAssetDownloadTask(asset:assetTitle:assetArtworkData:options:)`
- `AVAggregateAssetDownloadTask` — multi-rendition download task; created via `AVAssetDownloadURLSession.aggregateAssetDownloadTask(with:mediaSelections:assetTitle:assetArtworkData:options:)`
- `AVAssetDownloadDelegate` protocol:
  - `urlSession(_:assetDownloadTask:didLoad:totalTimeRangesLoaded:timeRangeExpectedToLoad:)`
  - `urlSession(_:aggregateAssetDownloadTask:didLoad:totalTimeRangesLoaded:timeRangeExpectedToLoad:for:)`
  - `urlSession(_:aggregateAssetDownloadTask:didCompleteFor:)`
  - `urlSession(_:assetDownloadTask:didFinishDownloadingTo:)`
  - `urlSession(_:aggregateAssetDownloadTask:willDownloadTo:)`
  - `urlSession(_:task:didCompleteWithError:)`
- `AVURLAsset` — media asset; `assetCache: AVAssetCache?`
- `AVAssetCache` — `isPlayableOffline: Bool`, `mediaSelectionOptions(in:) -> [AVMediaSelectionOption]`
- `AVMediaSelection` / `AVMutableMediaSelection` — represents a set of selected renditions
- `AVAssetDownloadTaskMinimumRequiredMediaBitrateKey`
- `AVAssetDownloadTaskMinimumRequiredPresentationSizeKey` **[NEW in iOS 14]**
- `AVAssetDownloadTaskPrefersHDRKey` **[NEW in iOS 14]**
- `AVAssetDownloadTaskPrefersMultichannelKey`
- `AVAssetDownloadStorageManager` — `shared()`, `setStorageManagementPolicy(_:forURL:)`
- `AVMutableAssetDownloadStorageManagementPolicy` — `expirationDate`, `priority` (`.important`, `.default`)
- `AVContentKeySession` — offline key management:
  - `invalidatePersistableContentKey(_:options:completionHandler:)`
  - `invalidateAllPersistableContentKeys(forApp:options:completionHandler:)`

### Foundation / URLSession
- `URLSessionConfiguration.background(withIdentifier:)` — background session configuration
- `URLSession.getAllTasks(completionHandler:)` — restore tasks on app relaunch

## Code Highlights

Creating an `AVAssetDownloadTask` at 2 Mbps:
```swift
let hlsAsset = AVURLAsset(url: assetURL)
let config = URLSessionConfiguration.background(withIdentifier: "myDownloadSession")
let downloadSession = AVAssetDownloadURLSession(configuration: config,
    assetDownloadDelegate: self, delegateQueue: .main)
let task = downloadSession.makeAssetDownloadTask(asset: hlsAsset,
    assetTitle: "My Movie", assetArtworkData: artwork,
    options: [AVAssetDownloadTaskMinimumRequiredMediaBitrateKey: 2_000_000])!
task.resume()
```

Configuring storage management:
```swift
let manager = AVAssetDownloadStorageManager.shared()
let policy = AVMutableAssetDownloadStorageManagementPolicy()
policy.expirationDate = myExpiryDate
policy.priority = .important
manager.setStorageManagementPolicy(policy, forURL: downloadedAssetURL)
```

## Takeaways
- Use `AVAssetDownloadTask` for automatic rendition selection; use `AVAggregateAssetDownloadTask` when users need to choose specific language tracks.
- Reuse the same `AVURLAsset` from the download task for playback to allow AVFoundation to share resources between download and playback.
- Protect offline content with FairPlay Streaming persistent content keys; use `invalidatePersistableContentKey` when users delete downloads.
- Opt into OS-managed storage via `AVAssetDownloadStorageManager` to allow automatic cleanup during low-storage conditions.
- iOS 14 adds `AVAssetDownloadTaskMinimumRequiredPresentationSizeKey` and `AVAssetDownloadTaskPrefersHDRKey` for finer download quality control.

---
_Source: WWDC20 Session 10655 page (abstract, transcript, code samples, and resource links)._
