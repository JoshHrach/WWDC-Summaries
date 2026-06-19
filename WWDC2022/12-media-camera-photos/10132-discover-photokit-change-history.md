# Discover PhotoKit Change History
**WWDC22 · Session 10132** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10132/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
This session introduces the new PhotoKit persistent change history API, which allows apps to efficiently track additions, deletions, and updates to the photo library across app launches — without needing to perform multiple expensive fetches and diffs. The API is built around a persistent change token that captures the library state at a point in time, persisted across launches. On relaunch, the app fetches all changes since the stored token in a single API call.

This replaces the brittle pattern of re-fetching all tracked assets on launch, checking modification dates (which can have false positives from internal Photos processing), and diffing asset arrays to detect deletions. The new API provides exact sets of inserted, updated, and deleted local identifiers for assets, asset collections, and collection lists.

## Key Topics

### Persistent Change History Overview
- New `PHPhotoLibrary.fetchPersistentChanges(since:)` API **[NEW]** returns a sequence of `PHPersistentChange` objects representing changes since the provided token.
- `PHPersistentChangeToken` — lightweight, persistent token representing photo library state at a point in time **[NEW]**.
- Store the token to disk after processing; load it on next launch to fetch only what changed.
- Works for changes from any app (including third-party apps) and iCloud sync.
- In limited library mode, only returns changes for user-selected PhotoKit objects.
- Available on macOS, iOS, iPadOS, and tvOS.

### Change Detail Types
Each `PHPersistentChange` can provide `PHPersistentObjectChangeDetails` for three `PHObjectType` categories:
- `.asset` — `PHAsset` insertions, updates, deletions.
- `.assetCollection` — album/moment changes.
- `.collectionList` — folder changes.

For each type, `PHPersistentObjectChangeDetails` provides:
- `insertedLocalIdentifiers: Set<String>` **[NEW]**
- `updatedLocalIdentifiers: Set<String>` **[NEW]**
- `deletedLocalIdentifiers: Set<String>` **[NEW]**

### Error Handling
Two new error codes:
- `PHPhotosError.persistentChangeTokenExpired` — token is older than the available change history window **[NEW]**.
- `PHPhotosError.persistentChangeDetailsUnavailable` — changes cannot be fully reconstructed **[NEW]**.
On either error: re-fetch all tracked objects using standard `PHAsset.fetchAssets(with:options:)`.

### Other New PhotoKit APIs
- Cinematic video support: new `PHAssetMediaSubtype` for cinematic video, new `PHAssetCollectionSubtype` smart album for cinematic videos **[NEW]**.
- `PHAsset.hasAdjustments` — check if an asset has edits applied (useful when processing updated identifiers) **[NEW]**.
- `PHPhotosError.libraryInFileProviderSyncRoot` — error when photo library bundle is in a File Provider sync root on macOS (can cause corruption) **[NEW]**.
- `PHPhotosError.networkError` — returned when an asset resource cannot be found due to network issues **[NEW]**.

### Best Practices
- Fetch persistent changes on a background thread to avoid blocking the UI.
- Only process change types relevant to your app (don't process all changes).
- Batch fetch inserted/updated assets in one call rather than multiple smaller requests.
- Photo libraries can generate many internal change records from sync and processing; expect potentially large change sets if app is rarely launched.
- Complement with `PHPhotoLibraryChangeObserver` for live changes while the app is active.

## APIs & Frameworks

### PhotoKit **[NEW]**
- `PHPhotoLibrary.fetchPersistentChanges(since: PHPersistentChangeToken) throws -> [PHPersistentChange]` **[NEW]**
- `PHPhotoLibrary.currentChangeToken: PHPersistentChangeToken` **[NEW]**
- `PHPersistentChangeToken` — codable, persistent change token **[NEW]**
- `PHPersistentChange` — represents a batch of changes **[NEW]**
- `PHPersistentChange.changeToken: PHPersistentChangeToken` **[NEW]**
- `PHPersistentChange.changeDetails(for: PHObjectType) -> PHPersistentObjectChangeDetails?` **[NEW]**
- `PHPersistentObjectChangeDetails` **[NEW]**
  - `.insertedLocalIdentifiers: Set<String>`
  - `.updatedLocalIdentifiers: Set<String>`
  - `.deletedLocalIdentifiers: Set<String>`
- `PHObjectType.asset`, `.assetCollection`, `.collectionList`
- `PHPhotosError.persistentChangeTokenExpired` **[NEW]**
- `PHPhotosError.persistentChangeDetailsUnavailable` **[NEW]**
- `PHPhotosError.libraryInFileProviderSyncRoot` **[NEW]**
- `PHPhotosError.networkError` **[NEW]**
- `PHAsset.hasAdjustments: Bool` **[NEW]**
- `PHAssetMediaSubtype` — new cinematic video subtype **[NEW]**
- `PHAssetCollectionSubtype` — new cinematic video smart album **[NEW]**

### Existing (referenced)
- `PHPhotoLibraryChangeObserver` — live change notifications while app is running
- `PHAsset.fetchAssets(withLocalIdentifiers:options:)` — batch fetch by identifiers
- `PHFetchOptions` — predicate-based fetching

## Code Highlights

```swift
// Fetch and process persistent changes
do {
    let persistentChanges = try PHPhotoLibrary.shared()
        .fetchPersistentChanges(since: self.lastStoredToken)

    for persistentChange in persistentChanges {
        if let details = persistentChange.changeDetails(for: PHObjectType.asset) {
            // Process inserted assets
            let insertedAssets = PHAsset.fetchAssets(
                withLocalIdentifiers: Array(details.insertedLocalIdentifiers), options: nil)
            
            // Check updated assets for edits
            let updatedAssets = PHAsset.fetchAssets(
                withLocalIdentifiers: Array(details.updatedLocalIdentifiers), options: nil)
            updatedAssets.enumerateObjects { asset, _, _ in
                if asset.hasAdjustments { /* redraw */ }
            }
            
            // Handle deleted asset identifiers
            for id in details.deletedLocalIdentifiers {
                // find and update collages referencing this id
            }
        }
        self.lastStoredToken = persistentChange.changeToken
    }
} catch PHPhotosError.persistentChangeTokenExpired,
        PHPhotosError.persistentChangeDetailsUnavailable {
    // Fallback: re-fetch all tracked assets
    let fetchResult = PHAsset.fetchAssets(withLocalIdentifiers: trackedIdentifiers, options: nil)
}
```

## Takeaways
- The persistent change history API replaces fragile multi-fetch-and-diff patterns with a single, precise enumeration of inserted, updated, and deleted asset identifiers since a stored token.
- Always store the change token after processing and handle both expiry and unavailable-details errors with a full re-fetch fallback.
- Process changes on a background thread; batch-fetch assets by identifier set rather than making individual requests.
- Pair persistent history (for offline/between-launch changes) with `PHPhotoLibraryChangeObserver` (for live in-session changes) for complete coverage.

---
_Source: WWDC22 Session 10132 page (abstract, chapter summaries, code samples, and resource links)._
