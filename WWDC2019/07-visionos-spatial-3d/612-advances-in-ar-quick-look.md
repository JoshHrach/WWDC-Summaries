# Advances in AR Quick Look
**WWDC19 · Session 612** · [Watch](https://developer.apple.com/videos/play/wwdc2019/612/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
AR Quick Look is the system-level built-in viewer for experiencing 3D and augmented reality content on iOS, iPadOS, and macOS. In iOS 13, it receives a major expansion driven by two forces: deep integration with Reality Composer's new `.reality` file format, and a set of new rendering effects (ray-traced shadows, camera grain, HDR tone-mapping, people occlusion, depth of field, motion blur) that make virtual content far more believable in the real world.

The session also covers significant experience improvements — vertical surface support, face anchors for try-on, image anchors, multi-model nested USDZ files, an animation scrubber, levitation gesture, and video recording. For developers, new web and iOS APIs enable customization of scaling behavior and sharing URLs, and a new retail integration adds Apple Pay call-to-action support directly within the viewer.

## Key Topics

### Reality File Support **[NEW]**
- AR Quick Look now displays `.reality` files authored in Reality Composer alongside USDZ.
- Reality files are self-contained: multiple scenes, anchor type metadata, interactive behaviors (triggers + actions), audio, physics — all in one file.
- Behaviors driven from Reality Composer (tap triggers, proximity triggers, scene start triggers, move/show/hide/look-at/sound actions) play natively in AR Quick Look.
- AR Quick Look does **not** support custom app actions (those require a host app using RealityKit).

### New Anchor Types
- **Vertical surfaces** — walls and other vertical planes detected via ML-enhanced plane detection.
- **Face anchors** — front TrueDepth camera; content (glasses, masks) follows the user's face; face occlusion geometry hides intersecting parts. Up to 3 faces (ARKit 3). Falls back to horizontal if no TrueDepth camera.
- **Image anchors** — content tracks a known real-world 2D image; falls back to plane placement if the image is not visible; gestures (scale/rotate) disabled to maintain authored size.
- Horizontal/vertical anchor transitions: drag object from floor to wall seamlessly.

### Rendering Improvements
- **Dynamic ray-traced shadows** **[NEW]**: real-time, device-adaptive (projective on all supported devices; ray-traced on iPhone XS and later for softer, physically correct shadows). No need to bake shadows into models.
- **Camera grain** **[NEW]**: ARKit noise texture applied to virtual content to match low-light camera noise. All ARKit devices.
- **HDR with tone mapping** **[NEW]**: 16-bit per channel rendering, custom tone-mapping curve prevents blown-out highlights. A10X devices and later.
- **People occlusion** **[NEW]**: uses ARKit 3 person segmentation (Apple Neural Engine) to render people in front of virtual objects. A12 devices and later.
- **Depth of field** **[NEW]**: virtual objects blur at distances other than the camera focus distance. A12X devices.
- **Motion blur** **[NEW]**: device motion + frame interpolation applied to fast-moving virtual content. A12X devices.

### Viewing Experience Improvements
- Launches directly into AR (no 2D preview step).
- Faster plane detection using ML.
- **Multi-model nested USDZ** **[NEW]**: a single USDZ wrapping multiple USDZ files (metadata + library), displayed as a collection; models sorted by height and manipulated independently.
- **Animation scrubber** **[NEW]**: play/pause/rewind/scrub for USDZ animations ≥10s; triggered for reality files with a single on-start trigger + animation action.
- **Levitation gesture** **[NEW]**: two-finger swipe up to lift an object off its surface.
- **Video recording** **[NEW]**: tap and hold shutter button to record what you see; saved to Photos library.
- macOS Quick Look Viewer: now supports `.reality` files; built on RealityKit for consistent PBR rendering; generates thumbnails.

### Web Integration
- MIME type for USDZ: `model/vnd.usdz+zip`; MIME type for reality files: `model/vnd.reality`.
- HTML: `<a rel="ar" href="model.usdz"><img src="thumb.jpg"></a>` pattern unchanged.
- **Data URIs and Blob URLs** **[NEW]**: pass base64-encoded USDZ/reality data or blob URLs directly; same MIME type prefix.
- **URL fragment identifier customization** **[NEW]**:
  - `#allowsContentScaling=0` — disables pinch-to-scale (for known-size retail objects)
  - `#canonicalWebPageURL=https://...` — overrides the URL shared via iOS share sheet

### iOS App API **[NEW]**
- `ARQuickLookPreviewItem` **[NEW]** (ARKit framework, conforms to `QLPreviewItem`): initialized with a file URL; exposes `canonicalWebPageURL` and `allowsContentScaling` properties.
- Present via `QLPreviewController` with `QLPreviewControllerDataSource` returning `ARQuickLookPreviewItem`.

### Retail / Apple Pay Integration **[NEW, available later iOS 13]**
- Custom call-to-action banner at the bottom of the AR Quick Look viewer: title, subtitle, domain name, and an Apple Pay button.
- Users can purchase products directly without leaving the AR viewer.

## APIs & Frameworks

**ARKit / Quick Look**
- `ARQuickLookPreviewItem` **[NEW]** — `init(fileAt:)`, `canonicalWebPageURL: URL?`, `allowsContentScaling: Bool`
- `QLPreviewController` — presents `ARQuickLookPreviewItem` (unchanged API, new item type)
- `QLPreviewControllerDataSource` — `previewController(_:previewItemAt:)` returning `ARQuickLookPreviewItem`

**RealityKit (rendering back-end for AR Quick Look)**
- Dynamic ray-traced shadows (A12+ devices for full ray-trace; projective for others) **[NEW]**
- Camera grain via `ARFrame.cameraGrainTexture` / `ARFrame.cameraGrainIntensity` **[NEW]**
- HDR rendering pipeline (16-bit, tone-mapping) **[NEW]**
- People occlusion via `ARBodyTrackingConfiguration` / person segmentation **[NEW]**
- Depth of field **[NEW]**
- Motion blur **[NEW]**

**Reality Composer**
- `.reality` file format — scenes, anchors, behaviors, audio, physics **[NEW]**
- Anchor types: `.horizontal`, `.vertical` **[NEW]**, `.face` **[NEW]**, `.image` **[NEW]**
- Triggers: `OnTap`, `OnProximity`, `OnSceneStart` **[NEW]**
- Actions: `Move`, `Show`, `Hide`, `LookAt`, `PlaySound`, `Wait` **[NEW]**

**Web (HTML/HTTP)**
- `rel="ar"` on `<a>` element (iOS 12, unchanged)
- MIME type: `model/vnd.usdz+zip` (unchanged); `model/vnd.reality` **[NEW]**
- Fragment identifier params: `allowsContentScaling`, `canonicalWebPageURL` **[NEW]**
- Data URI / Blob URL support **[NEW]**

## Code Highlights

```swift
// iOS app: presenting an AR Quick Look item with customization
import ARKit
import QuickLook

class ViewController: QLPreviewControllerDataSource {
    func showARQuickLook(for url: URL, productPageURL: URL) {
        let item = ARQuickLookPreviewItem(fileAt: url)
        item.canonicalWebPageURL = productPageURL
        item.allowsContentScaling = false  // for known-size furniture/products
        
        let previewController = QLPreviewController()
        previewController.dataSource = self
        present(previewController, animated: true)
    }
    
    func numberOfPreviewItems(in controller: QLPreviewController) -> Int { 1 }
    func previewController(_ controller: QLPreviewController,
                           previewItemAt index: Int) -> QLPreviewItem { arItem }
}
```

```html
<!-- Web: AR Quick Look with customization via fragment identifier -->
<a rel="ar" href="chair.usdz#allowsContentScaling=0&canonicalWebPageURL=https://shop.example.com/chair">
  <img src="chair-thumb.jpg">
</a>

<!-- Reality file on web -->
<a rel="ar" href="scene.reality">
  <img src="scene-thumb.jpg">
</a>
```

## Takeaways
- AR Quick Look in iOS 13 is no longer just a static model viewer — via `.reality` files, it becomes a full interactive AR experience player with behaviors, audio, and physics.
- All major rendering improvements (ray-traced shadows, people occlusion, HDR, depth of field, motion blur) are automatic; no code changes needed — AR Quick Look selects the right tier per device.
- The new `ARQuickLookPreviewItem` iOS API and URL fragment identifier web API use exactly the same two customization flags (`allowsContentScaling`, `canonicalWebPageURL`), providing a unified story for web and native.
- The Apple Pay / call-to-action retail integration makes AR Quick Look a direct purchase surface, enabling "try before you buy" commerce flows entirely within the system viewer.

---
_Source: WWDC19 Session 612 page (abstract, chapter summaries, code samples, and resource links)._
