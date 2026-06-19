# What's New in Low-Latency HLS
**WWDC20 · Session 10228** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10228/)

_Platforms:_ iOS 14, tvOS 14, macOS Big Sur 11

## Overview
Low-Latency HLS (LL-HLS) exited beta at WWDC20, becoming available to all developers without an entitlement for the first time. The protocol delivers live video with two seconds or less of stream delay, combining the quality and scalability of traditional HLS with latency competitive with broadcast television and social media platforms.

The Low-Latency extensions were formally incorporated into the second revision draft of the HLS RFC on ietf.org, adding appendices for the Low-Latency Server Configuration Profile and the CDN tune-in algorithm. The provisional beta spec page transitioned to an informative article with descriptions and examples.

Several significant protocol changes shipped based on beta feedback, most notably replacing HTTP/2 Push with Blocking Preload Hints, improving CDN compatibility, and adding CMAF support to the server reference implementation.

## Key Topics

### Exit from Beta
LL-HLS is now generally available in iOS 14, tvOS 14, and macOS Big Sur. No entitlement is required. Full HLS feature parity is included: adaptive bitrate switching, FairPlay Streaming, fMP4 CMAF, and more.

### Protocol Formalization
All Low-Latency rules are now part of the core HLS RFC on ietf.org (second revision draft), including a Server Configuration Profile appendix and the CDN tune-in algorithm appendix.

### Blocking Preload Hints (Replacing H2 Push)
HTTP/2 Push was removed because it was incompatible with ad-supported content delivery workflows and CDN architectures. Blocking Preload Hints replace it: the client requests the next partial segment before it is ready, and the server holds the connection open until it can respond. This also drives CDN cache fill directly from the client request, eliminating an extra edge-to-origin round trip.

### Rendition Report Changes
The report delivery directive was eliminated to prevent combinatorial explosion of request URLs that would reduce CDN caching efficiency. All Rendition Reports are now included in every playlist update.

### Playlist Delta Updates Enhancement
Date-range tags can now be incorporated into Playlist Delta Updates, so long DVR windows only carry the most recent date-range tag rather than the full history.

### Gap Signaling
New attributes on `EXT-X-PART` and `EXT-X-RENDITION-REPORT` tags signal encoded outages, enabling clients to handle gaps better in Low-Latency streams.

### Updated Server Reference Implementation
The HLS Low-Latency tools were updated to support fMP4/CMAF packaging. The Go-based web server now bundles HTTP/2 and delivery directive handling in a single script, replacing the previous PHP + separate web server setup. The Low-Latency tools are merged back into the main HLS tools package — one download for everything.

## APIs & Frameworks

- **HLS (HTTP Live Streaming)** — core streaming protocol
- **Low-Latency HLS extensions** **[NEW — out of beta]** — sub-2-second live stream delay
- **`EXT-X-PART`** tag — partial segment declaration **[NEW attribute: GAP]**
- **`EXT-X-PRELOAD-HINT`** tag — Blocking Preload Hint mechanism **[NEW, replaces H2 Push]**
- **`EXT-X-RENDITION-REPORT`** tag — rendition status in every playlist update **[NEW attribute: GAP]**
- **`EXT-X-SERVER-CONTROL`** tag — server-side Low-Latency configuration
- **Playlist Delta Updates (`_HLS_skip`)** — now supports date-range tags **[UPDATED]**
- **Blocking Playlist Reload** — client-driven request hold for next playlist
- **fMP4 / CMAF** — fragmented MPEG-4 packaging support in reference server **[NEW]**
- **FairPlay Streaming** — DRM support in LL-HLS
- **HTTP/2** — required transport (Push no longer required)
- **HLS Tools (Go-based server)** — combined LL-HLS + standard HLS tooling package **[UPDATED]**

## Code Highlights

This session is conceptual/protocol-focused. No code samples were presented. See the related sessions for implementation detail:
- "Discover HLS Blocking Preload Hints" (WWDC20 Session 10229)
- "Optimize live streams with HLS Playlist Delta Updates" (WWDC20 Session 10230)
- "Reduce latency with HLS Blocking Playlist Reload" (WWDC20 Session 10231)
- "Improve stream authoring with HLS Tools" (WWDC20 Session 10225)

## Takeaways

- LL-HLS is production-ready and entitlement-free starting with iOS 14, tvOS 14, and macOS Big Sur 11.
- HTTP/2 Push is replaced by Blocking Preload Hints, improving CDN compatibility and performance.
- All Low-Latency rules are now normative in the HLS RFC on ietf.org; the reference server supports CMAF and is easier to deploy.
- Developers targeting low-latency live streaming should adopt LL-HLS now, as the protocol is stable and fully documented.

---
_Source: WWDC20 Session 10228 page (abstract, transcript, and resource links)._
