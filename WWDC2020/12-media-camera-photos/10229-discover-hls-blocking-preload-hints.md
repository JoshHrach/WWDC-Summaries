# Discover HLS Blocking Preload Hints
**WWDC20 · Session 10229** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10229/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14 (server-side streaming technology)

## Overview
This session is part of the Low-Latency HLS (LL-HLS) series and focuses specifically on Blocking Preload Hints — a required mechanism that allows HLS clients to request the next Partial Segment before it has been fully encoded, eliminating the round-trip bubble that would otherwise occur between a client seeing a new segment listed in a playlist update and then separately fetching it.

Preload Hints parallel the Blocking Playlist Reload feature: just as a client can ask for the next playlist version in advance (and the server blocks until it is ready), a client can ask for the next Partial Segment in advance. Both unblock at approximately the same time, so the client receives the segment and the updated playlist together with minimal latency. The session also explains how Preload Hints integrate with CMAF Chunk delivery via open-ended byte-range requests and Chunked Transfer Encoding, enabling interoperability with Low-Latency DASH.

## Key Topics

### Blocking Preload Hints Fundamentals
- Every LL-HLS playlist **must** include an `EXT-X-PRELOAD-HINT` tag pointing to the URL of the next expected Partial Segment.
- When the client receives the playlist, it immediately issues a `GET` request for the hinted URL; the server blocks the response until that Partial Segment is available.
- This eliminates the round-trip between "seeing a new segment in the playlist" and "requesting it" — typically critical at global CDN scale where edge-to-origin round trips can exceed 100 ms.
- A `Blocking Playlist Reload` request is usually pending simultaneously; both unblock and arrive at approximately the same time.
- Preload Hints are also used to signal upcoming `EXT-X-MAP` tags (e.g., at ad boundaries) so the client can begin fetching initialization data early.

### Playlist Syntax
- `EXT-X-PRELOAD-HINT` tag attributes:
  - `TYPE=PART` — hints a Partial Segment
  - `URI` — URL of the hinted resource
  - `BYTERANGE-START` — start offset for byte-range Partial Segments (updated each time a new chunk is appended)
  - `BYTERANGE-LENGTH` — omitted when length is not yet known (open-ended range)

### CMAF Chunk Delivery Integration
- Partial Segments can be expressed as byte-ranges of a larger parent segment URL.
- When delivering via CMAF Chunks with Chunked Transfer Encoding:
  - The `EXT-X-PRELOAD-HINT` tag contains only the parent segment URL (no byte-range length).
  - Each time a new CMAF Chunk is appended, the server updates `BYTERANGE-START` in the hint to point to the end of the last chunk.
  - The client issues one `GET` for the parent URL; the server progressively delivers each Chunk via Chunked Transfer Encoding.
  - This single-request model enables the same media to serve both LL-HLS and Low-Latency DASH clients.
  - Caveat: many CDNs do not support Chunked Transfer Encoding for live content delivery.

### Server Behavior Rules
- A server may change its plan after publishing a Preload Hint (e.g., an operator decides to return early from an ad). When the playlist updates without the previously hinted URL, the client cancels the pending hint request and loads whatever is in the new playlist.
- Hint requests for pre-recorded content (e.g., ads) can be served immediately without blocking.
- Preload Hints are **required** in all LL-HLS streams — they are not optional.

## APIs & Frameworks

### HLS Playlist Tags
- `EXT-X-PRELOAD-HINT` **[Required in LL-HLS]** — declares the next Partial Segment for preloading
  - `TYPE=PART`
  - `URI=<url>`
  - `BYTERANGE-START=<offset>` — updated per chunk append
  - `BYTERANGE-LENGTH=<length>` — omitted for open-ended requests
- `EXT-X-PART` — declares a Partial Segment in the playlist
  - `URI=<url>`, `BYTERANGE=<length>@<start>`, `DURATION=<seconds>`, `INDEPENDENT=YES`
- `EXT-X-MAP` — media initialization section; can also be hinted via `EXT-X-PRELOAD-HINT`
- `EXT-X-SERVER-CONTROL` — playlist-level tag that enables LL-HLS features (see session 10231)

### Delivery Protocols
- HTTP/2 — required for LL-HLS to allow multiple simultaneous blocking requests
- Chunked Transfer Encoding — required for CMAF Chunk delivery via single open-ended GET
- CMAF (Common Media Application Format) — fMP4 Fragments delivered as Chunks

### AVFoundation (client-side, handled automatically)
- `AVPlayer` / `AVPlayerItem` — automatically handles LL-HLS Preload Hints when using an LL-HLS stream URL
- No additional client-side API changes required

## Code Highlights
Preload hinting is a server-side and playlist-format feature — no client-side code changes are required. Example playlist snippet:

```
#EXTM3U
#EXT-X-VERSION:9
...
#EXT-X-PART:DURATION=1.0,URI="seg0.m4s",BYTERANGE=3000@0
#EXT-X-PART:DURATION=1.0,URI="seg0.m4s",BYTERANGE=5000@3000
#EXT-X-PRELOAD-HINT:TYPE=PART,URI="seg1.m4s",BYTERANGE-START=0
```

After the next chunk is appended, the playlist updates to:
```
#EXT-X-PART:DURATION=1.0,URI="seg1.m4s",BYTERANGE=3000@0
#EXT-X-PRELOAD-HINT:TYPE=PART,URI="seg1.m4s",BYTERANGE-START=3000
```

## Takeaways
- Blocking Preload Hints are **required** in LL-HLS playlists — include `EXT-X-PRELOAD-HINT` pointing to the next expected Partial Segment in every playlist update.
- The client issues the hint request immediately; the server blocks until the segment is ready, eliminating a full round-trip of latency that is especially costly at global CDN scale.
- Use open-ended byte-range Preload Hints with Chunked Transfer Encoding to deliver CMAF Chunks efficiently and achieve interoperability with Low-Latency DASH from the same origin media.
- Servers may change plans mid-hint (e.g., early ad return); clients will automatically cancel and reload from the updated playlist.

---
_Source: WWDC20 Session 10229 page (abstract, transcript, and resource links)._
