# What's New in Streaming Audio for Apple Watch
**WWDC20 · Session 10636** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10636/)

_Platforms:_ watchOS 7

## Overview
watchOS 7 extended Apple Watch audio streaming capabilities in two important ways: support for the xHE-AAC codec in AVPlayer for more efficient audio delivery, and FairPlay Streaming support via `AVContentKeySession` for protected content. These additions allow watch audio apps to deliver higher-quality audio at lower bitrates and expand their libraries to include DRM-protected catalog content.

The session also covers performance best practices unique to streaming on watchOS, where the device is constrained by battery, processing power, and highly variable network conditions due to user mobility.

## Key Topics

### xHE-AAC Codec Support
Extended High-Efficiency AAC (xHE-AAC) is now supported in `AVPlayer` on watchOS 7. It delivers equivalent audio quality at lower bitrates (or higher quality at equivalent bitrates) compared to AAC-LC, HE-AAC, and HE-AACv2. For watch specifically — where network conditions are variable and bandwidth is at a premium — xHE-AAC is the recommended primary codec.

For interoperability, Apple recommends including AAC-LC, HE-AAC, or HE-AACv2 fallback variants in the HLS manifest alongside xHE-AAC, allowing `AVPlayer` to select the best available variant automatically.

### FairPlay Streaming on watchOS
FairPlay Streaming (FPS), which decrypts and plays back encrypted media using a key server workflow, is now available on watchOS 7 via `AVContentKeySession`. The watch app acts as a liaison between AVFoundation and the content key server. `AVContentKeySession` is decoupled from the lifecycle of a specific asset, giving apps fine-grained control over key pre-fetching and lifecycle management. Apps can initiate key loading proactively at any time — not just in response to playback demands — to reduce startup latency.

### Best Practices for watchOS Streaming
- Avoid unnecessary HTTP redirects on any playback resources (manifests, keys, segments); each redirect adds directly to startup time.
- Pre-fetch content keys and certificates before playback starts; use `AVContentKeySession` to initiate key loading proactively; cache certificates using HTTP expiry headers.
- Use a higher HLS target segment duration (~20 seconds) for audio-only streams; this improves resilience to network mobility and battery life without significantly impacting startup time.
- Minimize the number of network round trips during critical playback start.

## APIs & Frameworks

### AVFoundation
- `AVPlayer` — main playback engine for streaming audio on watchOS
  - xHE-AAC codec support **[NEW in watchOS 7]**
  - HE-AACv2, HE-AAC, AAC-LC — existing codec variants for fallback
- `AVContentKeySession` **[NEW in watchOS 7]** — FairPlay Streaming key handling; decoupled from asset lifecycle
  - `processContentKeyRequest(with:)` — handle key loading request from AVFoundation
  - `makeSecureTokenForExpirationDate(completionHandler:)` — key management
  - Supports proactive key pre-fetching before playback starts
- `AVContentKeyRequest` — on-demand key loading request from AVFoundation
- `AVAssetResourceLoaderDelegate` — alternative custom protocol approach (still supported)

### HLS (HTTP Live Streaming)
- xHE-AAC rendition variant in `EXT-X-STREAM-INF` **[NEW]** — recommended primary variant for watchOS
- Target duration `EXT-X-TARGETDURATION` — recommend ~20 seconds for audio-only watchOS streams
- Multi-codec manifests with xHE-AAC + AAC-LC/HE-AAC fallback variants

### FairPlay Streaming (FPS)
- Application Certificate — server certificate for key exchange
- SPC (Server Playback Context) — encrypted key request sent to key server
- CKC (Content Key Context) — encrypted key response from key server
- Key caching — use HTTP expiry to determine certificate/key cache lifetime

## Code Highlights

No code samples were provided in this session. Key implementation patterns:

1. Encode assets in xHE-AAC and include as primary HLS variant; add AAC-LC/HE-AAC as fallback variants in the manifest.
2. Use `AVContentKeySession` on watchOS 7 with the same FairPlay Streaming workflow used on iOS — the watch app handles key requests from AVFoundation and forwards to the key server.
3. Pre-fetch content keys and cache the application certificate to reduce startup latency on constrained watch networks.

## Takeaways

- xHE-AAC is now the recommended codec for audio streaming on watchOS 7 — it delivers better quality at lower bitrates, which matters on Watch where network bandwidth is limited and variable.
- FairPlay Streaming via `AVContentKeySession` is now supported on watchOS 7, enabling apps to stream DRM-protected content without a separate custom key-handling approach.
- `AVContentKeySession` decouples key loading from asset lifecycle; pre-fetching keys before playback starts is strongly recommended to minimize startup latency on watch.
- Use ~20-second HLS segment target durations for audio-only watch streams to maximize resilience to network drops without harming startup time.

---
_Source: WWDC20 Session 10636 page (abstract, transcript, and resource links)._
