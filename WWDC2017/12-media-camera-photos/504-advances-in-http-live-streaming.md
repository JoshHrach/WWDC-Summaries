# Advances in HTTP Live Streaming
**WWDC17 · Session 504** · [Watch](https://developer.apple.com/videos/play/wwdc2017/504/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, tvOS 11

## Overview
This session covers a broad set of new features and API improvements landing in HLS for iOS 11 and related platforms. The headline announcements are support for HEVC (H.265) video and IMSC1 subtitles, each requiring fragmented MPEG-4 packaging. The session also confirms that the HLS spec has been approved as an IETF Internet Draft and is on its way to becoming a numbered RFC, giving the industry a stable reference point while Apple continues extending HLS on a new draft baseline.

Two streaming-infrastructure improvements are introduced: the `EXT-X-GAP` tag for indicating encoder outages in live streams without dropping the session, and playlist meta-variables (PHP-style `{$variable}` syntax) that allow late-binding of values like auth tokens across master and media playlists. A live multi-stream synchronization feature shows how multiple `AVPlayer` instances can be kept in sync using date-time tags and `AVPlayer.setRate(_:time:atHostTime:)`.

The second half of the session focuses on content-key management: the new `AVContentKeySession` API decouples FairPlay Streaming key loading from asset loading, enabling key preloading, load-balanced key renewal for large live audiences, simpler offline key workflows, and a new dual-expiry-window model that enables iTunes-style rental semantics on persistent keys.

## Key Topics

- **HLS RFC publication** — HLS spec draft -23 approved by IETF; will become an RFC; new internet draft will extend it.
- **HEVC in HLS** — ~40% encoding efficiency gain over H.264; hardware decode on A9+ devices and latest Macs; software decode on all iOS 11 / tvOS 11 / High Sierra devices; must use fragmented MPEG-4 containers; `CODECS` attribute required in master playlist; can be mixed with H.264 variants.
- **IMSC1 subtitles** — TTML-derived delivery format with richer styling than WebVTT; packaged as XML inside MPEG-4 fragments; text profile only (`stpp.TTML.im1t` codec tag); can coexist with VTT in one master playlist for backward compatibility.
- **EXT-X-GAP tag** — lets a packager mark dummy segments during encoder outages; players can seek another variant without the gap, or play silence until media recovers.
- **Playlist meta-variables** — `{$VAR}` substitution defined via a new tag; variables can be imported from a master playlist into media playlists for late-binding of auth tokens and paths.
- **Synchronized multi-stream playback** — multiple independent `AVPlayer` instances locked to a shared clock via `EXT-X-PROGRAM-DATE-TIME` and `AVPlayer.setRate(_:time:atHostTime:)`; sample app `SyncStartTV` provided.
- **Resolution cap** — `AVPlayerItem.preferredMaximumResolution` caps bitrate ladder climbing for thumbnails and multi-stream layouts.
- **Offline HLS storage management** — iOS 11 Settings shows per-app offline asset disk usage; new `AVAssetDownloadStorageManager` lets apps set `AVAssetDownloadStorageManagementPolicy` with `priority` (`.important` / `.default`) and `expirationDate`.
- **Aggregate asset download** — `AVAggregateAssetDownloadTask` batches multiple media-selection (e.g., audio language) downloads in one task with unified progress.
- **AVContentKeySession** — new class for decoupling FairPlay Streaming key loading from asset loading; supports key preloading, demand-driven loading, and offline persistent keys.
- **Dual-expiry-window for persistent keys** — two configurable windows: storage expiry (from key creation) and playback expiry (from first play); enables rental model on offline assets.

## APIs & Frameworks

**AVFoundation**
- `AVContentKeySession` **[NEW]** — manages content decryption keys independently of assets
- `AVContentKeyRequest` **[NEW]** — replaces `AVAssetResourceLoadingRequest` for FairPlay key loading
- `AVPersistableContentKeyRequest` **[NEW]** — subclass of `AVContentKeyRequest` for offline persistent keys
- `AVContentKeySessionDelegate` **[NEW]** — delegate protocol replacing `AVAssetResourceLoaderDelegate` for key delivery
- `AVContentKeySession.processContentKeyRequest(withIdentifier:initializationData:options:)` **[NEW]**
- `AVContentKeyRequest.makeStreamingContentKeyRequestData(forApp:contentIdentifier:options:completionHandler:)` **[NEW]**
- `AVContentKeyRequest.respond(with:)` **[NEW]**
- `AVPersistableContentKeyRequest.persistableContentKey(fromKeyVendorResponse:options:)` **[NEW]**
- `AVAssetDownloadStorageManager` **[NEW]** — singleton for managing offline asset lifecycle
- `AVAssetDownloadStorageManagementPolicy` **[NEW]** — policy with `priority` and `expirationDate`
- `AVMutableAssetDownloadStorageManagementPolicy` **[NEW]** — mutable version of the policy
- `AVAssetDownloadStorageManagementPolicy.Priority` **[NEW]** — `.important`, `.default`
- `AVAggregateAssetDownloadTask` **[NEW]** — batches multiple media-selection downloads
- `AVPlayer.setRate(_:time:atHostTime:)` — used for synchronized multi-stream start
- `AVPlayerItem.preferredMaximumResolution` **[NEW]** — caps resolution for thumbnail/multi-stream use

**HLS Playlist Tags (spec-level)**
- `EXT-X-GAP` **[NEW]** — marks segments with no media data during encoder outage
- `EXT-X-DEFINE` **[NEW]** — declares playlist meta-variables with `NAME`, `VALUE`, or `IMPORT` attributes
- `{$VARIABLE_NAME}` substitution syntax **[NEW]** — used in URL strings within playlists
- `EXT-X-PROGRAM-DATE-TIME` — existing tag; now used for multi-stream sync
- `CODECS` attribute — extended with HEVC codec string format and IMSC1 `stpp.TTML.im1t`

**Supported Codecs / Formats**
- HEVC (H.265) in fragmented MPEG-4 **[NEW]**
- IMSC1 text profile subtitles in MPEG-4 fragments **[NEW]**
- Common Encryption (CBCS mode) — works identically for HEVC as for H.264

## Code Highlights

Initiating key preloading with AVContentKeySession:
```swift
let contentKeySession = AVContentKeySession(keySystem: .fairPlayStreaming)
contentKeySession.setDelegate(self, queue: DispatchQueue.main)
contentKeySession.processContentKeyRequest(withIdentifier: assetID,
                                           initializationData: nil,
                                           options: nil)
```

Capping stream resolution for multi-stream layouts:
```swift
playerItem.preferredMaximumResolution = CGSize(width: 1920, height: 1080)
```

Handling a persistent key request:
```swift
func contentKeySession(_ session: AVContentKeySession,
                       didProvide keyRequest: AVContentKeyRequest) {
    keyRequest.respondByRequestingPersistableContentKeyRequest()
}

func contentKeySession(_ session: AVContentKeySession,
                       didProvide keyRequest: AVPersistableContentKeyRequest) {
    // create SPC, get CKC, persist key
    let persistedKeyData = try keyRequest.persistableContentKey(
        fromKeyVendorResponse: ckc, options: nil)
    // store persistedKeyData; respond:
    let response = AVContentKeyResponse(fairPlayStreamingKeyResponseData: persistedKeyData)
    keyRequest.processContentKeyResponse(response)
}
```

## Takeaways

- Adopt HEVC with fragmented MPEG-4 immediately for ~40% bandwidth savings; mix H.264 and HEVC variants for backward compatibility.
- Switch FairPlay key handling from `AVAssetResourceLoader` to `AVContentKeySession` for preloading, load-balanced renewal, and simpler offline key workflows.
- Use `AVAssetDownloadStorageManager` and `AVAggregateAssetDownloadTask` to manage offline asset disk space and multi-language downloads more robustly.
- The new `EXT-X-GAP` tag and playlist meta-variables give server-side teams better tools for resilient live streams and shared auth-token injection.

---
_Source: WWDC17 Session 504 page (abstract, transcript, and resource links)._
