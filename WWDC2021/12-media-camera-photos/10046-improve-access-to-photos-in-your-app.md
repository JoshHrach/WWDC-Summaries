# Improve Access to Photos in Your App
**WWDC21 · Session 10046** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10046/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session covers iOS 15 improvements across three areas: the system PHPicker (ordered selection, preselection, loading progress), new PHCloudIdentifier APIs for mapping the same asset across different devices signed in to iCloud Photos, and Limited Photos Library updates (custom album creation and a completion-handler-based picker presentation). Assets Library is announced as deprecated since iOS 9 with removal planned, and new PhotoKit error codes are added for change request, resource request, and access failures.

## Key Topics

**PHPicker Improvements (iOS 15)**
- *Ordered selection*: Set `PHPickerConfiguration.selection = .ordered` to display selection order numbers (1, 2, 3…) instead of plain checkmarks. Requires `selectionLimit > 1`.
- *Preselection*: New `PHPickerConfiguration.preselectedAssetIdentifiers: [String]` allows the picker to open with photos already selected. Returned `PHPickerResult` objects for preselected photos have **empty item providers** — the app must use its cached results from the previous picker session to get the actual asset data for those identifiers. Results for items deselected by the user are omitted. On cancellation, only preselected items are returned (all providers empty).
- *Progress reporting*: `NSItemProvider.loadObject(ofClass:completionHandler:)` now returns a `Progress` object that reflects actual download progress when iCloud Photos + Optimize Storage is active. Use it to show a real progress bar rather than an indeterminate spinner.

**PHCloudIdentifier — Cross-Device Asset Mapping (iOS 15 / macOS 12)**
Every photo library has device-specific `localIdentifier`s that differ between devices, even when synced via iCloud Photos. `PHCloudIdentifier` provides a stable identifier that maps to the same underlying asset across devices. The workflow:
1. On the source device, call `PHPhotoLibrary.cloudIdentifierMappings(forLocalIdentifiers:)` to get a `[String: PHCloudIdentifierMapping]` dictionary.
2. Extract `PHCloudIdentifier` values, archive them to strings, and sync to other devices via CloudKit or another service.
3. On the destination device, call `PHPhotoLibrary.localIdentifierMappings(for:)` to get `[PHCloudIdentifier: PHLocalIdentifierMapping]`, then extract the `localIdentifier` strings to fetch assets normally.

Two error codes on mapping objects: `.identifierNotFound` (asset absent or inaccessible) and `.multipleIdentifiersFound` (library can't uniquely match — retrieve candidates from `userInfo[PHLocalIdentifiersErrorKey]`). Because mapping is expensive, perform it only at load/save boundaries and store cloud identifiers for cross-device sharing.

**Limited Library Updates (iOS 15)**
- Apps can now create, fetch, and update their own photo albums while in limited library access mode (previously blocked).
- New `PHPhotoLibrary.presentLimitedLibraryPicker(from:completion:)` overload accepts a completion handler that receives an array of newly selected asset identifiers, so the app knows exactly which photos the user added to their limited selection.

## APIs & Frameworks

- **PhotoKit** (`Photos` framework)
- `PHPickerConfiguration` — picker configuration
  - `var selection: PHPickerConfiguration.Selection` **[NEW]** — `.default` or `.ordered`
  - `var preselectedAssetIdentifiers: [String]` **[NEW]** — pre-select assets by identifier
- `PHPickerResult` — `var itemProvider: NSItemProvider` (empty for preselected-but-unchanged assets)
- `NSItemProvider` — `func loadObject(ofClass:completionHandler:) -> Progress` — progress for iCloud downloads
- `PHCloudIdentifier` **[NEW]** — stable cross-device asset/album identifier
  - `init(stringValue:)` / `var stringValue: String` — serialization
- `PHCloudIdentifierMapping` **[NEW]** — mapping result wrapping `cloudIdentifier` or `error`
- `PHLocalIdentifierMapping` **[NEW]** — mapping result wrapping `localIdentifier` or `error`
- `PHPhotoLibrary` — shared instance methods
  - `func cloudIdentifierMappings(forLocalIdentifiers:) -> [String: PHCloudIdentifierMapping]` **[NEW]**
  - `func localIdentifierMappings(for:) -> [PHCloudIdentifier: PHLocalIdentifierMapping]` **[NEW]**
  - `func presentLimitedLibraryPicker(from:completion: ([String]) -> Void)` **[NEW]**
- `PHLocalIdentifiersErrorKey: String` **[NEW]** — key in Multiple Identifiers Found error `userInfo`
- `PHAssetCreationRequest`, `PHAssetChangeRequest` — new error codes for change/resource/access failures **[NEW]**
- `PHAuthorizationStatus.limited` — limited library mode (iOS 14+)
- **AssetsLibrary** (`ALAssetsLibrary`) — **deprecated** since iOS 9, removal planned; migrate to PHPicker or PhotoKit

## Code Highlights

Ordered selection:
```swift
var config = PHPickerConfiguration(photoLibrary: .shared())
config.selectionLimit = 5
config.selection = .ordered           // NEW
let picker = PHPickerViewController(configuration: config)
```

Preselection setup:
```swift
config.preselectedAssetIdentifiers = previousSelection.map { $0.assetIdentifier }
```

Handling preselected results (empty item providers):
```swift
func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
    var newSelection: [String: PHPickerResult] = [:]
    for result in results {
        let id = result.assetIdentifier!
        // Re-use cached result if provider is empty (preselected, unchanged)
        newSelection[id] = existingSelection[id] ?? result
    }
    currentSelection = newSelection
}
```

Cloud identifier mapping (source device → destination device):
```swift
// Source device
let mappings = PHPhotoLibrary.shared().cloudIdentifierMappings(forLocalIdentifiers: localIDs)
let cloudIDs = mappings.compactMapValues { $0.cloudIdentifier }

// Destination device
let localMappings = PHPhotoLibrary.shared().localIdentifierMappings(for: Array(cloudIDs.values))
for (cloudID, mapping) in localMappings {
    if let localID = mapping.localIdentifier { /* use localID */ }
    else if mapping.error?.code == .identifierNotFound { /* show placeholder */ }
}
```

## Takeaways

- Adopt the system PHPicker to provide users privacy transparency; ordered selection and preselection make it suitable for richer editing workflows without requiring full PhotoKit access.
- Use `PHCloudIdentifier` to build cross-device photo project experiences — sync cloud identifiers via CloudKit, and convert to local identifiers on each device at load/save time.
- Limited library apps can now create and manage their own albums, and the new completion-handler picker API tells the app exactly which new photos the user chose.
- Migrate away from deprecated `ALAssetsLibrary` (deprecated since iOS 9) before Apple removes it in a future SDK.

---
_Source: WWDC21 Session 10046 page (abstract, transcript, and resource links)._
