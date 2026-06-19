# Deliver Reliable Streams with HLS Content Steering
**WWDC22 · Session 10144** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10144/)

_Platforms:_ iOS, iPadOS, macOS, tvOS (AVFoundation / HLS)

## Overview
This session covers two new additions to HLS Content Steering (introduced in WWDC21): **Pathway Cloning** and **bucket-based Steering Server rules**. Content Steering is the HLS mechanism that lets a server-side Steering Server dynamically redirect clients between CDN Pathways at runtime, enabling load balancing, failover, and regional traffic management without requiring a new multivariant playlist.

Pathway Cloning solves the problem of dynamically spawned CDN fleets: because a new CDN did not exist when the client downloaded the multivariant playlist, it has no Pathway for that CDN. The Steering Manifest can now define new Pathways inline by cloning and modifying URI components of an existing Pathway. Bucket-based steering rules allow stateless, scalable partitioned traffic management across thousands of concurrent clients.

## Key Topics

### HLS Content Steering Recap
Each CDN is represented as a **Pathway** — a named group of variant streams and media renditions. Pathways are declared in the multivariant playlist using `PATHWAY-ID`. The `EXT-X-CONTENT-STEERING` tag specifies `SERVER-URI` (where clients poll for updates) and `PATHWAY-ID` (the initial preferred Pathway). Clients poll the Steering Server for **Steering Manifests** (JSON documents) that contain a `PATHWAY-PRIORITY` array listing Pathways in preference order. Clients switch Pathways immediately upon receiving a new priority order.

### Pathway Cloning (New in 2022)
A **Pathway Clone** is a new Pathway derived from an existing one by applying URI replacement rules. This is announced in the `PATHWAY-CLONES` array inside a Steering Manifest. Each clone object specifies:
- `BASE-ID` — the existing Pathway to clone from
- `ID` — the new Pathway ID
- `URI-REPLACEMENT` — an object describing how to transform URIs

Three URI replacement modes:
1. **`HOST`** — replaces only the hostname in all stream URIs of the cloned Pathway
2. **`PARAMS`** — inserts or replaces specific query parameters in all stream URIs
3. **`PER-VARIANT-URIS`** / **`PER-RENDITION-URIS`** — overrides specific stream URIs entirely, keyed by `STABLE-VARIANT-ID` or `STABLE-RENDITION-ID`; used when selected streams should be served from a different host than the rest

To use `PER-VARIANT-URIS` / `PER-RENDITION-URIS`, assign `STABLE-VARIANT-ID` and `STABLE-RENDITION-ID` attributes to variant and rendition streams in the multivariant playlist.

### Determining Which Pathways to Clone
Since the Steering Server is stateless, it cannot know which Pathways a given client's multivariant playlist contains. The recommended pattern is to embed the playlist's Pathway set as a query parameter in the `SERVER-URI` at playlist generation time. The server extracts the parameter, computes the difference between its current Pathway set and the client's known set, and includes only the delta in `PATHWAY-CLONES`.

### Bucket-Based Steering Server Rules
Clients are assigned to a **bucket** (a small integer, e.g., 0–11) by the server on first contact. The bucket number is appended to the `RELOAD-URI` in the Steering Manifest response so subsequent requests carry it. The server applies deterministic per-bucket rules — no session state required. Example: divide 12 buckets evenly across 3 CDN Pathways (4 buckets per CDN) to achieve 33%/33%/33% traffic distribution.

### Global Traffic Steering
Content Steering enables regional load balancing: route European clients to CDN1 via Pathway Priority, North American clients to CDN2, etc. When CDN load shifts (e.g., due to daylight), the Steering Server re-issues Manifests to rebalance in real time without re-fetching playlists.

## APIs & Frameworks

### HLS Multivariant Playlist Tags and Attributes
- `EXT-X-CONTENT-STEERING` tag **[NEW in HLS Content Steering 1.2]**
  - `SERVER-URI` — URL of the Steering Server
  - `PATHWAY-ID` — initial Pathway ID to use at startup
- `PATHWAY-ID=<id>` attribute on `EXT-X-STREAM-INF` and `EXT-X-MEDIA` — groups variants and renditions into named Pathways
- `STABLE-VARIANT-ID=<id>` **[NEW]** — stable identifier for a variant stream; used by `PER-VARIANT-URIS` in Pathway Cloning
- `STABLE-RENDITION-ID=<id>` **[NEW]** — stable identifier for a rendition; used by `PER-RENDITION-URIS` in Pathway Cloning

