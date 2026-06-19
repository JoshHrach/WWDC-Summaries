# Embed the Photos Picker in Your App
**WWDC23 · Session 10107** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10107/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
This session covers new improvements to the system Photos picker introduced in iOS 17, focusing on the ability to embed the picker inline within an app rather than only presenting it as a modal sheet. Because the picker remains out-of-process even when embedded, privacy is maintained without requiring any photo library access permissions.

The session details new SwiftUI, UIKit, and AppKit APIs for controlling picker style, accessory visibility, disabled capabilities, and continuous selection. It also covers a new Options menu that lets users strip sensitive metadata from selected photos, and best practices for receiving HDR images, HDR videos, and Cinematic mode videos in their original formats.

A live demo shows progressively transforming a photo annotation app from a modal picker button to a seamlessly embedded picker using a handful of modifier calls.

## Key Topics

- **Embedded picker (inline style)** — Using `.photosPickerStyle(.inline)` to embed the out-of-process picker directly in the view hierarchy, with an automatic first-launch onboarding banner explaining privacy protections.
- **Accessory visibility** — Hiding the navigation bar and toolbar with `.photosPickerAccessoryVisibility(_:edges:)`.
- **Disabled capabilities** — Hiding search bar, albums tab, sidebar, staging area, Cancel/Add buttons with `.photosPickerDisabledCapabilities(_:)`.
- **Continuous selection** — Setting `selectionBehavior` to `.continuous` for real-time live selection updates without requiring an Add button tap.
- **Compact style** — `.photosPickerStyle(.compact)` for single-row embedded pickers in space-constrained layouts.
- **Options menu** — New system UI allowing users to strip location and other sensitive metadata from selected assets; requires no adoption for Transferable or PHPickerViewController users.
- **HDR and Cinematic mode** — Using `.current` encoding policy and generic content types (`.image`, `.movie`) to receive assets in original formats without transcoding.

## APIs & Frameworks

**SwiftUI**
- `PhotosPicker` view **[NEW enhancements]**
- `.photosPickerStyle(_:)` modifier **[NEW]** — values: `.presentation` (default), `.inline`, `.compact`
- `.photosPickerAccessoryVisibility(_:edges:)` modifier **[NEW]** — hides nav bar, toolbar, sidebar per edge
- `.photosPickerDisabledCapabilities(_:)` modifier **[NEW]** — disables: `.search`, `.collectionNavigation`, `.stagingArea`, `.selectionActions`
- `PhotosPickerSelectionBehavior.continuous` **[NEW]** — live updates on selection change
- `.ignoresSafeArea()` — used to allow embedded picker to extend to screen edges

**UIKit / AppKit**
- `PHPickerConfiguration` — existing; new `selection` property set to `.continuous` **[NEW]**
- `PHPickerConfiguration.mode` — new `.compact` value **[NEW]**
- `PHPickerConfiguration.edgesWithoutContentMargins` **[NEW]** — hides accessory edges
- `PHPickerConfiguration.disabledCapabilities` **[NEW]** — set of capabilities to disable
- `PHPickerViewController` — embed as child view controller; set frame or Auto Layout constraints
- `PHPickerConfiguration.Update` **[NEW]** — update picker configuration while picker is displayed
- `PHPickerViewController.deselectAsset(_:)` — existing
- `PHPickerViewController.moveAsset(_:afterAsset:)` — existing

**PhotoKit**
- `PHPickerFilter` — existing; used for filtering asset types
- `PHImageRequestOptions` — used for full library access fallback for Cinematic mode originals

**Encoding / Content Types**
- `PHPickerConfiguration.preferredItemEncoding` — set to `.current` **[NEW recommended practice]** for no-transcode HDR
- `UTType.image`, `UTType.movie` — generic content types for original-format requests
- Avoiding specific types like `UTType.jpeg` to prevent forced transcoding

## Code Highlights

Embedding the picker with continuous selection and hidden accessories:
```swift
PhotosPicker(
    selection: $selectedItems,
    selectionBehavior: .continuous,
    matching: .images
) {
    // empty label
}
.photosPickerStyle(.inline)
.photosPickerAccessoryVisibility(.hidden)
.photosPickerDisabledCapabilities(.selectionActions)
.frame(height: 300)
.ignoresSafeArea()
```

Receiving images in original format (no transcoding) in UIKit:
```swift
var config = PHPickerConfiguration()
config.preferredItemEncoding = .current
config.filter = .images
// request generic .image UTType, not .jpeg
```

## Takeaways

- Use `.photosPickerStyle(.inline)` to embed the Photos picker without requiring any photo library permission — privacy is preserved out-of-process automatically.
- Set `selectionBehavior` to `.continuous` to drive live UI updates without an Add button.
- Use `.current` encoding policy and generic UTTypes (`.image`, `.movie`) when you need HDR or Cinematic mode videos in their original formats.
- Avoid requesting full photo library access; the Options menu and embedded picker give users more granular control over what they share.

---
_Source: WWDC23 Session 10107 page (abstract, chapter summaries, code samples, and resource links)._
