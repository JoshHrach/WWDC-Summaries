# What's new in the Photos picker
**WWDC22 · Session 10023** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10023/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9

## Overview
The system Photos picker (PHPicker) expanded significantly in 2022, gaining support for macOS and watchOS for the first time, a new SwiftUI API that works across all four platforms, and new filtering capabilities. The picker continues to run out-of-process so apps never need to request photo library access, preserving user privacy.

On iPad, the picker was redesigned with a sidebar for faster collection navigation. On macOS, a native AppKit `PHPickerViewController` (`NSViewController` subclass) is now available, and the system Photos picker UI is also integrated directly into `NSOpenPanel` at no adoption cost. watchOS gains its first-ever out-of-process picker for images stored on the watch, supporting browsing by grid or collection.

The SwiftUI `PhotosPicker` API is the recommended cross-platform entry point, built on top of the `Transferable` protocol for flexible data loading. It automatically adapts its presentation to the platform and available screen space.

## Key Topics

### New PHPickerFilter Options
New filter types for screenshots, screen recordings, slo-mo videos, cinematic videos, depth effect photos, and bursts. New compound operators `.all(of:)` and `.not(_:)` complement the existing `.any(of:)`. Most new filters are back-deployed to iOS 15.

### iPadOS Redesign
The iPadOS picker now shows a sidebar for fast navigation between collections. Falls back to the compact picker UI in Split Screen or when space is limited.

### macOS Support (New)
`PHPickerViewController` is now an `NSViewController` subclass for AppKit apps. Replaces the legacy `PHMediaLibrary` browser. Also integrated into `NSOpenPanel` — apps get the new picker UI automatically with no code changes.

### watchOS Support (New)
New out-of-process picker for images on Apple Watch. Shows a grid and collection browsing UI optimized for the smaller screen. Supports selection order display, selection limits, and up to 500 most-recent images (or 1,000 for Family Setup devices via iCloud Photos).

### SwiftUI PhotosPicker API
`PhotosPicker` view available on iOS, iPadOS, macOS, and watchOS. Accepts a `Binding` to `[PhotosPickerItem]` and a filter. Uses `Transferable` to load asset data. Supports loading as `SwiftUI.Image`, custom `Transferable` types, or files via `FileTransferRepresentation` for memory-efficient large asset handling.

### PHPickerViewController Improvements
`deselectAssets(withIdentifiers:)` — programmatic deselection by asset identifier.
`moveAsset(withIdentifier:afterAssetWithIdentifier:)` — programmatic reordering.

## APIs & Frameworks

**PhotoKit / PHPicker**
- `PHPickerConfiguration` — shared across UIKit, AppKit, SwiftUI
- `PHPickerViewController` (UIKit — `UIViewController`) — unchanged
- `PHPickerViewController` (AppKit — `NSViewController`) **[NEW]**
- `PHPickerFilter.screenshots` **[NEW]**
- `PHPickerFilter.screenRecordings` **[NEW]**
- `PHPickerFilter.slomoVideos` **[NEW]**
- `PHPickerFilter.cinematicVideos` **[NEW]** (iOS 16+ only, not back-deployed)
- `PHPickerFilter.depthEffectPhotos` **[NEW]**
- `PHPickerFilter.bursts` **[NEW]**
- `PHPickerFilter.all(of:)` **[NEW]** — compound "and" filter (back-deployed iOS 15)
- `PHPickerFilter.not(_:)` **[NEW]** — compound "not" filter (back-deployed iOS 15)
- `PHPickerFilter.any(of:)` — existing compound "or" filter
- `PHPickerFilter(playbackStyle:)` **[NEW]** — filter by `PHAsset.PlaybackStyle`
- `PHPickerViewController.deselectAssets(withIdentifiers:)` **[NEW]**
- `PHPickerViewController.moveAsset(withIdentifier:afterAssetWithIdentifier:)` **[NEW]**
- `PHPickerResult` — returned selected item result
- `NSItemProvider` — used with UIKit/AppKit picker results

**SwiftUI PhotosUI Framework**
- `PhotosPicker` **[NEW]** — cross-platform SwiftUI picker view (iOS, iPadOS, macOS, watchOS)
- `PhotosPickerItem` **[NEW]** — placeholder for a selected photo/video
- `PhotosPickerItem.loadTransferable(type:completionHandler:)` **[NEW]**
- `PhotosPicker(selection:matching:photoLibrary:)` — initializer with filter
- `PhotosPicker(selection:maxSelectionCount:matching:photoLibrary:)` — with limit
- `PHPickerFilter` — used as the `matching` parameter in SwiftUI

**Transferable Protocol (SwiftUI)**
- `Transferable` — protocol for type-safe data transfer
- `FileTransferRepresentation` — loads asset as a file for memory-efficient handling
- `DataRepresentation` — loads asset as `Data`

**AppKit Integration**
- `NSOpenPanel` — now shows system Photos picker UI automatically **[NEW behavior]**

## Code Highlights

New compound PHPickerFilter usage:
```swift
// Show images excluding screenshots
configuration.filter = .all(of: [.images, .not(.screenshots)])

// Show cinematic videos (iOS 16+)
configuration.filter = .cinematicVideos
```

SwiftUI PhotosPicker with Transferable loading:
```swift
struct ContentView: View {
    @Binding var selection: [PhotosPickerItem]

    var body: some View {
        PhotosPicker(selection: $selection, matching: .images) {
            Text("Select Photos")
        }
    }
}

// Loading selected item as a Transferable type:
item.loadTransferable(type: Image.self) { result in
    switch result {
    case .success(let image): imageState = .success(image)
    case .failure(let error): imageState = .failure(error)
    }
}
```

## Takeaways
- `PHPicker` now runs on iOS, iPadOS, macOS, and watchOS — a single API covers all platforms, with SwiftUI's `PhotosPicker` as the easiest cross-platform entry point.
- All apps using `NSOpenPanel` automatically get the new macOS Photos picker UI with no code changes.
- New `PHPickerFilter` compound operators (`.all(of:)`, `.not(_:)`) and new asset-type filters (screenshots, cinematic, etc.) are back-deployed to iOS 15.
- Use `FileTransferRepresentation` to load large video assets as files rather than in-memory data to avoid excessive memory usage.

---
_Source: WWDC22 Session 10023 page (abstract, chapter summaries, code samples, and resource links)._
