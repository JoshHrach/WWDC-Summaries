# Build Better Document-Based Apps
**WWDC23 · Session 10056** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10056/)

_Platforms:_ iOS 17, iPadOS 17

## Overview
This session introduces `UIDocumentViewController` — a new UIKit base class for document content view controllers in iPadOS 17. It works together with `UIDocument` to automatically configure the navigation bar with sharing, drag-and-drop, undo/redo buttons, a document title menu, renaming support, and more. The session also covers the complete `UIDocument` subclassing pattern for loading and saving, and walks through migrating an existing desktop-class document editor to the new APIs by removing a significant amount of manual navigation bar and renaming boilerplate.

For SwiftUI apps, `DocumentGroup` automatically gains all these features without code changes. This session focuses on the UIKit path.

## Key Topics

### UIDocument: Loading and Saving
- `UIDocument` is an abstract base class for all document types; subclass it for each file format your app supports.
- All documents are backed by a `URL` — typically a file URL, but custom URL schemes backed by databases are also supported.
- Load/save operations are asynchronous; `UIDocument` is thread-safe and handles file coordination internally.
- **Simple file-based documents**: override `load(fromContents:ofType:)` (called when opened) and `contents(forType:)` (called when saving). Content type is `Data` for flat files, `FileWrapper` for packages.
- **Full control**: override `save(to:for:completionHandler:)` and `read(from:)` for custom file coordination and reading/writing (e.g., database-backed documents).
- `updateChangeCount(.done)` — call whenever a model property changes to mark the document as needing saving; `UIDocument` autosaves at appropriate times.

### UIDocumentViewController **[NEW]**
- New abstract base class for content view controllers managing a `UIDocument`.
- Automatically configures the navigation item with: document title, title menu, `UIDocumentProperties`, rename delegate, sharing button, drag-and-drop, undo/redo key commands.
- **`documentDidOpen()`** — override to populate views when the document is opened or assigned. Call a shared `configureViewForCurrentDocument()` method from both `documentDidOpen` and `viewDidLoad` (no timing guarantee between the two).
- **`navigationItemDidUpdate()`** — override to add app-specific navigation bar customizations; called after every system change to the navigation item.
- **`undoRedoItemGroup`** — add to navigation bar to show undo/redo buttons; `UIDocumentViewController` enables/disables based on undo manager availability.
- **Empty state**: when no document is associated, the view controller automatically shows an empty state UI.
- **Root view controller use**: when used as the app's root VC (no browser above it), adds a document picker button to the navigation bar; requires `UIDocumentClass` key in `Info.plist`.
- **`openDocument(completionHandler:)`** — programmatically open the document and receive a callback when ready (e.g., to present the controller).

### Automatic Renaming **[NEW]**
- `UIDocument` now conforms to `UINavigationItemRenameDelegate` in iPadOS 17.
- When using `UIDocumentViewController`, renaming from the title menu is configured automatically — no manual delegate setup needed.
- Without `UIDocumentViewController`: set `navigationItem.renameDelegate = document` to enable renaming with the system handling the file rename.

### Migration Pattern
Three steps to migrate an existing document editor to `UIDocumentViewController`:
1. Change base class from `UIViewController` to `UIDocumentViewController`.
2. Move view population code to `documentDidOpen()` and navigation bar setup to `navigationItemDidUpdate()`.
3. Delete code that `UIDocumentViewController` now handles automatically: rename delegate conformance and setup, `UIDocumentProperties` configuration, `navigationItem.style` and `backAction` assignments.

The session demonstrates that migration typically results in a net reduction of code, with the remaining code being app-specific logic only.

## APIs & Frameworks

### UIKit — UIDocument
- `UIDocument` — abstract base class for document types (existing)
- `UIDocument.load(fromContents:ofType:) throws` — load from flat file or package (existing)
- `UIDocument.contents(forType:) throws -> Any` — provide `Data` or `FileWrapper` for saving (existing)
- `UIDocument.save(to:for:completionHandler:)` — full-control async save (existing)
- `UIDocument.read(from:) throws` — full-control sync read (existing)
- `UIDocument.updateChangeCount(_:)` — mark document dirty; triggers autosave (existing)
- `UIDocument.SaveOperation` — `.forCreating`, `.forOverwriting` (existing)
- `UIDocument.documentState` — `UIDocumentState` flags including `.closed`, `.inConflict`, `.savingError` (existing)
- `UIDocument.performAsynchronousFileAccess(_:)` — coordinate file access on background queue (existing)

