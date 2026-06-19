# Meet the new Photos picker
**WWDC20 · Session 10652** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10652/)

_Platforms:_ iOS 14, iPadOS 14, Mac Catalyst

## Overview
`PHPickerViewController` is the modern, privacy-first replacement for `UIImagePickerController` (now deprecated for photo library access). The new picker runs out-of-process from the host app — it appears to be inside the app but actually renders in a separate system process — so the app cannot access or screenshot its contents. Only the assets the user explicitly selects are passed back to the app.

No photo library authorization prompt is ever shown to the user when presenting `PHPickerViewController`; the picker works without any `PHPhotoLibrary` access. Key improvements over `UIImagePickerController`: built-in search, pinch-to-zoom on the grid, multi-select with a review tray, and a clean consistent Photos-app-style UI.

## Key Topics

### Privacy Architecture
- Picker runs in a **separate process** from the host app; the app cannot inspect or screenshot picker content
- No `NSPhotoLibraryUsageDescription` permission prompt shown when presenting the picker
- Only the user-selected items are passed back — explicit user intent
- `UIImagePickerController` photo library functionality is **deprecated** in iOS 14

### PHPickerConfiguration
- `selectionLimit: Int` — `1` by default; set to `0` for unlimited selection
- `filter: PHPickerFilter?` — restricts which media types are shown:
  - `.images` — photos and Live Photos
  - `.videos` — videos and Live Photo video component
  - `.livePhotos` — only the Live Photo asset type
  - `.any(of: [PHPickerFilter])` — logical OR combination
- Initialize without a photo library for privacy-mode operation (no `PHAsset` identifiers returned)
- Initialize with `PHPickerConfiguration(photoLibrary: PHPhotoLibrary.shared())` to get `PHPickerResult.assetIdentifier` for PhotoKit integration (requires library access only if app subsequently fetches assets)

### PHPickerViewController
- Initialized with a `PHPickerConfiguration`
- Assign a delegate conforming to `PHPickerViewControllerDelegate`
- The **app is responsible** for presenting and dismissing the picker; the picker does not manage its own presentation
- Call `picker.dismiss(animated:)` at the start of the delegate callback (before loading images)

### PHPickerResult
- `itemProvider: NSItemProvider` — use to load the media in any supported format
  - `canLoadObject(ofClass:)` + `loadObject(ofClass:completionHandler:)` — async; always handle errors
  - Load as `UIImage`, `URL`, `Data`, etc. depending on the needed representation
- `assetIdentifier: String?` — populated only when picker was initialized with a `PHPhotoLibrary`; use with `PHAsset.fetchAssets(withLocalIdentifiers:options:)` for PhotoKit operations

### Multi-select Pattern
- Set `selectionLimit = 0` for unlimited selection
- Store all returned `[NSItemProvider]` references after dismissal; create an iterator
- Load images lazily from the iterator on demand (e.g., on tap) — avoids loading all images upfront
- Picker UI updates automatically for multi-select mode (staging area tray and Add button appear)

### PhotoKit Integration
- For apps that need `PHAsset` access (non-destructive editing, library organization):
  - Initialize: `PHPickerConfiguration(photoLibrary: PHPhotoLibrary.shared())`
  - Extract: `results.compactMap(\.assetIdentifier)`
  - Fetch: `PHAsset.fetchAssets(withLocalIdentifiers: identifiers, options: nil)`
- Still requires the user to grant photo library access before the app can work with `PHAsset` objects
- Limited Photos Library (new in iOS 14): if user selects "Select Photos" at the permission prompt, only a subset of `PHAsset`s is accessible; picker always shows the full library and all items can be selected

### Deprecations
- `AssetsLibrary` — deprecated years ago; migrate to PhotoKit
- `UIImagePickerController` for photo library access — **deprecated in iOS 14**; migrate to `PHPickerViewController`

## APIs & Frameworks

- **PhotosUI**
  - `PHPickerViewController` **[NEW]** — out-of-process system picker for photos and videos
  - `PHPickerConfiguration` **[NEW]** — configuration object:
    - `init()` — privacy mode; no `assetIdentifier` in results
    - `init(photoLibrary:)` **[NEW]** — PhotoKit mode; `assetIdentifier` populated in results
    - `selectionLimit: Int` **[NEW]** — `1` default; `0` = unlimited
    - `filter: PHPickerFilter?` **[NEW]** — restricts visible media types
  - `PHPickerFilter` **[NEW]** — struct with static properties:
    - `.images` — photos and Live Photos
    - `.videos` — videos
    - `.livePhotos` — Live Photos only
    - `.any(of:)` — combination filter
  - `PHPickerResult` **[NEW]** — per-selection result object:
    - `itemProvider: NSItemProvider` — loads the media data
    - `assetIdentifier: String?` — PhotoKit asset identifier (only in PhotoKit mode)
  - `PHPickerViewControllerDelegate` **[NEW]** — single required method:
    - `picker(_:didFinishPicking:)` — called with `[PHPickerResult]`; empty array when cancelled
- **PhotoKit**
  - `PHAsset.fetchAssets(withLocalIdentifiers:options:)` — fetch assets by identifier after PHPicker selection (requires library access)
  - `PHPhotoLibrary.shared()` — used to initialize PhotoKit-mode picker configuration
- **UIKit (deprecated)**
  - `UIImagePickerController` — photo library source type deprecated; migrate to `PHPickerViewController`

## Code Highlights

Single-image picker setup:
```swift
var configuration = PHPickerConfiguration()
configuration.filter = .images     // images only; selectionLimit defaults to 1

let picker = PHPickerViewController(configuration: configuration)
picker.delegate = self
present(picker, animated: true)
```

Delegate — loading a single UIImage:
```swift
func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
    picker.dismiss(animated: true)   // always dismiss first
    guard let itemProvider = results.first?.itemProvider,
          itemProvider.canLoadObject(ofClass: UIImage.self) else { return }
    itemProvider.loadObject(ofClass: UIImage.self) { [weak self] image, error in
        DispatchQueue.main.async {
            self?.imageView.image = image as? UIImage
        }
    }
}
```

Multi-image lazy loading:
```swift
// Configure unlimited selection
configuration.selectionLimit = 0

// In delegate
var itemProviders: [NSItemProvider] = []
var iterator: IndexingIterator<[NSItemProvider]>?

func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
    picker.dismiss(animated: true)
    itemProviders = results.map(\.itemProvider)
    iterator = itemProviders.makeIterator()
    displayNextImage()
}
```

PhotoKit mode — getting PHAsset identifiers:
```swift
let configuration = PHPickerConfiguration(photoLibrary: PHPhotoLibrary.shared())
// ...after picker delegate callback:
let identifiers = results.compactMap(\.assetIdentifier)
let fetchResult = PHAsset.fetchAssets(withLocalIdentifiers: identifiers, options: nil)
```

## Takeaways
- `PHPickerViewController` is the privacy-first replacement for `UIImagePickerController`; it requires no photo library authorization and passes only user-selected items back to the app.
- Most apps that only need image or video data should use `PHPickerViewController` without `PHPhotoLibrary` — no permission prompt, no access to the broader library.
- For multi-select, store all `NSItemProvider` references and load lazily; do not load all images at once after selection.
- Apps that need `PHAsset` access (non-destructive editing, library organization) should initialize `PHPickerConfiguration` with a `PHPhotoLibrary` instance to receive `assetIdentifier` values in results.

---
_Source: WWDC20 Session 10652 page (abstract, transcript, and code samples)._
