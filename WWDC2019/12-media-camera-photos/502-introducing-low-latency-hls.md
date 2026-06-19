# Introducing Low-Latency HLS
**WWDC19 · Session 502** · [Watch](https://developer.apple.com/videos/play/wwdc2019/502/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, tvOS 13; server-side/CDN

## Overview
Low-Latency HLS is a new mode of HTTP Live Streaming that achieves 1–2 seconds of end-to-end latency from production backend to viewer, at scale over public internet using commodity CDNs. This is a major improvement over regular HLS (6–30+ seconds) and matches or beats traditional broadcast latency. The target is specifically live, socially-shared content like sports, breaking news, live gaming, and awards shows where audience synchronization matters.

The design preserves everything that makes regular HLS compelling: adaptive bitrate, content protection, ad insertion, CDN scale, and backward compatibility — older clients fall back to normal-latency HLS automatically. The technology requires HTTP/2 (for Push support), new server-side playlist authoring, CDN configuration for request aggregation, and a published Low-Latency HLS specification plus reference tools provided by Apple.

For AVPlayer app developers, Low-Latency HLS works automatically when pointing at a compliant stream. Two new AVPlayer APIs enable fine-tuning of live-edge offset behavior.

## Key Topics

**Root Cause of Regular HLS Latency**
Regular HLS latency comes from four stacked delays: (1) 6-second segments mean media sits on the server for up to 6 seconds before publication; (2) client polling may miss a segment and wait another 6 seconds; (3) a second round-trip is needed to actually fetch the discovered segment; (4) CDN caches serve stale playlists due to time-to-live policies, adding more delay.

**Five Changes for Low-Latency HLS**

1. **Partial Segments (Parts)** — Segments are now also published as short partial segments (e.g., 0.5s CMAF chunks) added to the playlist as they complete. Clients at the live edge load Parts rather than waiting for the full 6s parent segment. Parts drift off the playlist as their parent segment moves away from the live edge, keeping playlists compact. New playlist tags: `#EXT-X-PART-INF:PART-TARGET=<seconds>` and `#EXT-X-PART:URI=...,DURATION=...,INDEPENDENT=YES`.

2. **Blocking Playlist Reload** — Instead of polling, clients request the next playlist update in advance using a query parameter (`_HLS_msn=<seq>` or `_HLS_msn=<seq>&_HLS_part=<part>`). The server holds the request open until that update is ready, then immediately responds. CDN-side: because each playlist update has a distinct query-parameter URL, CDNs treat each as a separate cache entity, eliminating stale-cache delivery. Server advertises support via `#EXT-X-SERVER-CONTROL:CAN-BLOCK-RELOAD=YES`.

3. **HTTP/2 Push** — When requesting a blocking playlist update, the client includes a `_HLS_push=<segment>` query parameter. When the server unblocks the playlist response, it simultaneously pushes the referenced partial segment, eliminating the follow-up round trip to fetch it. Requires HTTP/2.

4. **Delta Playlist Updates** — After the initial full playlist load, clients request delta updates (`_HLS_skip=YES`). The server returns only the most recent segments near the live edge, replacing the earlier portion with an `#EXT-X-SKIP:SKIPPED-SEGMENTS=<n>` tag. Server advertises the skip distance via `#EXT-X-SERVER-CONTROL:CAN-SKIP-UNTIL=<seconds>`. Updates often fit in a single network packet.

5. **Rendition Reports** — Each playlist update includes `#EXT-X-RENDITION-REPORT` tags peeking into other bitrate renditions' current sequence numbers and part numbers. This allows the client to compose a direct request to the most current version of an alternate tier without needing to first refresh that tier's playlist — enabling faster bitrate switching at the live edge.

**HLS Origin API**
A new mechanism for clients to advertise capability and send directives to servers. Directives are reserved query parameters beginning with `_HLS_` on playlist URLs. Parameters appear in deterministic order to preserve CDN cache key consistency.

**CDN Configuration Requirements**
- Use HTTP/2 with Push support and standard priority controls.
- Put the full bitrate ladder on each CDN server (minimize connection setup for tier switching).
- Configure request aggregation (coalescing): if two clients request the same blocking update simultaneously, park the second request and serve them both when the first response arrives (Apache Traffic Server: "read-while-write"; other CDNs: "early publish" or similar).

**App Developer APIs**
AVPlayer automatically uses Low-Latency mode when connected to a compliant stream — no code changes required. Two new APIs for fine-tuning:
- `AVPlayerItem.recommendedTimeOffsetFromLive` — system recommendation based on observed round-trip time; read to detect if current offset risks stalling.
- `AVPlayerItem.automaticallyPreservesTimeOffsetFromLive` — when `true`, after a buffering stall AVPlayer jumps back to the same distance from live rather than resuming at the stalled position, keeping the viewer at live.

**Rollout / Entitlement**
During the beta period, apps require a special entitlement to enable Low-Latency mode. Up to 10,000 beta users are accessible via TestFlight. Production App Store submission follows after the beta.

**Reference Tools**
Apple provides the Low-Latency HLS Beta Tools package: a programmatic test stream generator, a camera-based stream packager producing Low-Latency playlists, and an Apache front-end implementing the full Origin API (Blocking Playlist Reload, Delta Updates, Rendition Reports).

## APIs & Frameworks

**HTTP Live Streaming (Low-Latency Mode)** **[NEW]**

New playlist tags:
- `#EXT-X-PART-INF:PART-TARGET=<seconds>` **[NEW]**
- `#EXT-X-PART:URI=...,DURATION=...,INDEPENDENT=YES/NO` **[NEW]**
- `#EXT-X-SERVER-CONTROL:CAN-BLOCK-RELOAD=YES,CAN-SKIP-UNTIL=<seconds>` **[NEW]**
- `#EXT-X-SKIP:SKIPPED-SEGMENTS=<n>` **[NEW]**
- `#EXT-X-RENDITION-REPORT:URI=...,LAST-MSN=...,LAST-PART=...` **[NEW]**

New query parameters (HLS Origin API):
- `_HLS_msn=<n>` — request blocking playlist update containing media sequence n **[NEW]**
- `_HLS_part=<n>` — additionally wait for part n of that sequence **[NEW]**
- `_HLS_skip=YES` — request delta playlist update **[NEW]**
- `_HLS_push=<segment>` — request server push of segment alongside playlist **[NEW]**
- `_HLS_report=<uri>` — request Rendition Report for another rendition **[NEW]**

**AVFoundation** (AVPlayer)
- `AVPlayerItem.recommendedTimeOffsetFromLive: CMTime` **[NEW]**
- `AVPlayerItem.automaticallyPreservesTimeOffsetFromLive: Bool` **[NEW]**
- `AVPlayer` / `AVPlayerItem` — automatic Low-Latency HLS support (no code change needed) **[NEW behavior]**

**Transport / Protocol**
- HTTP/2 with Server Push — required for Low-Latency HLS
- CMAF (Common Media Application Format) chunks — recommended partial segment format for FMP4 content
- Transport Stream (TS) — also supported as partial segment format

## Code Highlights

Configuring live-edge offset in AVPlayer:
```swift
let playerItem = AVPlayerItem(url: streamURL)

// Read system recommendation and compare to current offset
let recommended = playerItem.recommendedTimeOffsetFromLive
// If current offset is too close to live edge (risk of stall), back off:
// playerItem.configuredTimeOffsetFromLive = recommended + CMTime(seconds: 0.5, ...)

// Keep viewer at live after buffering stalls:
playerItem.automaticallyPreservesTimeOffsetFromLive = true

let player = AVPlayer(playerItem: playerItem)
```

Example playlist fragment with Parts and server control:
```
#EXTM3U
#EXT-X-SERVER-CONTROL:CAN-BLOCK-RELOAD=YES,CAN-SKIP-UNTIL=36.0
#EXT-X-PART-INF:PART-TARGET=0.5
...
#EXTINF:6.0,
segment43.ts
#EXT-X-PART:URI="segment44.0.ts",DURATION=0.5,INDEPENDENT=YES
#EXT-X-PART:URI="segment44.1.ts",DURATION=0.5
...
```

Blocking playlist reload request URL:
```
GET /live.m3u8?_HLS_msn=1803&_HLS_part=0&_HLS_push=segment44.0.ts
```

Delta playlist reload request:
```
GET /live.m3u8?_HLS_msn=1803&_HLS_skip=YES
```

## Takeaways
- Low-Latency HLS achieves 1–2 second live latency at CDN scale by combining partial segments, blocking playlist reload with cache-busting URLs, HTTP/2 Push, delta updates, and rendition reports — five incremental changes to existing HLS mechanics.
- App developers using AVPlayer get Low-Latency automatically; the two new `AVPlayerItem` properties for live-edge offset control are optional quality-of-experience tuning.
- Server and CDN operators have the most work: adopting the HLS Origin API, emitting partial segments, enabling HTTP/2 with Push, and configuring request coalescing (aggregation) on CDN edges.
- The Low-Latency HLS spec and reference Apache-based tools are available via the session page to bootstrap backend implementations.

---
_Source: WWDC19 Session 502 page (abstract, chapter summaries, code samples, and resource links)._
