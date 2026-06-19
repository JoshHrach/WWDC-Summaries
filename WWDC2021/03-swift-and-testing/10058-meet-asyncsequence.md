# Meet AsyncSequence
**WWDC21 · Session 10058** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10058/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
`AsyncSequence` is a new Swift protocol that allows iterating over a sequence of values produced asynchronously over time, using the same familiar `for` loop syntax as regular `Sequence`. Where a normal sequence's `makeIterator().next()` is synchronous, an async sequence's `makeAsyncIterator().next()` is `async` — it suspends on each element and resumes when the iterator produces a value or throws.

The session demonstrates `AsyncSequence` using a live earthquake data feed, showing how streaming CSV data from a URL can be processed line-by-line as bytes arrive, without waiting for the full download. It covers existing SDK APIs that now vend async sequences (`URL.lines`, `FileHandle.bytes`, `URLSession.bytes(from:)`, `NotificationCenter.notifications(named:)`), the transformation methods available on `AsyncSequence` (map, filter, reduce, dropFirst, etc.), and how to build your own async sequences using `AsyncStream` and `AsyncThrowingStream`.

Building on async/await fundamentals, this session shows that everything that works with regular `Sequence` — including `break`, `continue`, `map`, `filter`, `first(where:)` — works with `AsyncSequence` using `for await` / `for try await` syntax.

## Key Topics

**AsyncSequence Protocol**
Mirrors `Sequence` but with an async iterator. Terminal state is nil from `next()` or a thrown error. After an error, subsequent calls to `next()` return nil. Non-throwing and throwing variants exist.

**for-await-in / for-try-await-in**
The compiler transforms `for await x in seq` into a while loop calling `await iterator.next()`. `break` and `continue` work normally. Throwing sequences require `for try await` inside a `do-catch`.

**Concurrent Iteration with Task**
Wrapping a `for await` loop in a `Task {}` lets iteration run concurrently with other work. Tasks can be cancelled to terminate potentially infinite async sequences.

**SDK Async Sequences (new in iOS 15)**
`URL.lines` and `FileHandle.bytes.lines` for line-by-line file/network reading. `URLSession.bytes(from:)` / `bytes(for:)` for byte-level streaming. `NotificationCenter.notifications(named:object:)` for observing notifications as an async sequence.

**AsyncStream / AsyncThrowingStream**
A concrete, general-purpose type for adapting callback/delegate-based APIs to async sequences. The construction closure receives a `Continuation` with `yield(_:)`, `finish()`, and `onTermination` handler. `AsyncThrowingStream` adds the ability to throw errors from the continuation.

**Sequence Transformation Methods on AsyncSequence**
`map`, `filter`, `reduce`, `dropFirst`, `drop(while:)`, `prefix`, `prefix(while:)`, `first(where:)`, `contains`, `allSatisfy`, `min`, `max`, `compactMap`, `flatMap` — all available as async variants.

## APIs & Frameworks

### Swift Standard Library **[NEW in Swift 5.5]**
- `AsyncSequence` protocol — base protocol for async iteration **[NEW]**
- `AsyncIteratorProtocol` protocol — `mutating func next() async throws -> Element?` **[NEW]**
- `for await ... in` — async iteration syntax **[NEW]**
- `for try await ... in` — async iteration over throwing sequence **[NEW]**
- `AsyncStream<Element>` — concrete async sequence backed by a continuation **[NEW]**
  - `AsyncStream.Continuation.yield(_:)` — produce a value
  - `AsyncStream.Continuation.finish()` — signal completion
  - `AsyncStream.Continuation.onTermination` — cleanup handler
- `AsyncThrowingStream<Element, Failure>` — async sequence that can throw **[NEW]**

### Foundation
- `URL.lines` — `AsyncLineSequence<URL.AsyncBytes>`, async sequence of lines from file or network URL **[NEW]**
- `URL.resourceBytes` — async sequence of bytes from a URL **[NEW]**
- `FileHandle.bytes` — `AsyncBytes` property on `FileHandle` **[NEW]**
- `FileHandle.AsyncBytes.lines` — converts byte sequence to line sequence **[NEW]**
- `URLSession.bytes(from:delegate:) async throws -> (AsyncBytes, URLResponse)` — byte-level streaming from URL **[NEW]**
- `URLSession.bytes(for:delegate:) async throws -> (AsyncBytes, URLResponse)` — byte-level streaming from URLRequest **[NEW]**
- `NotificationCenter.notifications(named:object:) -> AsyncSequence` — notification stream **[NEW]**

### AsyncSequence Transformation Methods **[NEW]**
- `.map(_:)`, `.compactMap(_:)`, `.flatMap(_:)`
- `.filter(_:)`
- `.reduce(_:_:)`, `.reduce(into:_:)`
- `.drop(while:)`, `.dropFirst(_:)`
- `.prefix(_:)`, `.prefix(while:)`
- `.first(where:)`, `.contains(_:)`, `.contains(where:)`
- `.allSatisfy(_:)`, `.min()`, `.min(by:)`, `.max()`, `.max(by:)`

### Swift Evolution Proposals Referenced
- SE-0298: Async/Await: Sequences
- SE-0314: AsyncStream and AsyncThrowingStream

## Code Highlights

Streaming earthquake data line-by-line from a URL:
```swift
let endpointURL = URL(string: "https://earthquake.usgs.gov/...")!
for try await event in endpointURL.lines.dropFirst() {
    let values = event.split(separator: ",")
    print("Magnitude \(values[4]) at \(values[1]) \(values[2])")
}
```

Reading bytes from URLSession:
```swift
let (bytes, response) = try await URLSession.shared.bytes(from: url)
for try await byte in bytes { ... }
```

Observing a specific notification:
```swift
let notification = await NotificationCenter.default
    .notifications(named: .NSPersistentStoreRemoteChange)
    .first { $0.userInfo[NSStoreUUIDKey] == storeUUID }
```

Adapting a callback-based class to `AsyncStream`:
```swift
let quakes = AsyncStream(Quake.self) { continuation in
    let monitor = QuakeMonitor()
    monitor.quakeHandler = { quake in continuation.yield(quake) }
    continuation.onTermination = { @Sendable _ in monitor.stopMonitoring() }
    monitor.startMonitoring()
}
for await quake in quakes.filter({ $0.magnitude > 3 }) { ... }
```

## Takeaways
- `AsyncSequence` brings the full expressiveness of `Sequence` — including `map`, `filter`, and `for` loops — to asynchronous, over-time data sources.
- `AsyncStream` is the recommended way to bridge existing callback/delegate patterns to async sequences; it handles safety, iteration, buffering, and cancellation.
- `URL.lines`, `FileHandle.bytes`, `URLSession.bytes(from:)`, and `NotificationCenter.notifications(named:)` provide ready-made async sequences for the most common data sources.
- Wrap infinite or long-running iterations in `Task {}` to run them concurrently and retain the ability to cancel via `Task.cancel()`.

---
_Source: WWDC21 Session 10058 page (abstract, chapter summaries, code samples, and resource links)._
