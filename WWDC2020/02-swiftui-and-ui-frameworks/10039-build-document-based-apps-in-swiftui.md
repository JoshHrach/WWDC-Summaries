# Build Document-Based Apps in SwiftUI
**WWDC20 · Session 10039** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10039/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
SwiftUI 2.0 introduces `DocumentGroup`, a new scene type that lets developers add full document-based app capabilities with minimal code. By composing `DocumentGroup` into an `App` alongside other scenes like `WindowGroup`, developers get automatic platform-appropriate behavior: a Document Browser with navigation bar search and sharing on iOS/iPadOS, and State Tracking, Handoff, and standard menu commands on macOS — all without writing the boilerplate that UIDocumentBrowserViewController or NSDocument typically require.

The session builds a cross-platform (iOS + macOS) drawing app called ShapeEdit from scratch, demonstrating how to declare a custom file type using Uniform Type Identifiers, conform a value-type data model to `FileDocument`, and wire it into a `DocumentGroup` scene. The result is a fully functional document-based app supporting open-in-place, multi-window editing, and undo/redo.

The new `FileDocument` protocol uses Swift value types rather than class-based documents, enabling copy-on-write semantics so the app can begin saving while the user continues editing.

## Key Topics

**DocumentGroup Scene**
`DocumentGroup` is declared in the `App.body` and takes a document type conforming to `FileDocument` (or `ReferenceFileDocument` for class-based models), a `newDocument` factory, and a view-builder closure that receives a `FileDocumentConfiguration`. The `file.$document` binding provides read-write access to the model and triggers undo registration and document dirtying automatically.

**FileDocument Protocol**
A value-type protocol with three requirements:
1. `static var readableContentTypes: [UTType]` — the file types the app can open.
2. `init(configuration:)` — deserialize from `FileDocumentReadConfiguration` (provides `FileWrapper`).
3. `fileWrapper(configuration:)` — serialize to `FileDocumentWriteConfiguration` (provides `FileWrapper`).
Conforming types should also conform to `Codable` for easy JSON serialization.

**Uniform Type Identifiers**
New `UniformTypeIdentifiers` framework (replacing `kUTType*` constants). Types are declared in the target's Info.plist under Exported Type Identifiers (types the app owns) or Imported Type Identifiers (types declared elsewhere). `UTType(exportedAs:)` creates an owned constant; `UTType(importedAs:)` returns a computed variable that may change as apps are installed.

**Composability**
Multiple `DocumentGroup` scenes and `WindowGroup` scenes can coexist in one app, enabling apps like Xcode that have document support alongside non-document UI.

## APIs & Frameworks

### SwiftUI **[NEW]**
- `DocumentGroup(newDocument:content:)` **[NEW]** — scene for document-based apps
- `DocumentGroup(viewing:content:)` **[NEW]** — read-only document viewer variant
- `FileDocument` protocol **[NEW]** — value-type document model
  - `static var readableContentTypes: [UTType]`
  - `static var writableContentTypes: [UTType]` (defaults to `readableContentTypes`)
  - `init(configuration: FileDocumentReadConfiguration) throws`
  - `func fileWrapper(configuration: FileDocumentWriteConfiguration) throws -> FileWrapper`
- `ReferenceFileDocument` protocol **[NEW]** — class-based document model (for reference semantics)
- `FileDocumentConfiguration<Document>` **[NEW]** — closure argument type; exposes `$document` binding
- `FileDocumentReadConfiguration` **[NEW]** — provides `contentType` and `file` (`FileWrapper`)
- `FileDocumentWriteConfiguration` **[NEW]** — provides `contentType` and `existingFile` (`FileWrapper?`)
- `TextEditor(text:)` **[NEW]** — multi-line text editing view (used in default template)

### Uniform Type Identifiers **[NEW framework]**
- `UTType` **[NEW]** — strongly typed replacement for `CFString` UTI
- `UTType(exportedAs:conformingTo:)` **[NEW]** — declare an owned type
- `UTType(importedAs:conformingTo:)` **[NEW]** — reference an externally-declared type
- `UTType.plainText`, `UTType.data`, `UTType.content` — built-in type constants
- Info.plist keys: `UTExportedTypeDeclarations`, `UTImportedTypeDeclarations`
- `CFBundleTypeName`, `UTTypeIdentifier`, `UTTypeConformsTo`, `CFBundleTypeExtensions`

### Foundation
- `FileWrapper` — wraps file or directory for read/write
- `JSONEncoder`, `JSONDecoder` — for `Codable` document serialization
- `Codable` — conformance on document model struct for JSON I/O

## Code Highlights

Minimal document-based app:
```swift
@main
struct TextEdit: App {
    var body: some Scene {
        DocumentGroup(newDocument: TextDocument()) { file in
            TextEditor(text: file.$document.text)
        }
    }
}
```

Custom FileDocument conformance:
```swift
struct ShapeEditDocument: FileDocument {
    static var readableContentTypes: [UTType] { [.shapeEditDocument] }

    var graphics: [Graphic] = []

    init(configuration: FileDocumentReadConfiguration) throws {
        let data = configuration.file.regularFileContents!
        graphics = try JSONDecoder().decode([Graphic].self, from: data)
    }

    func fileWrapper(configuration: FileDocumentWriteConfiguration) throws -> FileWrapper {
        let data = try JSONEncoder().encode(graphics)
        return FileWrapper(regularFileWithContents: data)
    }
}

extension UTType {
    static let shapeEditDocument = UTType(exportedAs: "com.example.ShapeEdit.shapeedit")
}
```

## Takeaways
- `DocumentGroup` is a new SwiftUI scene type that delivers full document-based app behavior (browser, open-in-place, undo, multi-window) across iOS and macOS with a handful of lines of code.
- `FileDocument` uses Swift value-type semantics, enabling copy-on-write safe saving while editing continues; use `ReferenceFileDocument` only if reference semantics are required.
- The new `UniformTypeIdentifiers` framework provides strongly-typed `UTType` constants; use `exportedAs` for your app's own types (constant) and `importedAs` for types from other parties (computed, may change).
- `DocumentGroup` composes with other scene types (`WindowGroup`) in the same app, making it suitable for apps like Xcode that combine document editing with non-document UI.

---
_Source: WWDC20 Session 10039 page (abstract, chapter summaries, code samples, and resource links)._
