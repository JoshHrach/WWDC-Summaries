# Explore Media Formats for the Web
**WWDC23 · Session 10122** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10122/)

_Platforms:_ Safari 17, iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
This session covers modern image formats and new video streaming technology added to Safari 17. The image section explains when to use JPEG XL, AVIF, HEIC, and WebP alongside legacy formats (GIF, JPEG, PNG), using the HTML `<picture>` element for progressive enhancement without User-Agent sniffing. The video section introduces the **Managed Media Source API** (`ManagedMediaSource`), a new Safari 17 API that combines the flexibility of Media Source Extensions (MSE) with the power efficiency of native HLS playback — enabling 5G modem access, intelligent buffer eviction under memory pressure, and automatic AirPlay support via `<source>` fallback.

## Key Topics

- **Legacy image formats** — GIF (limited color, animation), JPEG (lossy, progressive, photos), PNG (lossless, transparency, illustrations). Still required for broad compatibility but supplemented by modern formats.
- **Modern image formats in Safari 17** — Four formats now supported (WebP since Safari 14; JPEG XL, HEIC new in Safari 17; AVIF widely supported):
  - **JPEG XL** **[NEW in Safari 17]** — High compression via Modular Entropy Coding; lossless transcoding from JPEG (up to 60% size reduction); progressive decoding; supports wide color gamut and HDR.
  - **AVIF** **[NEW in Safari 17]** — AV1-based; up to 10x smaller than JPEG; up to 12-bit color; widest cross-browser support; best choice for fallback after newer formats.
  - **HEIC / HEIF** **[NEW in Safari 17]** — HEVC-based; format used by iPhone/iPad camera; hardware-accelerated in WKWebView; not widely supported on non-Apple platforms.
  - **WebP** (Safari 14+) — Advanced compression; animations with video-like quality; broader cross-browser support.
- **Wide color gamut and HDR** — JPEG XL, AVIF, and HEIC all support wide color gamut (billions of colors) and HDR (greater brightness range); enables more vibrant landscape and skin tone rendering.
- **HTML `<picture>` element** — Browser selects first matching `<source>` from top to bottom; enables serving HEIC → JPEG XL → AVIF → WebP → JPEG without server-side UA detection; `<img>` src is the final fallback.
- **Evolution of adaptive video streaming** — Timeline from plugins (Flash/QuickTime) → HTML5 video → HLS (Apple, 2009) → Media Source Extensions (MSE, 2013); MSE drawbacks: high power usage on mobile, inability to use 5G modem, complex buffer management.
- **Managed Media Source API** **[NEW in Safari 17]** — Drop-in MSE replacement; browser controls buffering timing via `startstreaming`/`endstreaming` events; intelligent buffer eviction via `bufferedchange` event; enables 5G modem access on iPhone/iPad; live content auto-switches to LTE/4G to extend battery. Available on macOS and iPadOS; experimental flag on iPhone in Safari 17.
- **AirPlay with Managed Media Source** — Add HLS `<source>` child element to `<video>` as fallback; Safari auto-shows AirPlay button and switches to HLS stream on AirPlay device. Required for Managed MSE to be active (or explicitly disable via `disableRemotePlayback`).
- **HLS.js integration** — Third-party library that transparently uses `ManagedMediaSource` when available; simple fallback pattern with native HLS check first, then HLS.js.

## APIs & Frameworks

**Web APIs (Safari 17)**
- `ManagedMediaSource` **[NEW]** — Managed version of `MediaSource`; browser controls buffer timing; reduces power use; enables 5G modem access
- `ManagedSourceBuffer` **[NEW]** — `SourceBuffer` counterpart for `ManagedMediaSource`
- `startstreaming` event **[NEW]** — fired on `ManagedSourceBuffer` when browser wants more data appended
- `endstreaming` event **[NEW]** — fired on `ManagedSourceBuffer` when browser has enough data; player should pause fetching
- `bufferedchange` event **[NEW]** — fired on `ManagedSourceBuffer` when buffered ranges change (including eviction)
- `MediaSource` — existing MSE API; replaced by `ManagedMediaSource` for efficiency
- `SourceBuffer` — existing MSE buffer; replaced by `ManagedSourceBuffer`
- `disableRemotePlayback` (Remote Playback API) — must call explicitly if no AirPlay source provided with Managed MSE

**HTML Elements**
- `<picture>` element — specify alternative image sources for format negotiation; browser selects first supported format
- `<source>` (in `<picture>`) — `type` attribute specifies MIME type (e.g., `image/jxl`, `image/avif`, `image/heic`, `image/webp`)
- `<source>` (in `<video>`) — specify HLS playlist URL as AirPlay fallback when using Managed MSE

**Image Formats**
- JPEG XL (`image/jxl`) **[NEW in Safari 17]** — lossless JPEG transcoding, progressive, wide color/HDR
- HEIC / HEIF (`image/heic`) **[NEW in Safari 17]** — iPhone/iPad camera format, hardware-accelerated in WKWebView
- AVIF (`image/avif`) **[NEW in Safari 17]** — AV1-based, 12-bit color, widest compatibility
- WebP (`image/webp`) — supported since Safari 14; advanced compression; animations

## Code Highlights

Progressive image format selection with `<picture>`:
```html
<picture>
  <source srcset="image.heic" type="image/heic">
  <source srcset="image.jxl" type="image/jxl">
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Example image">
</picture>
```

Migrating from MSE to Managed Media Source with fallback:
```javascript
function getMediaSource() {
    if (typeof ManagedMediaSource !== "undefined") {
        return ManagedMediaSource;
    }
    return MediaSource;
}
const MS = getMediaSource();
const mediaSource = new MS();
```

Handling Managed Media Source buffer events:
```javascript
sourceBuffer.addEventListener("startstreaming", fetchAndAppendNextSegment);
sourceBuffer.addEventListener("endstreaming", stopFetching);
sourceBuffer.addEventListener("bufferedchange", checkBufferedRanges);
```

AirPlay fallback for Managed MSE:
```html
<video id="my-video">
  <source src="playlist.m3u8" type="application/x-mpegURL">
</video>
```

## Takeaways

- Use the HTML `<picture>` element to offer modern formats (HEIC, JPEG XL, AVIF, WebP) as progressive enhancement over JPEG/PNG without any server-side User-Agent detection — the browser automatically picks the first supported format.
- Replace `MediaSource` with `ManagedMediaSource` in Safari 17 for better power efficiency and 5G modem access; migration requires only a constructor swap plus handlers for `startstreaming`, `endstreaming`, and `bufferedchange` events.
- Always provide an HLS `<source>` child in your `<video>` element when using Managed Media Source to enable AirPlay; Safari will automatically display the AirPlay button and switch to HLS on the connected device.
- AVIF has the broadest cross-browser support among modern formats and should be included as a fallback; HEIC is ideal for WKWebView-based apps since it maps directly to the iPhone/iPad camera format and is hardware-accelerated.

---
_Source: WWDC23 Session 10122 page (abstract, transcript, and resource links)._
