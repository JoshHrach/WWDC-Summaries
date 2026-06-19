# Handle the Limited Photos Library in Your App
**WWDC20 · Session 10641** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10641/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
iOS 14 introduces the Limited Photos Library — a new authorization state where users can select exactly which photos and videos an app is allowed to access via PhotoKit, rather than granting full library access. This session explains the new `.limited` authorization status, the updated access-level-aware authorization APIs, how to present the built-in limited library management UI from within your app, and how to suppress the automatic system prompt that would otherwise appear on first PhotoKit access.

The session also makes a strong case for reconsideration: many apps that request full photo library access don't actually need it. Features like profile picture upload, sending images in messages, or embedding photos in documents can all be implemented with the new `PHPickerViewController` without requiring any authorization at all.

## Key Topics

**What Limited Library Access Means**
- New iOS 14 authorization state: the user selects a specific subset of assets your app can see via PhotoKit **[NEW]**
- Acts as a filter on all PhotoKit fetches — your app only sees selected assets and resources
- User can modify the selection at any time via Settings → Privacy → Photos → (App) → Selected Photos
- App is notified of selection changes through `PHPhotoLibraryChangeObserver`
- Affects all apps using PhotoKit — even apps already shipped before iOS 14

**When the Automatic Prompt Appears**
- For apps that have not adopted the new APIs: once per app lifecycle, on first PhotoKit fetch after launch, the system shows a prompt asking users to keep or modify their selection
- This prompt can be suppressed via an Info.plist key

**Reconsider Whether You Need PhotoKit at All**
- `PHPickerViewController` (WWDC20: "Meet the New Photos Picker") requires zero authorization — use it for profile photos, sharing images, embedding photos in documents
- Apps that only save photos: use `.addOnly` access level instead of `.readWrite`
- Apps requiring full access: browsing, editing, camera replacement, backup apps

**New Authorization APIs**
- Old APIs (`PHPhotoLibrary.authorizationStatus()` and `requestAuthorization(_:)`) are deprecated — they return `.authorized` even for limited access, hiding the new state
- New APIs take a `PHAccessLevel` parameter (`.addOnly` or `.readWrite`)
- New `PHAuthorizationStatus.limited` value **[NEW]** — only returned by the new APIs with `.readWrite` access level
- Limited access does not affect `.addOnly` authorization

**Behavioral Differences Under Limited Access**
- Assets your app creates are automatically added to the user's selection
- User albums: cannot be fetched or created (app-specific albums require design changes)
- No access to cloud shared assets or albums

**Presenting Limited Library Picker**
- `PHPhotoLibrary.shared().presentLimitedLibraryPicker(from:)` **[NEW]** — presents the native asset selection UI in-app
- Tie to a visible button or affordance in your photo browsing UI
- Monitor `PHPhotoLibraryChangeObserver` to receive change notifications after the picker dismisses

**Suppressing the System Prompt**
- Info.plist key `PHPhotoLibraryPreventAutomaticLimitedAccessAlert = YES` **[NEW]** — suppresses the automatic first-launch selection prompt
- Set this key once you've added your own in-app affordance for managing the selection

## APIs & Frameworks

### PhotoKit — Authorization (Updated)
- `PHAccessLevel` **[NEW]** — enum with cases `.addOnly` and `.readWrite`
- `PHPhotoLibrary.authorizationStatus(for: PHAccessLevel) -> PHAuthorizationStatus` **[NEW]** — access-level-aware status query
- `PHPhotoLibrary.requestAuthorization(for: PHAccessLevel, handler: (PHAuthorizationStatus) -> Void)` **[NEW]** — access-level-aware authorization request
- `PHAuthorizationStatus.limited` **[NEW]** — returned when user grants limited access
- Old APIs (no access level parameter) — deprecated; always return `.authorized` for limited users

### PhotoKit — Limited Library Management (New)
- `PHPhotoLibrary.presentLimitedLibraryPicker(from viewController: UIViewController)` **[NEW]** — presents asset selection UI
- `PHPhotoLibraryChangeObserver` — existing protocol; receives change notifications when selection changes
  - `photoLibraryDidChange(_ changeInstance: PHChange)` — called after user modifies selection via picker or Settings

### Info.plist Keys
- `PHPhotoLibraryPreventAutomaticLimitedAccessAlert` **[NEW]** — Boolean, suppresses system prompt on first PhotoKit fetch when set to `YES`
- `NSPhotoLibraryUsageDescription` — existing read/write privacy string (required)
- `NSPhotoLibraryAddUsageDescription` — existing add-only privacy string (required for `.addOnly`)

## Code Highlights

Query authorization status with access level:
```swift
import Photos

let accessLevel: PHAccessLevel = .readWrite
let authorizationStatus = PHPhotoLibrary.authorizationStatus(for: accessLevel)

switch authorizationStatus {
case .limited:
    print("limited authorization granted — show selection affordance")
case .authorized:
    print("full access")
case .denied, .restricted:
    print("no access")
case .notDetermined:
    print("not determined")
@unknown default:
    break
}
```

Request read/write authorization:
```swift
let requiredAccessLevel: PHAccessLevel = .readWrite
PHPhotoLibrary.requestAuthorization(for: requiredAccessLevel) { authorizationStatus in
    switch authorizationStatus {
    case .limited:
        // User chose "Select Photos" — show in-app selection affordance
        break
    case .authorized:
        // Full access granted
        break
    default:
        break
    }
}
```

Present limited library picker with change observation:
```swift
import PhotosUI

class PhotoBrowserViewController: UIViewController, PHPhotoLibraryChangeObserver {
    let library = PHPhotoLibrary.shared()

    override func viewDidLoad() {
        super.viewDidLoad()
        library.register(self)
    }

    @IBAction func manageSelectionTapped(_ sender: UIButton) {
        library.presentLimitedLibraryPicker(from: self)
    }

    func photoLibraryDidChange(_ changeInstance: PHChange) {
        DispatchQueue.main.async {
            // Reload collection view with updated limited selection
        }
    }
}
```

## Takeaways
- Every existing app using PhotoKit can now be put into limited access mode by users in iOS 14 — update your authorization checks to use the new access-level-aware APIs and handle the `.limited` status.
- If your app only needs to pick or save a photo (not browse the library), switch to `PHPickerViewController` (no auth needed) or `.addOnly` access level — this avoids the entire limited library complexity.
- Add an explicit in-app affordance (button, menu item) calling `presentLimitedLibraryPicker(from:)` so users can manage their selection without going to Settings, and set `PHPhotoLibraryPreventAutomaticLimitedAccessAlert = YES` in Info.plist to suppress the automatic system prompt.
- Monitor `PHPhotoLibraryChangeObserver` for selection changes regardless of how they occur (in-app picker, Settings, iCloud sync, or asset deletion).

---
_Source: WWDC20 Session 10641 page (abstract, transcript, code samples, and resource links)._
