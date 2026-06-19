# Improve Global Streaming Availability with HLS Content Steering
**WWDC21 · Session 10141** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10141/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
HLS Content Steering is a new HLS feature that allows streaming providers to dynamically control CDN selection policies for active clients via a server-side "steering server." Rather than relying on a fixed ordering of variant streams in the master playlist, providers can push real-time CDN priority updates to all connected clients, enabling live load balancing and near-instant failover without playlist changes. The feature is fully backward-compatible — existing clients that do not understand the new tags simply ignore them and fall back to normal variant selection.

## Key Topics

**The Problem: Static CDN Priority**
Traditional HLS master playlists list variant streams as a fixed ordered list. Clients walk through the list in order, which makes load balancing or CDN failover reactive and coarse-grained. Overloaded CDN nodes can only be addressed by redirecting new sessions; existing clients keep using the congested CDN. With Content Steering, providers host a lightweight steering server that pushes CDN priority updates to all active clients.

**Pathways: Grouping Variants by CDN**
Variants in the master playlist are annotated with a `PATHWAY-ID` attribute grouping them into "pathways" (typically one per CDN). Only variants in the currently active pathway participate in ABR variant selection. All existing variants are preserved, making the change backward-compatible.

**The CONTENT-STEERING Tag**
A new `#EXT-X-CONTENT-STEERING` tag in the master playlist specifies:
- `SERVER-URI` — URL of the steering server that delivers Steering Manifest responses
- `PATHWAY-ID` — initial pathway to use at startup

When media groups are present, duplicate renditions must exist for each pathway with unique `GROUP-ID` values (e.g., `CN-audio`, `JP-audio`).

**The Steering Manifest**
Clients periodically issue HTTP GET requests to the steering server. The server responds with a JSON Steering Manifest containing:
- `PATHWAY-PRIORITY` — ordered array of pathway IDs (most preferred first). Clients switch to the highest-priority pathway that has at least one eligible variant (not penalized by errors, codec unsupported, etc.).
- `TTL` — seconds until the next manifest request. Servers can vary TTL per client to spread load (add jitter).
- `RELOAD-URI` — optional; the URI to use for the next request (useful for injecting per-client session state).

**Client Evaluation Logic**
On receiving a Steering Manifest, the client:
1. Excludes penalized variant streams (network errors) and unsupported-codec variants.
2. Picks the highest-priority pathway with at least one eligible variant.
3. Switches to that pathway immediately if it differs from the current pathway.
4. Schedules the next request after `TTL` seconds.

**Use Cases**
- *Load balancing*: Push priority-list updates to a subset of clients to migrate traffic from an overloaded CDN to a less-loaded one in real time.
- *Error recovery/failover*: When a regional outage penalizes a pathway, clients fall through the priority list to the next working CDN without waiting for server-side playlist updates.

**Tooling**
Playlist and Steering Manifest validation are supported in Apple's latest HTTP Live Streaming Tools.

## APIs & Frameworks

- **HLS (HTTP Live Streaming)** — playlist-level feature, no Swift/Objective-C API
- `#EXT-X-CONTENT-STEERING` master playlist tag **[NEW]**
  - `SERVER-URI=<url>` — steering server endpoint
  - `PATHWAY-ID=<id>` — initial pathway
- `PATHWAY-ID=<id>` attribute on `#EXT-X-STREAM-INF` and `#EXT-X-I-FRAME-STREAM-INF` **[NEW]**
- `PATHWAY-ID=<id>` attribute on `#EXT-X-MEDIA` (renditions/media groups) **[NEW]**
- Steering Manifest (JSON) — served by provider's steering server **[NEW]**
  - `PATHWAY-PRIORITY: [String]` — ordered CDN/pathway priority list
  - `TTL: Int` — seconds until next manifest request
  - `RELOAD-URI: String` (optional) — next request URI
- HTTP Live Streaming Tools — adds Steering Manifest validation **[UPDATED]**
- **AVFoundation / AVPlayer** — transparent consumption; no new AVFoundation API required

## Code Highlights

Example master playlist with Content Steering:
```
#EXTM3U

#EXT-X-CONTENT-STEERING:SERVER-URI="https://steering.example.com/manifest",PATHWAY-ID="CN"

# CN pathway variants
#EXT-X-STREAM-INF:BANDWIDTH=6000000,CODECS="avc1.640028,mp4a.40.2",PATHWAY-ID="CN"
https://cdn-cn.example.com/high/stream.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=2000000,CODECS="avc1.4d401f,mp4a.40.2",PATHWAY-ID="CN"
https://cdn-cn.example.com/low/stream.m3u8

# JP pathway variants
#EXT-X-STREAM-INF:BANDWIDTH=6000000,CODECS="avc1.640028,mp4a.40.2",PATHWAY-ID="JP"
https://cdn-jp.example.com/high/stream.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=2000000,CODECS="avc1.4d401f,mp4a.40.2",PATHWAY-ID="JP"
https://cdn-jp.example.com/low/stream.m3u8
```

Example Steering Manifest response (JSON):
```json
{
  "PATHWAY-PRIORITY": ["JP", "SG", "CN"],
  "TTL": 300,
  "RELOAD-URI": "https://steering.example.com/manifest?session=abc123"
}
```

## Takeaways

- Content Steering enables real-time, server-driven CDN load balancing and failover for HLS streams without requiring playlist regeneration or client reconnection.
- The master playlist change is minimal and backward-compatible — add `#EXT-X-CONTENT-STEERING` tag and `PATHWAY-ID` attributes to existing variant and media group entries.
- The steering server only needs to serve a simple JSON document; the `TTL` field and optional jitter control how aggressively clients poll for updates.
- Clients automatically respect CDN penalty state (from network errors) when evaluating the priority list, so Content Steering complements rather than replaces HLS's built-in error handling.

---
_Source: WWDC21 Session 10141 page (abstract, full transcript, and resource links)._
