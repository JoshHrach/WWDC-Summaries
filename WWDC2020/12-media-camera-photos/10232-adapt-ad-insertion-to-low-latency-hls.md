# Adapt Ad Insertion to Low-Latency HLS
**WWDC20 · Session 10232** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10232/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session covers how to integrate server-side ad insertion (SSAI) into Low-Latency HLS (LL-HLS) streams. The speaker reviews how regular live HLS works — segments added every target duration, clients reloading playlists on the same cadence — and then explains how LL-HLS partial segments reduce the publishing latency from a full segment duration (e.g., 6 seconds) down to the partial segment duration (e.g., 2 seconds).

Ad insertion in LL-HLS follows the same general pattern as regular HLS: markers from the source feed trigger a short alignment segment, followed by ad segments (separated by `#EXT-X-DISCONTINUITY` tags), and then a return to program segments. The key difference is that in LL-HLS, ads are also spooled out as partial segments at the live edge — so the playlist updates more frequently during ad breaks, preserving the low-latency timing model throughout.

The session also covers preload hinting requirements for ad segments and the "early return" scenario where a live broadcast cuts back to program content before the ad finishes.

## Key Topics
- **Regular live HLS timing model** — Segments published every target duration; live edge defined by the last segment in the playlist; publishing delay equals segment duration.
- **LL-HLS partial segments** — Partial segments subdivide parent segments at the live edge, reducing publishing latency proportionally (e.g., 2-second partials instead of 6-second segments).
- **Server-side ad insertion (SSAI) overview** — Source feed with markers; decisioning engine selects from inventory; packager replaces program segments with ad segments; discontinuity tags separate content.
- **Short/alignment segments** — Ads may not land on regular segment boundaries; packager inserts a short segment to align ad start precisely.
- **Ad spooling in LL-HLS** — Same segmentation as regular HLS (boundaries, discontinuities, key rotation), but ads also publish as partial segments at the partial segment cadence (e.g., every 2 seconds).
- **Blocking Playlist Reload** — The HLS Origin API feature (part of LL-HLS) must be implemented for both program and ad segments to preserve the low-latency timing model; covered in a companion WWDC20 session.
- **Preload Hinting** — Every playlist must include a `#EXT-X-PRELOAD-HINT` tag pointing to the next expected partial segment URL. Ads require hints just like program content, but ad origins do not need to enforce blocking semantics since ad content is prerecorded.
- **Media initialization section hints** — When transitioning to an ad, the first preload hint signals a new map (media initialization section) before the first ad partial segment URL.
- **Early return** — When a live producer cuts back to program content mid-ad break: stop serving ad partials, conjure a short parent segment ending at the last published partial, signal a discontinuity, and update the preload hint to point to the next program partial segment.

## APIs & Frameworks

This is a server-side streaming/protocol session. No client-side Swift or Objective-C APIs are introduced. All technical content concerns the HLS playlist format and origin server behavior.

### HLS Playlist Tags (relevant to ad insertion in LL-HLS)
- **`#EXT-X-DISCONTINUITY`** — Separates program and ad content segments in the playlist
- **`#EXT-X-MAP`** — Specifies media initialization section; a new map tag is required when transitioning codec or format at an ad boundary
- **`#EXT-X-PART`** — Partial segment tag at the live edge; ads must be served as `#EXT-X-PART` entries at the partial segment cadence
- **`#EXT-X-PRELOAD-HINT`** — Points to the next expected partial segment URL (program or ad); required in every playlist update
- **Blocking Playlist Reload** (`_HLS_msn`, `_HLS_part` query parameters) — Origin API enabling clients to hold open playlist requests until new content is available; must be supported for both program and ad content

## Code Highlights
No client-side code samples. The session demonstrates playlist structure with annotated text diagrams showing segment/partial segment sequencing, discontinuity placement, and preload hint transitions at ad boundaries.

Example playlist structure at an ad transition (conceptual):
```
#EXTM3U
...
#EXT-X-PART:DURATION=2.0,URI="prog_seg7_part1.mp4"  ; last program partial
#EXT-X-PRELOAD-HINT:TYPE=MAP,URI="ad_init.mp4"       ; hints new map for ad
#EXT-X-PRELOAD-HINT:TYPE=PART,URI="ad_seg1_part1.mp4"; hints first ad partial
```

After 2 seconds, the updated playlist:
```
...
#EXT-X-DISCONTINUITY
#EXT-X-MAP:URI="ad_init.mp4"
#EXT-X-PART:DURATION=2.0,URI="ad_seg1_part1.mp4"
#EXT-X-PRELOAD-HINT:TYPE=PART,URI="ad_seg1_part2.mp4"
```

## Takeaways
- Ad insertion in LL-HLS mirrors regular HLS in structure (alignment segments, discontinuities, key rotation) but requires ads to also publish as partial segments at the live-edge cadence.
- Blocking Playlist Reload must be implemented on the origin for both program and ad segments to preserve the LL-HLS timing model.
- Preload hints are required for ad partials just like program partials; ad origins do not need to enforce blocking behavior since ad content is prerecorded.
- Early return from an ad break is handled by stopping ad partials, synthesizing a short parent segment, signaling discontinuity, and updating the preload hint to the next program partial.

---
_Source: WWDC20 Session 10232 page (abstract, chapter summaries, code samples, and resource links)._
