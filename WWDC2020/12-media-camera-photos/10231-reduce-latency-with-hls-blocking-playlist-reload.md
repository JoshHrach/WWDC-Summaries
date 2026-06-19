# Reduce latency with HLS Blocking Playlist Reload
**WWDC20 · Session 10231** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10231/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14 (server-side HLS protocol feature)

## Overview
Blocking Playlist Reload is a required component of Low-Latency HLS (LL-HLS) that dramatically improves segment discovery time in live streams. Instead of clients polling the server at fixed intervals (every target duration), clients can tell the server to hold the playlist request until a newer version — containing the next unseen segment or partial segment — is available. This eliminates the "stale playlist" problem common with HTTP caches and reduces latency for both LL-HLS and traditional live HLS streams.

The mechanism uses HLS **delivery directives** — query parameters appended to GET requests — allowing clients and servers to negotiate blocking behavior without changing the fundamental playlist format. The session explains the request/response flow for both regular and partial segments, edge cases, and how Blocking Playlist Reload interacts with CDNs and HTTP caches.

## Key Topics

### The Problem with Traditional Playlist Polling
- Clients reload the same playlist URL every target duration; if a client just misses a playlist update, it must wait another full target duration
- Stale playlist responses from HTTP caches compound the delay
- Low-Latency HLS requires much faster segment discovery (partial segments may be only 2 seconds long)

### How Blocking Playlist Reload Works
- Server advertises support by adding `CAN-BLOCK-RELOAD=YES` attribute to the `#EXT-X-SERVER-CONTROL` tag
- Client loads the playlist normally on first request (no delivery directives)
- Client immediately issues the next reload request with the `_HLS_msn` delivery directive set to the next media sequence number it expects
- For LL-HLS playlists: client also adds `_HLS_part` to specify the next partial segment expected
- Server holds the request until the playlist is updated to include that segment/part, then responds with the updated playlist
- If the server already has a playlist that is new enough, it responds immediately without blocking

### Delivery Directives
- `_HLS_msn=<N>` — request a playlist containing at least media sequence number N
- `_HLS_part=<P>` — combined with `_HLS_msn`, request a playlist containing at least part P of segment N
- Only applicable to live playlists (playlists with `#EXT-X-ENDLIST` ignore them)
- Server translates a part number beyond the end of a segment into part 0 of the following segment
- Server returns an error (HTTP error) if the playlist does not update within 3 target durations

### Edge Cases
- If the requested segment has already rolled out of the playlist window, the server unblocks immediately and returns the current playlist
- If the playlist has already advanced past the requested MSN, the server responds immediately with the current (newer) version
- Delivery directives are ignored on VOD playlists (those with `#EXT-X-ENDLIST`)

### CDN and HTTP Cache Interaction
- Unique URL per playlist update (unique `_HLS_msn`/`_HLS_part` combination) enables each version to be cached independently
- New requests with previously-unseen query parameter combinations bypass stale cache entries automatically
- Each response has a longer useful lifetime (recommended: 6 target durations for successful responses) vs. traditional HLS where responses must expire at 0.5 target durations
- CDNs should be configured to **coalesce duplicate edge requests** into a single request to the origin to reduce origin load
- Benefits both LL-HLS (required) and regular live HLS (optional but beneficial)

## APIs & Frameworks

This session covers server-side HLS protocol features; no client-side Apple SDK APIs are involved:

- **HLS (HTTP Live Streaming)** protocol — playlist format
- `#EXT-X-SERVER-CONTROL` tag — server capability advertisement
  - `CAN-BLOCK-RELOAD=YES` **[NEW for LL-HLS]** — server supports blocking playlist reloads
- `_HLS_msn` query parameter **[NEW delivery directive]** — request playlist with minimum media sequence number
- `_HLS_part` query parameter **[NEW delivery directive]** — request playlist with minimum partial segment index (used with `_HLS_msn`)
- `#EXT-X-TARGETDURATION` — regular segment target duration (e.g., 6 seconds)
- `#EXT-X-PART-INF` / `PART-TARGET` — partial segment target duration (e.g., 2 seconds) for LL-HLS
- `#EXT-X-MEDIA-SEQUENCE` — media sequence number counter in playlist
- `#EXT-X-PART` — partial segment declaration in LL-HLS playlist
- `#EXT-X-ENDLIST` — VOD playlist terminator (disables delivery directives)
- HTTP caching: `Cache-Control` headers, CDN request coalescing

## Code Highlights

No client-side code samples; this is a server/CDN configuration topic. Example playlist demonstrating `CAN-BLOCK-RELOAD`:

```
#EXTM3U
#EXT-X-VERSION:6
#EXT-X-TARGETDURATION:6
#EXT-X-SERVER-CONTROL:CAN-BLOCK-RELOAD=YES,PART-HOLD-BACK=3.0
#EXT-X-MEDIA-SEQUENCE:19
...
#EXTINF:6.0,
segmentE.ts   (Media Sequence Number 23)
```

Client's next request for regular segment:
```
GET /playlist.m3u8?_HLS_msn=24
```

Client's next request for LL-HLS partial segment:
```
GET /playlist.m3u8?_HLS_msn=7&_HLS_part=2
```

## Takeaways

- Adopt `CAN-BLOCK-RELOAD=YES` in `#EXT-X-SERVER-CONTROL` to enable Blocking Playlist Reload; it is required for Low-Latency HLS and beneficial for all live HLS streams.
- Unique delivery-directive URLs per playlist version fix the stale-cache problem inherent to traditional HLS polling and allow CDNs to cache each version independently for up to 6 target durations.
- Configure CDNs to coalesce duplicate edge requests into a single origin request to prevent origin overload under high viewer counts.
- Clients automatically handle edge cases (advanced playlists, rolled-off segments) because the server always returns the current playlist if it meets or exceeds the requested MSN/part.

---
_Source: WWDC20 Session 10231 page (abstract, chapter summaries, code samples, and resource links)._
