# Optimize live streams with HLS Playlist Delta Updates
**WWDC20 · Session 10230** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10230/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
HLS Playlist Delta Updates allow a live-stream server to send only the recently changed portion of a media playlist instead of the entire file on every reload. This is especially valuable for streams with large DVR windows, long-running events, or playlists dense with metadata tags — where full playlist downloads become expensive in both latency and bandwidth.

Delta Updates were introduced in a prior OS (iOS 13); this session covers the existing mechanism and a new extension added in iOS 14 / HLS playlist version 10 that also supports skipping `DATERANGE` tags.

## Key Topics

### Problem: Large Playlist Reload Overhead
- Live-stream clients poll the media playlist every target duration to discover new segments
- Long DVR windows and rich metadata produce large playlists — even after gzip compression
- Repeatedly downloading the full playlist wastes bandwidth and increases latency; on slow connections it can cause the player to select a lower quality tier

### How Playlist Delta Updates Work
1. Server advertises support by adding `CAN-SKIP-UNTIL=<seconds>` to its `#EXT-X-SERVER-CONTROL` tag — the value is the **skip limit** (minimum age of a segment entry to be eligible for skipping; always ≥ 6 × target duration)
2. Client downloads the first playlist response fully, establishing the baseline
3. On subsequent reloads, the client appends `_HLS_skip=YES` as a query parameter (the "HLS skip Delivery Directive") to the playlist URL
4. Server responds with a **delta update**: a partial playlist (version 9+) containing only what has changed
5. Client reconstructs the current full playlist by merging the delta with its cached previous version

### Delta Update Structure
- `#EXT-X-VERSION:9` — signals that this is a delta update (not backward-compatible; older clients must not attempt to parse it as a full playlist)
- `#EXT-X-MEDIA-SEQUENCE` — indicates how many segments have been removed since last reload (segments that rolled off the window)
- `#EXT-X-SKIP:SKIPPED-SEGMENTS=N` — replaces all segment URL lines (and their associated tags: `#EXT-X-DISCONTINUITY`, `#EXT-X-PROGRAM-DATE-TIME`, etc.) that are older than the skip limit; `N` is the count of skipped segments
- Remainder of the playlist: all segment entries and tags added since the skip limit (the "recent" portion)

### DATERANGE Tag Skipping (New in iOS 14 / Version 10)
- Problem: playlists with large numbers of `#EXT-X-DATERANGE` tags (used for ad insertion, chapter markers, etc.) still inflate the delta even after segment skipping
- Server advertises support with `CAN-SKIP-DATERANGES=YES` in `#EXT-X-SERVER-CONTROL`
- Client requests v2 delta with query parameter `_HLS_skip=v2`
- Server responds with `#EXT-X-VERSION:10` (incompatible; version 10 required)
- `#EXT-X-SKIP` tag now also includes `RECENTLY-REMOVED-DATERANGES="id1\tid2\t..."` — lists DATERANGE IDs removed since the skip limit
- Any DATERANGE added before the skip limit is omitted from the delta; the client retains it from its cached playlist
- Any DATERANGE added at or after the skip limit appears in the delta as usual
- After merging: client's reconstructed playlist contains all current DATERANGEs; removed ones are discarded based on `RECENTLY-REMOVED-DATERANGES`

### Client Reconstruction Algorithm
- Remove the first N segments (N from `#EXT-X-MEDIA-SEQUENCE` delta) from the cached playlist
- Replace the old segment entries (up to the skip limit) with the `#EXT-X-SKIP` tag acting as their proxy
- Append new segment entries from the delta
- For v2: additionally apply DATERANGE removals and merge new/updated DATERANGEs

### Benefits
- Reduces playlist response size from potentially thousands of lines to a few dozen lines per reload
- Lowers bandwidth consumption — particularly important for clients on slower connections (prevents unnecessary quality downgrades)
- Reduces reload latency, improving playback reliability
- Server-side: lowers CDN egress for playlist delivery on high-concurrency live events

## APIs & Frameworks

- **HLS (HTTP Live Streaming) — server-side playlist tags and client directives**
  - `#EXT-X-SERVER-CONTROL` — multi-attribute server capabilities advertisement tag
    - `CAN-SKIP-UNTIL=<seconds>` — enables Playlist Delta Updates; value is the skip limit (≥ 6 × `EXT-X-TARGETDURATION`)
    - `CAN-SKIP-DATERANGES=YES` **[NEW in iOS 14]** — enables DATERANGE skipping in v2 delta updates
  - `#EXT-X-SKIP` — delta update placeholder tag
    - `SKIPPED-SEGMENTS=N` — count of segment entries replaced by this tag
    - `RECENTLY-REMOVED-DATERANGES="<tab-separated IDs>"` **[NEW]** — DATERANGE IDs removed since skip limit (v2 only)
  - `#EXT-X-VERSION:9` — required for Playlist Delta Updates (segments only)
  - `#EXT-X-VERSION:10` **[NEW]** — required for v2 delta updates with DATERANGE skipping
  - `_HLS_skip=YES` — query parameter Delivery Directive for v1 delta updates (segment skipping only)
  - `_HLS_skip=v2` **[NEW]** — query parameter Delivery Directive for v2 delta updates (segment + DATERANGE skipping)
- **AVFoundation / AVKit** — client-side implementation is transparent; AVPlayer handles negotiation and playlist merging automatically

## Code Highlights

Server Control tag advertising both features (server-side playlist snippet):
```
#EXTM3U
#EXT-X-VERSION:10
#EXT-X-TARGETDURATION:6
#EXT-X-SERVER-CONTROL:CAN-BLOCK-RELOAD=YES,CAN-SKIP-UNTIL=36.0,CAN-SKIP-DATERANGES=YES
```

Example v1 delta update response (server sends in response to `_HLS_skip=YES`):
```
#EXTM3U
#EXT-X-VERSION:9
#EXT-X-MEDIA-SEQUENCE:1
#EXT-X-SKIP:SKIPPED-SEGMENTS=20
segment21.ts
#EXT-X-ENDLIST
```

Example v2 delta update response (server sends in response to `_HLS_skip=v2`):
```
#EXTM3U
#EXT-X-VERSION:10
#EXT-X-MEDIA-SEQUENCE:1
#EXT-X-SKIP:SKIPPED-SEGMENTS=20,RECENTLY-REMOVED-DATERANGES="daterange-id-1"
#EXT-X-DATERANGE:ID="daterange-P",...
segment21.ts
#EXT-X-DATERANGE:ID="daterange-Q",...
#EXT-X-ENDLIST
```

## Takeaways
- Playlist Delta Updates are a server-side optimization requiring no changes to playback client code — AVPlayer handles negotiation transparently
- Enable v1 Delta Updates by adding `CAN-SKIP-UNTIL=<N>` to `#EXT-X-SERVER-CONTROL`; set N to at least 6 × target duration
- Enable v2 (DATERANGE skipping, new in iOS 14) by also adding `CAN-SKIP-DATERANGES=YES`; respond with `#EXT-X-VERSION:10` deltas when clients send `_HLS_skip=v2`
- Critical: track which DATERANGEs have been removed since the skip limit and advertise them in `RECENTLY-REMOVED-DATERANGES` — failing to do so would leave stale DATERANGEs in client playlists
- Most impactful for streams with DVR windows over several hours, event streams with thousands of ad insertion markers, or any playlist that grows beyond a few hundred lines

---
_Source: WWDC20 Session 10230 page (abstract and transcript)._