### UIKit — UIDocumentViewController **[NEW]**
- `UIDocumentViewController` — new base class for document content view controllers **[NEW]**
- `UIDocumentViewController.document` — associated `UIDocument` instance; settable **[NEW]**
- `UIDocumentViewController.documentDidOpen()` — called when document is opened or assigned **[NEW]**
- `UIDocumentViewController.navigationItemDidUpdate()` — called after system navigation item changes **[NEW]**
- `UIDocumentViewController.undoRedoItemGroup` — `UIBarButtonItemGroup` for undo/redo buttons **[NEW]**
- `UIDocumentViewController.openDocument(completionHandler:)` — programmatically open document **[NEW]**

### UIKit — UINavigationItemRenameDelegate **[NEW on UIDocument]**
- `UIDocument: UINavigationItemRenameDelegate` — automatic conformance in iPadOS 17 **[NEW]**
- `UINavigationItem.renameDelegate` — set document as rename delegate for title menu renaming **[extended]**

### UIKit — Supporting APIs
- `UIDocumentProperties` — document metadata for title menu (existing; now auto-populated by `UIDocumentViewController`)
- `UINavigationItem.titleMenuProvider` — provides title menu (existing)
- `UIBarButtonItemGroup` — group for navigation bar items (existing)
- `UIDocumentPickerViewController` — presented by root `UIDocumentViewController` when no browser is present (existing)

### Info.plist Keys
- `UIDocumentClass` — maps a file type to the `UIDocument` subclass; required for root `UIDocumentViewController` to show document picker button

### SwiftUI (for completeness)
- `DocumentGroup` — SwiftUI document-based app scene; automatically gains all `UIDocumentViewController` features on iPadOS 17 with no code changes (existing)

## Code Highlights

UIDocument subclass — loading and saving:
```swift
override func load(fromContents contents: Any, ofType typeName: String?) throws {
    guard let data = contents as? Data,
          let text = String(data: data, encoding: .utf8) else {
        throw DocumentError.readError
    }
    self.text = text
}

override func contents(forType typeName: String) throws -> Any {
    guard let data = self.text?.data(using: .utf8) else {
        throw DocumentError.writeError
    }
    return data
}
```

Marking document dirty on property change:
```swift
var text: String? {
    didSet {
        if oldValue != nil && oldValue != text {
            self.updateChangeCount(.done)
        }
    }
}
```

UIDocumentViewController — robust view configuration:
```swift
override func documentDidOpen() { configureViewForCurrentDocument() }
override func viewDidLoad() { super.viewDidLoad(); configureViewForCurrentDocument() }

func configureViewForCurrentDocument() {
    guard let document = markdownDocument,
          !document.documentState.contains(.closed),
          isViewLoaded else { return }
    // Populate views from document
}
```

Navigation bar customization:
```swift
override func navigationItemDidUpdate() {
    navigationItem.trailingItemGroups = [undoRedoItemGroup, ...]
}
```

## Takeaways
- `UIDocumentViewController` eliminates the need to manually wire up document title menu, rename delegate, `UIDocumentProperties`, undo/redo buttons, and sharing — subclass it and delete the boilerplate.
- Always implement `configureViewForCurrentDocument()` and call it from both `documentDidOpen()` and `viewDidLoad()` — there is no ordering guarantee between the two.
- `UIDocument` now conforms to `UINavigationItemRenameDelegate`; you only need `navigationItem.renameDelegate = document` to enable renaming without `UIDocumentViewController`.
- For SwiftUI apps, `DocumentGroup` picks up all of these improvements automatically in iPadOS 17.

---
_Source: WWDC23 Session 10056 page (abstract, chapter summaries, code samples, and resource links)._
