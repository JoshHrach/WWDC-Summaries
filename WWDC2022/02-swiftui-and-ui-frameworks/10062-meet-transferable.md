# Meet Transferable
**WWDC22 · Session 10062** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10062/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9

## Overview
`Transferable` is a new Swift-first, declarative protocol introduced in the `CoreTransferable` framework that makes it easy to add sharing, drag and drop, copy/paste, and other data-transfer features to your app's model types. It replaces the need for manual `NSItemProvider` and `NSPasteboardItem` work with a composable, type-safe API integrated directly into SwiftUI.

Types conform to `Transferable` by implementing a single `transferRepresentation` property using a DSL of representations — `CodableRepresentation`, `DataRepresentation`, `FileRepresentation`, and `ProxyRepresentation` — that describe how the model converts to and from binary data and which UTType content type identifies the format. Multiple representations can be listed in priority order, so a model can be shared natively with apps that understand the custom type, but also fall back to text or other common formats for other receivers.

The session also introduces `ShareLink`, a new SwiftUI view that enables sharing from watchOS for the first time, plus a redesigned share sheet on iOS and macOS.

## Key Topics

### The Transferable Protocol
- Single requirement: `static var transferRepresentation: some TransferRepresentation`
- Representations are composable and priority-ordered; receiver picks the first content type it supports
- Built-in conformances: `String`, `Data`, `URL`, `AttributedString`, `Image`

### CodableRepresentation
- For types that already conform to `Codable`
- Uses JSON encoder/decoder by default; custom encoder/decoder pair supported
- Requires specifying a `UTType` content type (custom or standard)

### DataRepresentation
- For types stored in memory that have custom binary serialization
- Provide exporting closure (`→ Data`) and importing closure (`Data →`)
- Best when data fits comfortably in memory

### FileRepresentation
- For large on-disk assets (videos, large files) — avoids loading full file into memory
- Passes a file URL via `SentTransferredFile`; grants temporary sandbox extension to receiver
- Receiver copies the file to a permanent location during import

### ProxyRepresentation
- Delegates to another `Transferable` type (e.g., represent a model as its `name` string or `url`)
- Can describe export only, import only, or both
- Use `ProxyRepresentation` (not `FileRepresentation`) for plain URL values that point to web addresses

### Advanced Features
- `.exportingCondition { condition }` — conditionally suppress an export representation at runtime
- Custom `TransferRepresentation` types — compose representations into reusable building blocks via a `body` property, like custom SwiftUI views
- Representation ordering: receiver always picks the first content type it supports

### SwiftUI Integration
- `ShareLink(item:preview:)` — **[NEW]** share sheet trigger; now available on watchOS
- `.draggable(_:)` — drag source modifier
- `.dropDestination(payloadType:action:)` — drop target modifier
- `PasteButton(payloadType:) { items in }` — paste button with typed payload

### Uniform Type Identifiers
- Declare custom UTType in `Info.plist` and in code via `UTType(exportedAs:)`
- File extension registration lets the system associate files on disk with your app

## APIs & Frameworks

**CoreTransferable** **[NEW]**
- `Transferable` protocol — **[NEW]**
  - `static var transferRepresentation: some TransferRepresentation`
- `TransferRepresentation` protocol — **[NEW]**
- `CodableRepresentation<Item, ContentType>` — **[NEW]**
  - `init(contentType:)` (JSON default)
  - `init(for:contentType:encoder:decoder:)`
- `DataRepresentation<Item>` — **[NEW]**
  - `init(contentType:exporting:importing:)`
  - `init(exportedContentType:exporting:)`
  - `init(importedContentType:importing:)`
- `FileRepresentation<Item>` — **[NEW]**
  - `init(contentType:exporting:importing:)`
  - `SentTransferredFile` — **[NEW]** wraps a URL for file transfer
  - `ReceivedTransferredFile` — **[NEW]** wraps received URL with sandbox access
- `ProxyRepresentation<Item, ProxyType>` — **[NEW]**
  - `init(exporting:)` (key path or closure)
  - `init(importing:)`
  - `init(exporting:importing:)`
- `.exportingCondition(_ condition: (Item) -> Bool)` — **[NEW]** modifier on any representation

**SwiftUI**
- `ShareLink` — **[NEW]** share sheet trigger view; watchOS support new
  - `ShareLink(item:preview:)`
  - `ShareLink(item:subject:message:preview:)`
  - `SharePreview` — **[NEW]** title + image metadata for share sheet
- `.draggable(_:)` — drag source modifier
- `.dropDestination(payloadType:action:)` — drop target modifier
- `PasteButton(payloadType:) { items in }` — paste button

**UniformTypeIdentifiers**
- `UTType` — uniform type identifier
- `UTType(exportedAs:)` — declare a custom type exported by the app
- Standard types: `.commaSeparatedText`, `.mpeg4Movie`, `.jpeg`, `.png`, etc.

## Code Highlights

`CodableRepresentation` conformance with a custom UTType:
```swift
extension Profile: Transferable {
    static var transferRepresentation: some TransferRepresentation {
        CodableRepresentation(contentType: .profile)
        ProxyRepresentation(exporting: \.name) // fallback to plain text
    }
}

extension UTType {
    static var profile: UTType = UTType(exportedAs: "com.example.profile")
}
```

`FileRepresentation` for memory-efficient video transfer:
```swift
extension Video: Transferable {
    static var transferRepresentation: some TransferRepresentation {
        FileRepresentation(contentType: .mpeg4Movie) { SentTransferredFile($0.file) }
            importing: { received in
                let copy = try Self.copyVideoFile(source: received.file)
                return Self.init(file: copy)
            }
        ProxyRepresentation(exporting: \.file) // plain URL fallback
    }
}
```

Conditional export:
```swift
DataRepresentation(contentType: .commaSeparatedText) { ... } importing: { ... }
    .exportingCondition { $0.supportsCSV }
```

## Takeaways
- `Transferable` is the new unified Swift-first API for all data-transfer scenarios (sharing, drag and drop, copy/paste); it replaces direct `NSItemProvider` / `NSPasteboardItem` usage in most cases.
- Use `CodableRepresentation` for `Codable` types, `DataRepresentation` for in-memory serialization, `FileRepresentation` for on-disk large assets, and `ProxyRepresentation` to delegate to another transferable type.
- Multiple representations can be combined in priority order so the receiver picks the best format it understands.
- `ShareLink` in SwiftUI enables sharing from watchOS for the first time and provides a redesigned share sheet experience on iOS and macOS.

---
_Source: WWDC22 Session 10062 page (abstract, chapter summaries, code samples, and resource links)._
