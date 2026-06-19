# What's New in File Management and Quick Look
**WWDC19 · Session 719** · [Watch](https://developer.apple.com/videos/play/wwdc2019/719/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
iOS 13 and iPadOS 13 bring long-requested file management capabilities to the platform: apps can now access entire directory trees (not just individual files) through `UIDocumentPickerViewController`, and the Files app—along with any app using the document picker or browser—now supports external USB drives and SMB network servers natively.

On the Quick Look side, this session covers three major advances: a new cross-platform `QLThumbnailGenerator` framework for fetching rich file thumbnails, editing support (Markup, video trim/rotate) directly inside `QLPreviewController`, and the arrival of Quick Look thumbnail and preview extensions on macOS Catalina—replacing the old CFPlugin-based generator system.

## Key Topics

### Directory Access
- `UIDocumentPickerViewController` can now let users select an entire folder, granting the app recursive access to all its contents.
- A new `directoryURL` property lets developers set the default location shown when the picker opens.
- Apps must still use security-scoped resource access (`startAccessingSecurityScopedResource` / `stopAccessingSecurityScopedResource`) and `NSFileCoordinator` for reads and writes.
- Persistent access across launches is maintained by storing `bookmarkData` and resolving it with `URL(resolvingBookmarkData:)`.
- User-granted folder access is listed and revocable in Settings > Privacy > Files and Folders.

### External Storage: USB and SMB
- iPadOS 13 supports APFS, HFS+, FAT, and ExFAT formatted drives via USB-C and SD card reader adapters.
- SMB servers can be connected via "Connect to Server" in Files or the sidebar.
- Any app using `UIDocumentBrowserViewController` or `UIDocumentPickerViewController` gets USB/SMB support automatically when built with the iOS 13 SDK.
- Key considerations: file operations can be slow (seconds to minutes); volumes can disappear mid-operation; do not assume the file system is always APFS/HFS+; use `FileManager.moveItem(at:to:)` for cross-volume moves.
- Use `FileManager.url(for:in:appropriateFor:create:)` with `.itemReplacementDirectory` to get the correct temporary directory relative to the destination volume.
- File system is reported as LIFS (a virtual abstraction); check capabilities rather than file system identity.

### UIDocumentBrowserViewController Customization (New)
- `shouldShowFileExtensions` **[NEW]** — always show file extensions
- `defaultDocumentAspectRatio` **[NEW]** — customize create-button icon ratio
- `localizedCreateDocumentActionTitle` **[NEW]** — customize create-button label text

### Quick Look Thumbnails (QLThumbnailGenerator)
- New cross-platform framework replacing the macOS C API (`QLThumbnail`) and `NSURLThumbnailDictionaryKey`.
- Non-UI framework: returns `CGImage` by default; opt into `UIImage`/`NSImage` by linking UIKit/AppKit.
- Asynchronous with cancellation support.
- Representation types: `.icon` (generic, fast), `.lowQualityThumbnail` (cached/embedded), `.thumbnail` (full quality, slowest). Specify `.all` to get any available.

### Quick Look Editing in QLPreviewController
- `QLPreviewController` now supports Markup (images, PDFs) and video trim/rotate in iOS 13.
- Enable per-item via the optional delegate method `editingMode(for:)` returning `.updateContents` or `.createCopy`.
- `.updateContents`: controller overwrites original; implement `didUpdateContents(of:)` to react.
- `.createCopy`: controller calls `savedEditedCopy(of:at:)` with the modified file URL.

### Quick Look Extensions on macOS (New)
- Thumbnail extensions: replace the old CFPlugin generator; implement `QLThumbnailProvider` subclass; use `QLFileThumbnailRequest` and return a `QLThumbnailReply` (CG context, AppKit context, or file URL).
- Preview extensions: new for file previews in macOS Catalina; implement `QLPreviewProvider`; declare supported UTIs in `QLSupportedContentTypes`; implement `preparePreviewOfFile(at:completionHandler:)` in the provided view controller.
- Debug with `qlmanage` on macOS.
- Old CFPlugin generators are deprecated; migrate to extensions.

### iPad Apps on Mac
- `UIDocumentPickerViewController` maps `.import`/`.open` to `NSOpenPanel` and `.exportToService`/`.moveToService` to `NSSavePanel`.
- `UIDocumentBrowserViewController` presents an `NSOpenPanel` in a separate window.
- `QLPreviewController` launches a `QLPreviewPanel` (separate window); embedding the preview controller's view is not fully supported.

## APIs & Frameworks

### UIKit — Document Picker / Browser
- `UIDocumentPickerViewController` — file/folder picker
- `UIDocumentPickerViewController.init(documentTypes:in:)` — init with `kUTTypeFolder` for folder selection **[NEW behavior]**
- `UIDocumentPickerViewController.directoryURL` **[NEW]** — set default directory shown
- `UIDocumentBrowserViewController` — full document browser
- `UIDocumentBrowserViewController.shouldShowFileExtensions` **[NEW]**
- `UIDocumentBrowserViewController.defaultDocumentAspectRatio` **[NEW]**
- `UIDocumentBrowserViewController.localizedCreateDocumentActionTitle` **[NEW]**

### Foundation — File System
- `FileManager.moveItem(at:to:)` — cross-volume safe move
- `FileManager.url(for:in:appropriateFor:create:)` with `.itemReplacementDirectory` — correct temp dir for destination volume
- `URL.startAccessingSecurityScopedResource()` / `URL.stopAccessingSecurityScopedResource()`
- `NSFileCoordinator` — coordinated reads/writes
- `URL.bookmarkData(options:)` — persist access
- `URL(resolvingBookmarkData:bookmarkDataIsStale:)` — restore persisted URL

### QuickLookThumbnailing (New Framework)
- `QLThumbnailGenerator` **[NEW]** — shared generator instance
- `QLThumbnailGenerator.Request` **[NEW]** — specify URL, size, scale, representation types
- `QLThumbnailRepresentation` **[NEW]** — result with `.type`, `.cgImage`, `.uiImage`, `.nsImage`
- `QLThumbnailRepresentation.RepresentationType` **[NEW]** — `.icon`, `.lowQualityThumbnail`, `.thumbnail`, `.all`
- `QLThumbnailGenerator.generateBestRepresentation(for:completion:)` **[NEW]**
- `QLThumbnailGenerator.generateRepresentations(for:updateHandler:)` **[NEW]** — incremental updates
- `QLThumbnailProvider` **[NEW on macOS]** — subclass for thumbnail extension
- `QLFileThumbnailRequest` — thumbnail extension request (URL, max/min size, scale)
- `QLThumbnailReply` — reply object (CG context, AppKit context, or file URL)

### Quick Look — Preview
- `QLPreviewController` — preview controller (iOS)
- `QLPreviewControllerDataSource` — provide preview items
- `QLPreviewControllerDelegate` — react to preview events
- `QLPreviewControllerDelegate.editingMode(for:)` **[NEW]** — enable editing per item
- `QLPreviewControllerDelegate.previewController(_:didUpdateContentsOf:)` **[NEW]**
- `QLPreviewControllerDelegate.previewController(_:savedEditedCopyOf:at:)` **[NEW]**
- `QLPreviewEditingMode.disabled` / `.updateContents` / `.createCopy` **[NEW]**
- `QLPreviewProvider` **[NEW on macOS]** — preview extension base class
- `QLPreviewPanel` — macOS Quick Look panel (used automatically in Catalyst)

## Code Highlights

Folder selection via document picker:
```swift
let picker = UIDocumentPickerViewController(documentTypes: [kUTTypeFolder as String],
                                            in: .open)
picker.directoryURL = FileManager.default.homeDirectoryForCurrentUser
present(picker, animated: true)
```

Enumerating folder contents with security-scoped access:
```swift
url.startAccessingSecurityScopedResource()
let coordinator = NSFileCoordinator()
coordinator.coordinate(readingItemAt: url, options: [], error: nil) { newURL in
    let contents = try? FileManager.default.contentsOfDirectory(
        at: newURL, includingPropertiesForKeys: nil)
    // process contents
}
url.stopAccessingSecurityScopedResource()
```

Fetching a thumbnail with incremental updates:
```swift
let request = QLThumbnailGenerator.Request(fileAt: fileURL,
                                           size: CGSize(width: 120, height: 120),
                                           scale: UIScreen.main.scale,
                                           representationTypes: .all)
QLThumbnailGenerator.shared.generateRepresentations(for: request) { rep, type, error in
    DispatchQueue.main.async {
        imageView.image = rep?.uiImage
    }
}
```

## Takeaways
- Apps can now grant recursive directory access on iOS 13/iPadOS 13; always use security-scoped resource calls and `NSFileCoordinator` for correctness.
- USB and SMB support is automatic for apps built against the iOS 13 SDK, but developers must handle slow/failing I/O gracefully and avoid assumptions about volume availability or file system type.
- `QLThumbnailGenerator` provides a modern, cross-platform API for rich file thumbnails with incremental representation delivery.
- Quick Look editing (Markup + video trim) is now trivially enabled in `QLPreviewController` via one delegate method.

---
_Source: WWDC19 Session 719 page (abstract, chapter summaries, code samples, and resource links)._
