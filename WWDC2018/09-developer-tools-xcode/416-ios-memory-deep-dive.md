# iOS Memory Deep Dive
**WWDC18 · Session 416** · [Watch](https://developer.apple.com/videos/play/wwdc2018/416/)

_Platforms:_ iOS, macOS, watchOS, tvOS

## Overview
This session provides a thorough treatment of iOS memory: how the kernel categorizes pages (clean, dirty, compressed), what counts against your app's memory footprint, why iOS terminates apps based on memory pressure rather than swap, and how to profile and fix real memory problems. A major portion covers image memory — the single largest source of unexpected footprint — and demonstrates ImageIO-based downsampling as the correct alternative to `UIImage(named:)` for large images. The session also covers tools: Xcode Memory Debugger, Instruments Allocations and Leaks, and the `vmmap` command-line tool.

## Key Topics

### Memory Categories

- **Clean memory** — pages that have not been written since being mapped: the executable binary, frameworks (`.dylib`), memory-mapped read-only files, and zero pages not yet touched. The system can evict these pages and reload them from disk on demand without a performance penalty.
- **Dirty memory** — pages that have been written by the app: heap allocations (`malloc`), stack, global variables, and Core Data caches. Cannot be evicted without terminating the app.
- **Compressed memory** (since iOS 7) — dirty pages the kernel has compressed (typically 2:1 to 4:1 ratio) to free physical RAM. Compressed pages are decompressed on access. Counts against the footprint.
- **Memory footprint = dirty memory + compressed memory.** The kernel limits each app's footprint; exceeding it triggers a `jetsam` termination even if free RAM is available elsewhere.
- There is no disk swap on iOS — jetsam is the only pressure relief valve. Apps are terminated in order of footprint size; your app can be terminated even if the foreground app triggered the pressure.

### Why `UIImage(named:)` Is Expensive

- `UIImage(named:)` caches the decoded image in the image cache — useful for icons reused across the app, but wasteful for large images loaded once.
- JPEG/PNG files on disk are compressed; when decoded to a pixel buffer for display, a 4000×3000 JPEG can produce a 48 MB uncompressed BGRA buffer (4 bytes × 12M pixels), all dirty memory.
- Simply loading a large image with `UIImage(named:)` or `UIImage(contentsOfFile:)` adds the full decoded size to the footprint immediately on creation of the `CGContext` for display.

### Image Resizing with ImageIO (Correct Approach)

- Use `CGImageSourceCreateWithURL(_:_:)` to open the image source without decoding.
- Use `CGImageSourceCreateThumbnailAtIndex(_:_:_:)` with options:
  - `kCGImageSourceThumbnailMaxPixelSize` — constrain the decoded size to the display size.
  - `kCGImageSourceCreateThumbnailFromImageAlways: true`
  - `kCGImageSourceShouldCacheImmediately: true` — decode on the current thread rather than on first display.
  - `kCGImageSourceShouldCache: false` — do not keep a second copy in the system image cache.
- This approach decodes only the pixels needed for the display size; a 4000×3000 image shown in a 300×300 point view at 3× generates a 2.6 MB buffer instead of 48 MB.

### UIImage Format and Background Rendering

- `UIGraphicsImageRenderer` (iOS 10+) uses automatic format selection: `.automatic` picks RGBA/grayscale/wide based on content, avoiding unnecessary alpha channels and color-space overhead.
- `UIGraphicsImageRendererFormat.preferred()` — starts from the optimal format for the main screen.
- `UIImage.prepareForDisplay(completionHandler:)` (iOS 15+, mentioned as aspirational) — async background decode; not available in iOS 12 but the `preferredBackgroundFormat` and `preparingForDisplay()` patterns were being discussed at WWDC18 as upcoming API.

### Memory Profiling Tools

**Xcode Memory Debugger**
- Enable via Debug → Memory Graph (the "square with three circles" toolbar button during a live debug session).
- Shows all heap objects with their retain graph. Select an object to see what's holding it.
- Export `.memgraph` file for offline analysis with command-line tools.

**Instruments Allocations**
- Records all `malloc`/`free` events and heap allocations.
- Generation analysis: take a snapshot (mark a generation), perform an action, take another snapshot, then diff to see what was allocated and not freed. Identifies retain cycles and accidental retain.
- "All Heap & Anonymous VM" filter shows both heap allocations and VM regions (including image buffers).

**Instruments Leaks**
- Detects cycles in the retain graph at runtime.
- Run alongside Allocations for a complete picture: Leaks finds definite leaks; Allocations finds growing but not leaked memory.

**vmmap (command-line)**
- `vmmap --summary <pid or .memgraph>` — shows virtual memory regions grouped by type (MALLOC, mapped files, __TEXT, __DATA, etc.) with dirty and swapped/compressed sizes per region.
- `vmmap --verbose <pid>` — per-region breakdown useful for identifying large anonymous VM allocations (often image decoder buffers).
- Combine with `heap <pid>` to inspect individual allocation sites.

### NSCache and Purgeable Memory

- `NSCache` automatically evicts objects under memory pressure — use it for decoded image caches instead of `NSDictionary`.
- `NSPurgeableData` / `NSDiscardableContent` — mark memory as purgeable; the system can discard it under pressure without your involvement. `beginContentAccess()` / `endContentAccess()` gate access; `isContentDiscarded` checks whether it was dropped.
- Do not store decoded image pixel buffers in plain `Dictionary` or `Array` — they will not be evicted.

### Memory Warnings and Background

- `applicationDidReceiveMemoryWarning(_:)` and `UIViewController.didReceiveMemoryWarning()` — respond by releasing caches, unloading off-screen view controllers, and dropping `NSCache` entries.
- Background apps can also be terminated for memory; release as much as possible in `applicationDidEnterBackground(_:)`.
- On watchOS: memory limits are very tight (~30 MB watchOS 4); minimize image pre-caching.

## APIs & Frameworks

**ImageIO**
- `CGImageSourceCreateWithURL(_:_:)` — open image source (no decode)
- `CGImageSourceCreateWithData(_:_:)` — open from in-memory data
- `CGImageSourceCreateThumbnailAtIndex(_:_:_:)` — decode at reduced size; key options: `kCGImageSourceThumbnailMaxPixelSize`, `kCGImageSourceCreateThumbnailFromImageAlways`, `kCGImageSourceShouldCacheImmediately`, `kCGImageSourceShouldCache`
- `CGImageSourceCopyPropertiesAtIndex(_:_:_:)` — read image dimensions without decoding

**UIKit**
- `UIGraphicsImageRenderer` — automatic format (`UIGraphicsImageRendererFormat.automatic`), `preferred()` factory
- `UIGraphicsImageRendererFormat` — `preferredRange`, `opaque`, `scale`
- `UIImage(contentsOfFile:)` — loads without caching (use for large images shown once)
- `UIImage(named:)` — loads with caching (use for small, frequently reused images)

**Foundation**
- `NSCache` — evictable cache; `totalCostLimit`, `countLimit`, `delegate` (`NSCacheDelegate`)
- `NSPurgeableData` — purgeable byte buffer
- `NSDiscardableContent` — protocol for purgeable objects; `beginContentAccess()`, `endContentAccess()`, `discardContentIfPossible()`, `isContentDiscarded`

**Developer Tools**
- Xcode Memory Graph Debugger — live retain graph, `.memgraph` export
- Instruments Allocations — generation diff, heap growth analysis
- Instruments Leaks — retain cycle detection
- `vmmap --summary [pid|.memgraph]` — VM region summary
- `heap [pid|.memgraph]` — per-allocation-site breakdown
- `leaks [pid|.memgraph]` — command-line leak detection

## Code Highlights

Downsampling a large image for display using ImageIO:
```swift
func downsampledImage(at url: URL, to pointSize: CGSize, scale: CGFloat) -> UIImage {
    let imageSourceOptions = [kCGImageSourceShouldCache: false] as CFDictionary
    let imageSource = CGImageSourceCreateWithURL(url as CFURL, imageSourceOptions)!

    let maxDimension = max(pointSize.width, pointSize.height) * scale
    let downsampleOptions = [
        kCGImageSourceCreateThumbnailFromImageAlways: true,
        kCGImageSourceShouldCacheImmediately: true,
        kCGImageSourceCreateThumbnailWithTransform: true,
        kCGImageSourceThumbnailMaxPixelSize: maxDimension
    ] as CFDictionary

    let downsampledImage = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, downsampleOptions)!
    return UIImage(cgImage: downsampledImage)
}
```

Background image decoding to avoid main-thread hitches:
```swift
DispatchQueue.global(qos: .userInitiated).async {
    let image = self.downsampledImage(at: photoURL, to: cell.imageView.bounds.size,
                                      scale: UIScreen.main.scale)
    DispatchQueue.main.async {
        cell.imageView.image = image
    }
}
```

Using NSCache for decoded images:
```swift
private let imageCache = NSCache<NSURL, UIImage>()
imageCache.totalCostLimit = 50 * 1024 * 1024  // 50 MB

func cachedImage(for url: URL) -> UIImage? {
    return imageCache.object(forKey: url as NSURL)
}
func cache(_ image: UIImage, for url: URL) {
    let cost = Int(image.size.width * image.size.height * image.scale * image.scale * 4)
    imageCache.setObject(image, forKey: url as NSURL, cost: cost)
}
```

## Takeaways
- Memory footprint = dirty + compressed memory. There is no swap on iOS; exceeding the limit causes immediate termination even if other memory is free.
- `UIImage(named:)` decodes the full pixel buffer into dirty memory the moment the image is drawn — a 4000×3000 photo = 48 MB. Always downsample to display size using ImageIO's `CGImageSourceCreateThumbnailAtIndex` before decoding.
- Use `NSCache` instead of `Dictionary` for any decoded image or data cache so the system can evict it under memory pressure without terminating the app.
- Use the Xcode Memory Graph Debugger's `.memgraph` export with `vmmap --summary` to identify which VM regions are growing and where dirty memory originates.

---
_Source: WWDC18 Session 416 page (abstract, full transcript, and resource links)._