### Steering Manifest (JSON) Fields
- `PATHWAY-PRIORITY` — array of Pathway ID strings in descending preference order
- `RELOAD-URI` — URL the client should use for the next Steering Manifest request (used to embed bucket number)
- `PATHWAY-CLONES` **[NEW]** — array of Pathway Clone objects; each has:
  - `BASE-ID` — ID of the Pathway to clone
  - `ID` — new Pathway ID
  - `URI-REPLACEMENT` — object with optional fields:
    - `HOST` — replacement hostname string
    - `PARAMS` — dictionary of query parameter name → value to insert/replace
    - `PER-VARIANT-URIS` **[NEW]** — dictionary of `STABLE-VARIANT-ID` → full URI override
    - `PER-RENDITION-URIS` **[NEW]** — dictionary of `STABLE-RENDITION-ID` → full URI override

### HLS Tooling
- Apple HLS Tools — validate multivariant playlists after adding `EXT-X-CONTENT-STEERING` and `PATHWAY-ID` attributes
- IETF HLS specification (draft-pantos-hls-rfc8216bis) — authoritative technical reference

## Code Highlights

Multivariant playlist with Content Steering and two Pathways:
```
#EXTM3U
#EXT-X-CONTENT-STEERING:SERVER-URI="https://steering.example.com/steer?pathways=CDN1,CDN2",PATHWAY-ID="CDN1"

#EXT-X-STREAM-INF:BANDWIDTH=6000000,AUDIO="audio-cdn1",PATHWAY-ID="CDN1"
https://cdn1.example.com/high/index.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=3000000,AUDIO="audio-cdn1",PATHWAY-ID="CDN1"
https://cdn1.example.com/low/index.m3u8

#EXT-X-STREAM-INF:BANDWIDTH=6000000,AUDIO="audio-cdn2",PATHWAY-ID="CDN2"
https://cdn2.example.com/high/index.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=3000000,AUDIO="audio-cdn2",PATHWAY-ID="CDN2"
https://cdn2.example.com/low/index.m3u8
```

Steering Manifest redirecting a client to CDN1 (standard priority):
```json
{
  "PATHWAY-PRIORITY": ["CDN1", "CDN2"],
  "RELOAD-URI": "https://steering.example.com/steer?bucket=7"
}
```

Steering Manifest introducing a new CDN3 Pathway via cloning:
```json
{
  "PATHWAY-PRIORITY": ["CDN3", "CDN1", "CDN2"],
  "PATHWAY-CLONES": [
    {
      "BASE-ID": "CDN1",
      "ID": "CDN3",
      "URI-REPLACEMENT": {
        "HOST": "cdn3.example.com",
        "PARAMS": { "foo": "xyz", "bar": "123" }
      }
    }
  ],
  "RELOAD-URI": "https://steering.example.com/steer?bucket=7&pathways=CDN1,CDN2"
}
```

Per-stream URI override for a high-bitrate variant served from a faster host:
```json
"URI-REPLACEMENT": {
  "HOST": "cdn3.example.com",
  "PER-VARIANT-URIS": {
    "video-4k-dv": "https://faster.example.com/4k/index.m3u8"
  },
  "PER-RENDITION-URIS": {
    "audio-en-ac3": "https://faster.example.com/audio/en-ac3/index.m3u8"
  }
}
```

## Takeaways
- Content Steering 1.2 (WWDC21) standardized HLS CDN failover; the 2022 additions (Pathway Cloning, bucket steering) extend it to dynamically provisioned CDNs and stateless traffic partitioning.
- Pathway Cloning lets a Steering Server advertise newly spawned CDN fleets to existing clients without requiring a new multivariant playlist; only the changed URI components need to be transmitted.
- Embed the client's known Pathway set in `SERVER-URI` query parameters so the stateless Steering Server can compute which Pathways need to be cloned.
- Bucket-based steering assigns clients to buckets on first contact and embeds the bucket number in `RELOAD-URI`, enabling deterministic, scalable load-distribution rules without server-side session state.
- Use `STABLE-VARIANT-ID` / `STABLE-RENDITION-ID` in playlists to unlock per-stream URI replacement rules for streams that require a different host than the rest of the cloned Pathway.

---
_Source: WWDC22 Session 10144 page (abstract, transcript, and resource links)._
