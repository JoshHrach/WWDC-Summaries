# AR Quick Look, Meet Object Capture
**WWDC21 · Session 10078** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10078/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session bridges two complementary technologies: Object Capture (new in 2021), which generates photorealistic USDZ 3D models from photographs, and AR Quick Look, the system-wide AR viewer built into iOS. Together with Reality Composer for adding interactivity, these three tools form a complete pipeline for creating and distributing augmented reality content without requiring specialized 3D modeling skills or expensive software.

The session covers best practices for selecting Object Capture detail settings, a practical workflow demo (succulent/pottery e-commerce app), AR Quick Look integration for both apps and websites, and real-world use cases across e-commerce, museums, and education.

## Key Topics

### Object Capture → AR Quick Look Pipeline
1. **Photograph** the object from all angles on a neutral background (minimum 70% overlap between adjacent photos, fill the frame)
2. **Generate USDZ** using the Object Capture API (new in RealityKit, macOS Monterey)
3. Optionally **add interactivity** in Reality Composer (tap triggers, camera actions, audio, show/hide behaviors)
4. **Distribute** via AR Quick Look in app or on the web

### Object Capture Detail Settings for AR Quick Look
| Setting | Characteristics | Recommended Use |
|---|---|---|
| **Reduced** | Smallest file size, fastest download | Web distribution, multi-asset scenes, most use cases |
| **Medium** | Larger file, better fidelity | Complex objects, hundreds of images, app-local assets |

- Evaluate both Reduced and Medium before choosing — test on a variety of iOS hardware
- Tradeoff: visual fidelity vs. file size / download time
- Always provide sharp images, avoid motion blur, ensure adequate photo overlap

### Reality Composer Integration
- Combine multiple Object Capture USDZ assets in one scene
- Add tap triggers, "hide on start", "show/hide" actions to create interactive product viewers
- Add voiceover audio tracks for accessibility and narration
- Export the combined scene as a USDZ for AR Quick Look

### AR Quick Look Integration in Apps
- Use `QLPreviewController` with `ARQuickLookPreviewItem`
- `ARQuickLookPreviewItem(fileAt:)` — takes a local file URL to the USDZ
- `allowsContentScaling` — set to `false` to show object at true real-world scale (no pinch-to-scale)

### AR Quick Look Integration on Websites
- HTML `<a>` tag with `rel="ar"` attribute and a link to the `.usdz` file
- Set `allowsContentScaling=0` query parameter to disable scaling
- Apple Pay and custom action buttons can be surfaced directly in the AR Quick Look web view

### Real-World Use Cases
- **E-commerce**: product visualization, variant swapping (different colors/styles via tap), true-scale furniture placement
- **Museums**: 360-degree viewing of artifacts in protective cases, remote exhibits, voiceover audio narration
- **Education**: 3D visualization of concepts, student-created interactive models (e.g., Qlone app integration)

## APIs & Frameworks

### RealityKit / Object Capture
- `PhotogrammetrySession` — Object Capture API for generating USDZ from photos
- `PhotogrammetrySession.Request.Detail.reduced` — smallest file, recommended for web
- `PhotogrammetrySession.Request.Detail.medium` — better quality for complex objects
- (Described conceptually; detailed API covered in WWDC21 "Create 3D models with Object Capture")

### QuickLook / ARKit
- `QLPreviewController` — system view controller for Quick Look presentations
- `QLPreviewControllerDataSource` protocol
  - `numberOfPreviewItems(in:)` — return count
  - `previewController(_:previewItemAt:)` — return `QLPreviewItem`
- `ARQuickLookPreviewItem(fileAt:)` — wraps a local USDZ file URL
  - `allowsContentScaling: Bool` — `false` = true-to-life scale, no user scaling
- Web HTML integration: `<a href="model.usdz" rel="ar"><img src="thumb.jpg"/></a>`
  - Query param: `allowsContentScaling=0`
  - Apple Pay integration: `callToAction`, `checkoutTitle`, `checkoutSubtitle`, `price` query params

### Reality Composer
- Multi-asset scene composition (combine multiple USDZ files)
- Behavior system: tap triggers, hide/show actions, camera actions, audio playback
- Export to USDZ for AR Quick Look

## Code Highlights

Presenting AR Quick Look in an app:
```swift
func presentARQuickLook() {
    let previewController = QLPreviewController()
    previewController.dataSource = self
    present(previewController, animated: true)
}

func previewController(_ controller: QLPreviewController, previewItemAt index: Int) -> QLPreviewItem {
    let previewItem = ARQuickLookPreviewItem(fileAt: fileURL)
    previewItem.allowsContentScaling = false
    return previewItem
}
```

Web integration (HTML):
```html
<a href="model.usdz" rel="ar">
    <img src="thumbnail.jpg">
</a>
```

## Takeaways
- Object Capture + Reality Composer + AR Quick Look form a complete, no-3D-expertise-required pipeline for creating and distributing AR content.
- The "Reduced" detail setting is recommended for most use cases — it balances visual quality with the smaller file sizes needed for fast web downloads.
- AR Quick Look integration for apps requires just a few lines of code using `QLPreviewController` and `ARQuickLookPreviewItem`.
- The `allowsContentScaling = false` property is critical for product visualization use cases where accurate real-world scale matters.

---
_Source: WWDC21 Session 10078 page (abstract, chapter summaries, code samples, and resource links)._
