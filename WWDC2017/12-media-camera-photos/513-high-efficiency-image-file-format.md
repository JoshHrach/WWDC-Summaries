# High Efficiency Image File Format
**WWDC17 · Session 513** · [Watch](https://developer.apple.com/videos/play/wwdc2017/513/)

_Platforms:_ iOS 11, macOS High Sierra 10.13

## Overview
iOS 11 and macOS High Sierra introduce support for the High Efficiency Image File Format (HEIF), an ISO standard finalized in 2015 that Apple selected as its JPEG replacement after extensive evaluation. HEIF is a container format built on top of the ISO Base Media File Format (ISOBMFF, the same foundation as MP4/QuickTime) that completely separates container from codec — any codec can be embedded, though Apple uses HEVC (H.265), resulting in files with the `.heic` extension. The primary advantage over JPEG is approximately 2x better compression at equivalent visual quality, alongside native support for auxiliary images (depth maps, alpha channels), image sequences, tiles, rich metadata, HDR, wide color gamut, and high bit depth.

The session dives deep into the binary anatomy of a HEIF file: the `ftyp`, `meta`, and `mdat` boxes; the item property box (`ipco`) and item property association box (`ipma`); coded items (individual HEVC frames or tiles), derived items (image grids or overlays), and metadata items (EXIF, XMP). Image roles such as primary image, master, thumbnail, auxiliary, and hidden are defined by the standard to allow multiple representations of a scene to coexist in a single file.

HEVC was chosen specifically because its flexible coding tools — 64×64 down to 4×4 coding unit sizes, 35-angle intra prediction, CABAC entropy coding, deblocking and SAO filters — provide dramatically better compression than JPEG's 8×8 fixed-block, Huffman-coded approach, and because hardware HEVC decoders are becoming ubiquitous (Intel 6th-gen Core and later, Apple A-series chips).

## Key Topics
- **JPEG limitations** — fixed 8×8 block size, Huffman coding, no auxiliary images, no animation support, 25+ years old
- **HEIF design goals** — state-of-the-art compression, hardware-friendly encode/decode, high bit depth, wide color gamut, HDR, auxiliary images, animation, multi-image support, tiles, rich metadata, extensibility
- **Container anatomy** — ISOBMFF boxes: `ftyp`, `mdat`, `meta`, `moov`; item property box (`ipco`), association box (`ipma`), `pict` handler type
- **Image roles** — primary image (only one per file), master, thumbnail, auxiliary, hidden, derived, equivalent
- **Derived images** — image grid (tiles stitched together) and image overlay operations
- **Image sequences** — stored as MP4-style tracks with the `pict` track handler; supports capture time (burst) or display time (slideshow/animation) semantics; edit lists; looping; inter-frame prediction with optional constrained dependencies
- **Tiles (system tiles)** — independent HEVC-encoded items; parallel decode; reduced memory for resize/crop; used extensively by Apple's iOS 11 implementation (512×512 tiles, grid layout)
- **HEVC coding tools vs JPEG** — flexible block sizes (4×4–64×64 vs fixed 8×8); intra prediction (35 angular modes vs DC prediction); CABAC vs Huffman; per-block quantization parameters; deblocking filter; SAO filter
- **iOS 11 HEIF specifics** — `.heic` extension; HEVC Main Still Picture profile; HEVC Monochrome for depth; 512×512 tiles; 320×240 HEVC thumbnail; EXIF metadata embedded; depth as auxiliary image with XMP metadata
- **Box ordering best practice** — thumbnail early in file; `meta` box before coded data to enable pipeline setup before full download

## APIs & Frameworks

### Image I/O
- **`ImageIO` framework** **[NEW HEIF support]** — `CGImageSourceCreateWithURL`, `CGImageSourceCreateWithData` support `.heic` / `.heif` files
- **`kUTTypeHEIC`** **[NEW]** — uniform type identifier for HEIF/HEVC images
- **`CGImageDestinationAddImage`** — write HEIF-encoded images with appropriate UTI

### AVFoundation / Camera
- **`AVCapturePhotoOutput`** — new `isHighResolutionPhotoEnabled`; produces HEIF output on supported hardware
- **`AVCapturePhotoSettings`** — configure for HEVC/HEIF capture format

### Core Image / Photos
- **`CIImage`** — reads HEIF including auxiliary depth/disparity images
- **`PHAsset`** — represents HEIF assets in the Photos library; `PHAssetResourceType` includes depth data auxiliary resource

### File Format Structures (informational)
- **`ftyp` box** — file type and compatibility brands (`heic`, `heix`, `hevc`, `hevx`, `heim`, `heis`, `hevm`, `hevs`, `mif1`)
- **`meta` box** — top-level metadata; handler type `pict`
- **`ipco` box** (item property container) — descriptive properties: `ispe` (image size), `colr` (color info), `hvcC` (HEVC decoder config), `auxC` (auxiliary type: alpha/depth)
- **`ipco` box** — transformative properties: `clap` (clean aperture/crop), `irot` (rotation), `imir` (mirror)
- **`ipma` box** (item property association) — maps item IDs to property indices
- **`iloc` box** — item location; maps item IDs to byte ranges in `mdat`
- **`iinf` box** (item info) — item types and names
- **`iref` box** (item references) — links thumbnails, auxiliary images, and tiles to master items
- **`trak` box** with `pict` handler — image sequence tracks; timing, edit lists, looping, inter prediction constraints

## Code Highlights
No code samples were demonstrated in this session (it is a deep-dive format/specification talk). Key integration points are through Image I/O and AVFoundation:

```swift
// Reading a HEIC file
let source = CGImageSourceCreateWithURL(heicURL as CFURL, nil)!
let image = CGImageSourceCreateImageAtIndex(source, 0, nil)

// Checking for auxiliary depth image
let auxiliaryInfo = CGImageSourceCopyAuxiliaryDataInfoAtIndex(
    source, 0, kCGImageAuxiliaryDataTypeDisparity)
```

## Takeaways
- HEIF with HEVC delivers ~2x better compression than JPEG at equivalent quality; all iPhones running iOS 11 default to HEIF for still photos.
- The container's separation of codec from format is the key architectural advantage: HEIF can hold HEVC, H.264, JPEG, or future codecs in the same structural wrapper.
- Tiles enable efficient partial decoding for crop, zoom, and resize operations without loading entire multi-megapixel images into memory.
- Depth data (used by Portrait Mode), alpha images, and rich metadata all have well-defined homes inside a single HEIF file, replacing the fragmented approach required by JPEG.

---
_Source: WWDC17 Session 513 page (abstract, chapter summaries, code samples, and resource links)._
