# Advances in Foundation
**WWDC19 · Session 723** · [Watch](https://developer.apple.com/videos/play/wwdc2019/723/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
iOS 13 and Swift 5.1 bring a focused set of high-value additions to the Foundation framework across six areas: ordered collection diffing, data contiguity and buffer protocols, a new compression API, new units and formatters for internationalization, Operation Queue improvements (barriers and progress reporting), and filesystem guidance for the new USB and SMB volume support in iOS 13.

The session also documents Swift ergonomics improvements to two established Foundation types — `Scanner` and `FileHandle` — that replace Objective-C-heritage output-parameter and exception-based APIs with idiomatic Swift equivalents.

## Key Topics

### Ordered Collection Diffing **[NEW]**
- `CollectionDifference<ChangeType>` — represents the insertions and removals needed to transform one ordered collection into another.
- Each change carries the element value and its offset in the source or destination collection.
- `Collection.difference(from:)` — computes the diff between two ordered collections of the same type.
- `Collection.applying(_:)` — applies a `CollectionDifference` to produce a new collection.
- Works on any `BidirectionalCollection` with `Equatable` elements, not just `String`.
- Enables efficient animated updates to table views and collection views without manual index tracking.

### Data Contiguity & Buffer Protocols **[NEW]**
- From Swift 5, `struct Data` is guaranteed to be a **contiguous buffer** (discontiguous data previously backed by `DispatchData` will be flattened during the lifecycle).
- `ContiguousBytes` protocol **[NEW]** — marks a type as offering direct access to its raw bytes without copying; `Data`, `Array<UInt8>`, `UnsafeRawBufferPointer`, and `UnsafeMutableRawBufferPointer` all conform.
- `DataProtocol` **[NEW]** — a `Collection of UInt8` generalization of `Data`'s byte-access interface, supporting both contiguous and discontiguous buffers.
- `MutableDataProtocol` **[NEW]** — adds mutation capabilities on top of `DataProtocol`.
- Conforming types: `Data`, `Array<UInt8>`, `ArraySlice<UInt8>`, `DispatchData`, `UnsafeRawBufferPointer`, `UnsafeMutableRawBufferPointer`.
- Use `DataProtocol` as a generic constraint when writing methods that accept binary data to avoid unnecessarily restricting callers to `Data` or `[UInt8]`.

### Data Compression **[NEW]**
- `Data.compressed(using:)` **[NEW]** — one line to compress data.
- `Data.decompressed(using:)` **[NEW]** — one line to decompress.
- `NSData.CompressionAlgorithm` — four algorithms: `.lzfse` (Apple default, best balance), `.lz4` (fastest), `.lzma` (best compression ratio), `.zlib` (most compatible).

### New Units **[NEW]**
- `UnitDuration` extended with subsecond units **[NEW]**: `.milliseconds`, `.microseconds`, `.nanoseconds`, `.picoseconds`.
- `UnitFrequency` gains `.framesPerSecond` **[NEW]** — functionally equivalent to hertz but semantically correct for FPS measurements.
- `UnitInformationStorage` **[NEW]** — represents digital data sizes; base units: `.bits`, `.bytes`, `.nibbles`; SI prefix units from `.kilobits`/`.kilobytes` up through `.yottabits`/`.yottabytes`; binary prefix (base-2) units `.kibibits`/`.kibibytes` up through `.yobibits`/`.yobibytes`. Use with `MeasurementFormatter` or `ByteCountFormatter`.

### New Formatters **[NEW]**
- `RelativeDateTimeFormatter` **[NEW]** — formats a date as a human-readable relative string ("1 hour ago", "tomorrow", "in 3 days") locale-correctly.
  - `dateTimeStyle: RelativeDateTimeFormatter.DateTimeStyle` — `.named` ("yesterday"), `.numeric` ("1 day ago").
  - `unitsStyle: RelativeDateTimeFormatter.UnitsStyle` — `.full`, `.short`, `.abbreviated`, `.spellOut`.
  - `localizedString(for:relativeTo:)` — main API.
- `ListFormatter` **[NEW]** — formats an array of strings into a locale-correct list with proper separators and conjunctions ("A, B, and C" in English; different in other locales).
  - `itemFormatter: Formatter?` — formatter applied to each element before joining (e.g., a `DateFormatter` to format a list of dates).
  - `localizedString(byJoining:)` — static convenience for arrays of strings.

### Operation Queue Improvements **[NEW]**
- `OperationQueue.addBarrierBlock(_:)` **[NEW]** — adds a barrier closure that executes only after all previously submitted operations finish, and blocks any new operations from starting until it completes. Analogous to `DispatchQueue`'s barrier flag.
  - Use case: safe-save / checkpoint operations that must wait for all concurrent work to drain.
  - Avoids the incorrect pattern of polling `operationCount == 0`.
- `OperationQueue.progress: Progress` **[NEW]** — tracks the overall completion of all operations added to the queue.
  - Set `progress.totalUnitCount` to enable tracking; each operation completion contributes one unit.
  - Bind to a `UIProgressView` or similar to display queue progress.

### Filesystem: USB and SMB Volume Support
- iOS 13 adds support for USB and SMB (network) external volumes.
- Use `FileManager.default.url(for:.itemReplacementDirectory, ...)` for atomic safe-save temp file locations (avoids cross-volume moves).
- Use `.mappedIfSafe` data reading option for memory-mapped files — only maps files on non-removable volumes; reads SMB/USB files directly to avoid crash on volume disappearance.
- Defer filesystem access to non-main threads (SMB reads can be slow).
- Test for volume capabilities (e.g., APFS cloning) using URL resource keys before relying on them; external volumes may not support all APFS features.

### Swift API Improvements
- **`Scanner`** (Swift 5.1) **[NEW]**: new Swift-native API replaces Objective-C output-parameter style. Methods now return values directly (e.g., `scanner.scanInt()` returns `Int?`); uses Swift `String` (grapheme cluster-based) instead of `NSString`, enabling correct handling of emoji and complex Unicode.
- **`FileHandle`** **[NEW]**: new error-throwing API replaces exception-based Objective-C API. New methods: `read(upToCount:)`, `readToEnd()`, `write(contentsOf:)` (accepts `DataProtocol`), `offset()`, `seekToEnd()`, `seek(toOffset:)`, `close()` — all throw `Error` on failure.

## APIs & Frameworks

**Swift Standard Library / Foundation**
- `CollectionDifference<ChangeType>` **[NEW]** — `insertions`, `removals`, `inverse()`
- `CollectionDifference.Change` **[NEW]** — `.insert(offset:element:associatedWith:)`, `.remove(offset:element:associatedWith:)`
- `BidirectionalCollection.difference(from:)` **[NEW]**
- `BidirectionalCollection.applying(_:)` **[NEW]**
- `ContiguousBytes` protocol **[NEW]** — `withUnsafeBytes(_:)`
- `DataProtocol` protocol **[NEW]** — collection of `UInt8` with region support
- `MutableDataProtocol` protocol **[NEW]**
- `Data.compressed(using:) throws -> Data` **[NEW]**
- `Data.decompressed(using:) throws -> Data` **[NEW]**
- `NSData.CompressionAlgorithm` — `.lzfse`, `.lz4`, `.lzma`, `.zlib`
- `UnitInformationStorage` **[NEW]** — `Measurement<UnitInformationStorage>`
- `UnitDuration.milliseconds`, `.microseconds`, `.nanoseconds`, `.picoseconds` **[NEW]**
- `UnitFrequency.framesPerSecond` **[NEW]**
- `RelativeDateTimeFormatter` **[NEW]** — `dateTimeStyle`, `unitsStyle`, `calendar`, `locale`, `localizedString(for:relativeTo:)`
- `ListFormatter` **[NEW]** — `itemFormatter`, `locale`, `localizedString(byJoining:)`, `string(from:)`
- `OperationQueue.addBarrierBlock(_:)` **[NEW]**
- `OperationQueue.progress: Progress` **[NEW]**
- `Scanner` — new Swift-native value-returning API **[NEW]**: `scanInt()`, `scanDouble()`, `scanString(_:)`, `scanUpToString(_:)`, `scanCharacters(from:)`, etc.
- `FileHandle` — new throwing API **[NEW]**: `read(upToCount:)`, `readToEnd()`, `write(contentsOf:)`, `offset()`, `seekToEnd()`, `seek(toOffset:)`, `close()`

## Code Highlights

```swift
// Ordered collection diffing
let bird = "bird"
let bear = "bear"
let diff = bird.difference(from: bear)
// diff contains: remove 'e' at offset 1, remove 'a' at offset 2,
//                insert 'i' at offset 1, insert 'd' at offset 3
let result = bear.applying(diff)  // "bird"

// Data compression
let compressed = try originalData.compressed(using: .lzfse)
let restored = try compressed.decompressed(using: .lzfse)

// RelativeDateTimeFormatter
let formatter = RelativeDateTimeFormatter()
formatter.unitsStyle = .full
let str = formatter.localizedString(for: Date().addingTimeInterval(-3600), relativeTo: Date())
// "1 hour ago" (locale-dependent)

// ListFormatter with itemFormatter
let listFormatter = ListFormatter()
let dateFormatter = DateFormatter()
dateFormatter.dateStyle = .medium
listFormatter.itemFormatter = dateFormatter
let dates = [Date(), Date().addingTimeInterval(86400)]
let str = listFormatter.string(from: dates)  // "Jun 3, 2019 and Jun 4, 2019"

// OperationQueue barrier
let queue = OperationQueue()
queue.addOperation { /* concurrent work A */ }
queue.addOperation { /* concurrent work B */ }
queue.addBarrierBlock {
    // runs only after A and B complete; no new ops start until this finishes
    saveToDisk()
}

// OperationQueue progress
queue.progress.totalUnitCount = Int64(tasks.count)
tasks.forEach { queue.addOperation($0) }
progressView.observedProgress = queue.progress

// FileHandle new API
let handle = try FileHandle(forReadingFrom: url)
let data = try handle.readToEnd()
try handle.close()
```

## Takeaways
- `CollectionDifference` makes animated collection view updates trivial — compute the diff between two arrays, then apply it directly rather than writing custom index math.
- `DataProtocol` and `ContiguousBytes` are the correct generic constraints for binary-data methods; they accept `Data`, `[UInt8]`, `DispatchData` and more without copying.
- `RelativeDateTimeFormatter` and `ListFormatter` eliminate significant boilerplate for two very common localization tasks that are easy to get wrong for non-English locales.
- `OperationQueue.addBarrierBlock` is the correct solution for checkpoint/save patterns — never poll `operationCount`; the barrier guarantees mutual exclusion with zero polling.

---
_Source: WWDC19 Session 723 page (abstract, chapter summaries, code samples, and resource links)._
