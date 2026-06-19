# Discover Quick Look for Spatial Computing
**WWDC23 · Session 10085** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10085/)

_Platforms:_ visionOS 1

## Overview
Youssef from the AR Quick Look team explains how Quick Look works on visionOS, introducing a new presentation style called Windowed Quick Look alongside the existing in-app preview model. Quick Look on visionOS supports 3D USDZ content with volumetric viewing, spatial images and video, PDFs, and common image/video formats. Existing iOS Quick Look adoptions and websites with AR content links get most of this behavior for free.

The session covers two distinct modes: *Windowed Quick Look* (content previewed in a separate system window outside any app) and *in-app Quick Look* (previewed as a sheet inside your app's scene). Both share the same framework; the difference is only in how they're presented.

## Key Topics

### Windowed Quick Look **[NEW on visionOS]**
A new presentation style unique to visionOS. Files are previewed in a separate system-hosted window that is independent of the presenting app. Key characteristics:
- The presenting app can be closed while the Quick Look window persists.
- Multiple Windowed Quick Look windows can be open simultaneously alongside the app.
- 3D USDZ and Reality files get a **volumetric immersive viewing experience**: the user can scale up a model, which dims the environment and hides all other scenes for full focus on the 3D content.
- **SharePlay integration**: USDZ/Reality previews sync model orientation, scale, and animations across FaceTime participants; image previews enable collaborative Markup over SharePlay.

**How apps trigger Windowed Quick Look**: Provide a drag source (`onDrag` modifier with `NSItemProvider` containing a `URL`). When the user drops the file outside any app window, the system copies and presents it in a Windowed Quick Look preview. Remote URLs (e.g., iCloud) are downloaded first. Because a copy is used, edits (Markup, etc.) in the window are not written back to the source URL.

**How websites trigger Windowed Quick Look**: Mark anchor tags with `rel="ar"`. Safari on visionOS opens USDZ links in a Windowed Quick Look window alongside the webpage — identical to AR Quick Look on iOS/iPadOS, requiring no additional work.

### In-App Quick Look (Unchanged from iOS)
Present file previews as a sheet inside your app using the SwiftUI `quickLookPreview` modifier or `QLPreviewController`. Existing iOS adoption carries over to visionOS with no changes. The `.quickLookPreview(_:in:)` modifier takes a binding to the currently selected URL and an array of all URLs to cycle through.

### Supported File Types on visionOS
- **3D**: USDZ, Reality (`.reality`)
- **Spatial**: Spatial images, spatial video
- **Standard**: Images (JPEG, PNG, HEIC, etc.), video (MP4, MOV, etc.), PDF
- All benefit from system-provided trimming (video), Markup (PDF, image), and SharePlay without any app code.

### Website AR Content Linking (Carried Over)
Websites that added `rel="ar"` for iOS AR Quick Look get the same behavior on visionOS Safari for free. Existing customization options from WWDC19 ("Advances in AR Quick Look") such as disabling content scaling are also respected.

## APIs & Frameworks

### QuickLook Framework
- `QLPreviewController` — UIKit controller for presenting file previews in-app; works on visionOS **[NEW platform]**
- `QLPreviewControllerDataSource` — data source for `QLPreviewController`

### SwiftUI
- `.quickLookPreview(_:in:)` — presents a Quick Look preview sheet for the selected URL within an array of previewable URLs **[NEW modifier form; new on visionOS]**
- `.quickLookPreview(_:)` — simpler form presenting a single URL

### UIKit / AppKit
- `.onDrag { NSItemProvider(contentsOf: url) }` — makes a view a drag source providing a file URL; dropping outside any app triggers Windowed Quick Look **[NEW behavior on visionOS]**
- `NSItemProvider(contentsOf:)` — wraps a file URL for drag-and-drop

### Web / Safari
- HTML `rel="ar"` attribute on `<a>` / `<link>` tags — opens USDZ links in a Quick Look window on visionOS Safari (same as on iOS/iPadOS)

### Windowed Quick Look **[NEW on visionOS]**
- System-presented window hosting a Quick Look preview outside any app process
- Triggered by drag-and-drop from an app or by tapping an `rel="ar"` link in Safari
- SharePlay sessions for USDZ content (orientation/scale/animation sync) and images (collaborative Markup) — handled automatically by the system

## Code Highlights

```swift
// App: add drag support so files can be dragged out to a Quick Look window
import SwiftUI
import UniformTypeIdentifiers

struct FileList: View {
    @State var files: [File]

    var body: some View {
        List(files) { file in
            Button(file.name) { /* open */ }
                .onDrag {
                    // Returning an NSItemProvider with a URL triggers
                    // Windowed Quick Look when dropped outside any app
                    return NSItemProvider(contentsOf: file.url) ?? NSItemProvider()
                }
        }
    }
}
```

```swift
// App: in-app Quick Look preview sheet
import SwiftUI
import QuickLook

struct ReviewView: View {
    @State var files: [File]
    @State var previewedURL: URL?

    var body: some View {
        List(files) { file in
            Button(file.name) { previewedURL = file.url }
        }
        .quickLookPreview($previewedURL, in: files.map { $0.url })
    }
}
```

```html
<!-- Website: link USDZ for Windowed Quick Look in Safari on visionOS -->
<a href="model.usdz" rel="ar">
    <img src="model-preview.jpg" alt="View in 3D">
</a>
```

## Takeaways
- Windowed Quick Look is the preferred content preview pattern on visionOS: it lets users view files alongside your app or keep previews open after closing the app.
- Add drag support with `.onDrag { NSItemProvider(contentsOf: url) }` to enable Windowed Quick Look from your app; that's the only code change needed.
- Websites already using `rel="ar"` links get Windowed Quick Look in visionOS Safari automatically — no server changes needed.
- Use `.quickLookPreview(_:in:)` for in-app review workflows where a modal sheet is appropriate; existing iOS code carries over unchanged.

---
_Source: WWDC23 Session 10085 page (abstract, chapter summaries, code samples, and resource links)._
