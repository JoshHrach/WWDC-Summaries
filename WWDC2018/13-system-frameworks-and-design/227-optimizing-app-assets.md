# Optimizing App Assets
**WWDC18 · Session 227** · [Watch](https://developer.apple.com/videos/play/wwdc2018/227/)

_Platforms:_ iOS 12, macOS Mojave 10.14, tvOS 12, watchOS 5

## Overview
This session covers best practices for authoring, organizing, and deploying artwork assets in Xcode Asset Catalogs to achieve smaller app sizes and faster runtime performance. Two significant new compression technologies are introduced: HEIF (High Efficiency Image File Format) becomes the default lossy compression option, and Apple Deep Pixel Image Compression is introduced as a new adaptive lossless algorithm — together delivering 10–20% size reductions across all first-party apps.

The session also introduces OS Variant App Thinning, which generates a dedicated iOS 12-optimized variant of the app even for apps that back-deploy to earlier OS versions. On the design/production side, the talk covers color management, working spaces, vector asset preservation, resizable image slicing, and performance-class (memory and GPU tier) asset routing using Asset Catalogs and `NSDataAsset`.

## Key Topics

### Image Compression

**Automatic Image Packing**
- Asset Catalog groups images sharing similar color-spectrum profiles into single atlases
- Eliminates per-file metadata duplication
- Delivers up to 80% size reduction for large collections of small images
- Scales with the number of assets — more assets, more benefit

**HEIF as Default Lossy Compression (iOS 12 / Xcode 10)** **[NEW]**
- HEIF replaces JPEG as the default lossy option in Asset Catalogs
- Better compression ratio than JPEG; supports transparency natively
- Asset Catalog compilation pipeline auto-converts source images to HEIF — no manual action required
- Best for artwork with short on-screen duration: splash screens, animations, effects

**Apple Deep Pixel Image Compression (iOS 12 / Xcode 10)** **[NEW]**
- New adaptive lossless compression algorithm that selects the optimal algorithm per image color spectrum
- Handles both simple artwork (narrow palette, discrete colors) and complex artwork (photos, gradients) efficiently
- ~20% average size reduction across all Apple platform apps
- ~20% improvement in decode time vs. previous lossless formats

### App Thinning Enhancements

**OS Variant Thinning (iOS 12 / Xcode 10)** **[NEW]**
- Previously: back-deploying to older OS versions disabled new compression for all users
- Now: generates a separate iOS 12-optimized variant (HEIF + Deep Pixel) for iOS 12 users while still serving compatible variants to older devices
- Enabled automatically when building with Xcode 10 and iOS 12 SDK
- Demonstrated via Xcode archive → Organizer → Ad Hoc export → App Thinning report

### Design and Production Best Practices

**Color Management**
- Always retain color profiles in source assets; never strip them — they encode designer intent
- Asset Catalog compilation performs color matching at build time, eliminating per-device runtime cost
- sRGB 8-bit: most common, broadly applicable
- Display P3 16-bit: recommended for wide-color, vibrant designs targeting P3 displays

**Resizable Images / Show Slicing Editor**
- Use a single image with slicing metadata rather than split assets reassembled with 9-part draw calls
- `Show Slicing` editor in Xcode: drag dividing lines to mark stretchable vs. fixed regions
- Xcode strips the non-needed center portion from the on-disk asset at build time
- Keeps stretching metadata co-located with artwork; survives design updates cleanly

**Vector Assets (PDF)**
- Supply a single PDF; Xcode rasterizes to 1x/2x/3x at build time
- `Preserve Vector Data` option (iOS 11+): re-rasterizes at runtime only when image view exceeds natural size — enables crisp Dynamic Type scaling
- Tip: design at 2x resolution on a 1-unit grid to eliminate sub-pixel stroke alignment problems; drop result into 2x Asset Catalog slot

### Cataloging and Organization

**Multi-Bundle Asset Namespacing**
- Build assets into separate framework/bundle targets for large projects with multiple teams
- Each bundle provides its own namespace; retrieve with `UIImage(named:in:compatibleWith:)` or `NSBundle.image(forResource:)`

**Folder Namespacing (Provide Namespace)**
- Check "Provides Namespace" on an asset folder to auto-prepend folder name to contained asset names
- Useful for large structured collections (e.g., 50 rooms each with table/chair assets)

**Sprite Atlases in Non-SpriteKit Apps**
- Group related images into a `Sprite Atlas` for controlled packing (unlike automatic packing, you control the group name)
- Access images via standard `UIImage`/`NSImage` APIs — no SpriteKit dependency required
- `SKTextureAtlas.preloadTextureAtlases(named:withCompletionHandler:)` — async preload into memory; use only when images are needed immediately

### Performance Classes (Memory and GPU Tier Asset Routing)

**Memory Classes**: 1 GB, 2 GB, 3 GB, 4 GB device tiers
**Graphics Classes**: Metal 1 (A7) through Metal 4 (A11)
- Combine both axes in the Asset Catalog capability matrix to route assets to specific hardware tiers
- Selection priority: memory class first, then graphics class
- `NSDataAsset` — arbitrary file container (video, plist, binary data) that participates in performance-class thinning
- Example use: ship a hi-res HDR cut-scene to 3 GB / Metal 3+ devices; ship a still image to low-tier devices
- Example use: plist with configuration tuning parameters per capability tier (crowd size, render settings)

## APIs & Frameworks

- **Asset Catalog compiler** (`actool`) — HEIF default lossy compression **[NEW]**, Apple Deep Pixel Image Compression lossless **[NEW]**
- **OS Variant Thinning** **[NEW]** — automatic iOS 12-optimized App Store variant generation
- **`UIImage(named:in:compatibleWith:)`** — retrieve images from a specific bundle with trait collection
- **`NSBundle` category image(forResource:)** — macOS equivalent for bundle-namespaced images
- **`NSDataAsset`** — typed container for arbitrary Asset Catalog data (images, video, plist, binary)
- **`SKTextureAtlas.preloadTextureAtlases(named:withCompletionHandler:)`** — async atlas preload
- **Asset Catalog Slicing Editor** ("Show Slicing") — graphical resizable image metadata editor
- **Asset Catalog Slicing Inspector** — numeric edge inset and tiling behavior controls
- **"Preserve Vector Data"** (iOS 11 / Xcode 9) — re-rasterizes PDF vectors at runtime when scaled beyond natural size
- **Performance Classes** in Asset Catalog — Memory (1/2/3/4 GB) and Graphics (Metal 1–4) routing axes
- **Sprite Atlas** in Asset Catalog — controlled image grouping with standard UIImage/NSImage access

## Code Highlights

Retrieving an asset from a custom bundle:
```swift
let bundle = Bundle(identifier: "com.example.ArtworkKit")
let image = UIImage(named: "hero-banner", in: bundle, compatibleWith: traitCollection)
```

Using NSDataAsset for performance-class-routed data:
```swift
// Asset Catalog: "CrowdConfig" NSDataAsset with 2GB/Metal3 and 4GB/Metal4 plists
if let asset = NSDataAsset(name: "CrowdConfig") {
    let config = try PropertyListDecoder().decode(CrowdConfig.self, from: asset.data)
    setCrowdSize(config.crowdSize)
}
```

Async atlas preload:
```swift
SKTextureAtlas.preloadTextureAtlases(named: ["GameUI"]) {
    DispatchQueue.main.async { self.startScene() }
}
```

## Takeaways
- Switching to Xcode 10 + iOS 12 SDK and using Asset Catalogs gives a free 10–20% size reduction via HEIF lossy and Apple Deep Pixel lossless compression — no code changes needed.
- OS Variant Thinning means back-deploying is no longer a penalty: iOS 12 users get the optimized variant while older devices receive compatible assets.
- Use single-image slicing metadata instead of multi-piece 9-part draw assembly — smaller on disk, faster at render time, and easier to maintain.
- Performance class routing via memory and GPU tiers enables content that scales gracefully from older devices to the latest hardware without runtime branching.

---
_Source: WWDC18 Session 227 page (abstract, chapter summaries, code samples, and resource links)._
