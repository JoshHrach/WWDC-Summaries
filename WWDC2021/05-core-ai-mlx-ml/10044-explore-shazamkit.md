# Explore ShazamKit
**WWDC21 · Session 10044** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10044/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
ShazamKit brings Shazam's precise audio recognition technology directly into third-party apps. Rather than analyzing audio classes or classifiers, ShazamKit matches audio against exact signatures — lossy, non-invertible representations of audio — to deliver near-instantaneous matches against either the Shazam catalog (cloud-based, covering the world's music) or custom catalogs built from arbitrary audio (on-device).

The framework is divided into three components: Shazam catalog recognition, custom catalog recognition, and library management. Shazam catalog recognition requires enabling the ShazamKit App Service in the Developer portal; custom catalog matching runs entirely on-device with no such requirement.

Use cases include identifying background music in a video, syncing education app content to streaming lessons, enabling shoppable second-screen TV experiences, and driving dynamic visual effects based on detected genre metadata.

## Key Topics

**Audio Signatures and Catalogs**
ShazamKit never transmits raw audio. It creates a "signature" — a lossy, non-invertible representation at least an order of magnitude smaller than the source audio — and matches that against a catalog of reference signatures. The Shazam catalog lives in the cloud; custom catalogs match on-device.

**SHSession and Matching Flow**
The central API is `SHSession`. For streaming audio (e.g., microphone input), use `matchStreamingBuffer(_:at:)`. For pre-existing audio buffers, use `SHSignatureGenerator` to build a signature, then call `session.match(_:)`. Results arrive through the `SHSessionDelegate`.

**SHMediaItem and SHMatch**
A successful match returns an `SHMatch` containing an array of `SHMatchedMediaItem` objects (a subclass of `SHMediaItem`). Properties include `title`, `artist`, `artworkURL`, `appleMusicURL`, `matchOffset`, and a strongly typed `songs` property bridging to MusicKit's `Song` objects.

**Shazam Library Management**
`SHMediaLibrary.default` allows writing matched items to a user's Shazam library, which is end-to-end encrypted, requires two-factor authentication, syncs across devices, and attributes saves to the source app. No special permission is needed, but user opt-in is strongly recommended.

**Best Practices**
Use `matchStreamingBuffer` for real-time audio; stop microphone recording as soon as a result is obtained; always obtain user consent before writing to the Shazam library.

## APIs & Frameworks

- **ShazamKit** **[NEW]** — top-level framework
- `SHSession` **[NEW]** — manages matching against Shazam or custom catalogs
  - `init()` — matches against Shazam catalog
  - `func match(_ signature: SHSignature)` **[NEW]**
  - `func matchStreamingBuffer(_ buffer: AVAudioPCMBuffer, at time: AVAudioTime?)` **[NEW]**
  - `var delegate: SHSessionDelegate?`
- `SHSessionDelegate` **[NEW]** — protocol
  - `func session(_ session: SHSession, didFind match: SHMatch)`
  - `func session(_ session: SHSession, didNotFindMatchFor signature: SHSignature, error: Error?)`
- `SHSignatureGenerator` **[NEW]**
  - `func append(_ buffer: AVAudioPCMBuffer, at time: AVAudioTime?) throws`
  - `func signature() -> SHSignature`
- `SHSignature` **[NEW]** — opaque, non-invertible audio fingerprint
- `SHMatch` **[NEW]**
  - `var mediaItems: [SHMatchedMediaItem]`
- `SHMatchedMediaItem` **[NEW]** (subclass of `SHMediaItem`)
  - `var matchOffset: TimeInterval`
  - `var frequencySkew: Float`
- `SHMediaItem` **[NEW]**
  - `var title: String?`
  - `var artist: String?`
  - `var artworkURL: URL?`
  - `var appleMusicURL: URL?`
  - `var songs: [MusicKit.Song]` (Swift, bridges to MusicKit) **[NEW]**
- `SHMediaLibrary` **[NEW]**
  - `static var default: SHMediaLibrary`
  - `func add(_ mediaItems: [SHMediaItem], completionHandler: ((Error?) -> Void)?)`
- `SHCatalog` **[NEW]** — base class for catalogs
- `SHCustomCatalog` **[NEW]** — on-device custom catalog

## Code Highlights

Creating a session and matching a pre-existing audio buffer:
```swift
let session = SHSession()
session.delegate = self

let signatureGenerator = SHSignatureGenerator()
try signatureGenerator.append(buffer, at: nil)
let signature = signatureGenerator.signature()
session.match(signature)
```

Receiving a match via the delegate:
```swift
extension SongResultViewController: SHSessionDelegate {
    public func session(_ session: SHSession, didFind match: SHMatch) {
        guard let matchedMediaItem = match.mediaItems.first else { return }
        DispatchQueue.main.async {
            self.songView.titleLabel.text = matchedMediaItem.title
            self.songView.artistLabel.text = matchedMediaItem.artist
        }
    }
}
```

Adding a matched song to the user's Shazam library:
```swift
guard let matchedMediaItem = match.mediaItems.first else { return }
SHMediaLibrary.default.add([matchedMediaItem]) { error in
    if error != nil { /* handle error */ }
}
```

## Takeaways

- ShazamKit is a brand-new framework in iOS 15 / macOS 12 that exposes Shazam's audio fingerprinting both for the global Shazam catalog and for on-device custom catalogs.
- Audio is never transmitted as raw PCM; privacy is preserved via non-invertible signatures.
- `SHSession` is the main entry point; use `matchStreamingBuffer` for microphone input and `SHSignatureGenerator` + `match(_:)` for file-based audio.
- Library writes are attributed to the calling app and should always be preceded by explicit user consent.

---
_Source: WWDC21 Session 10044 page (abstract, chapter summaries, code samples, and resource links)._
