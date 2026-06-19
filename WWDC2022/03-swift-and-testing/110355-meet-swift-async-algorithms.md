# Meet Swift Async Algorithms
**WWDC22 · Session 110355** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110355/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
Swift Async Algorithms is a new open source package from Apple that extends Swift's concurrency model with a rich set of algorithms for processing values over time using `AsyncSequence`. It complements existing packages like Swift Algorithms and Swift Collections by focusing specifically on asynchronous, time-aware operations.

The session uses a messaging app as a running example to demonstrate how the package can simplify common async patterns: combining multiple data streams, rate limiting user input, batching server requests, and converting async sequences into standard collections. Most of the demonstrated algorithms directly replace complex, error-prone concurrency coordination code.

The package also integrates tightly with the new Swift `Clock`, `Instant`, and `Duration` APIs introduced in Swift 5.7, enabling precise, generic time-based algorithms that work uniformly across the `ContinuousClock` and `SuspendingClock` built-in clock types.

## Key Topics

### Combining Multiple AsyncSequences
- **`zip`** — iterates multiple `AsyncSequence` bases concurrently and produces tuples of paired results; rethrows errors from any base; used to match transcoded videos with their previews before upload.
- **`merge`** — concurrently iterates multiple `AsyncSequence` bases sharing the same `Element` type and interleaves their output into a single sequence; used to unify messages from multiple accounts.

### Clock, Instant, and Duration (Swift 5.7)
- **`Clock` protocol** — defines `now` and `sleep(until:)` primitives.
- **`ContinuousClock`** — progresses even while the device is asleep; best for human-relative durations.
- **`SuspendingClock`** — suspends when the machine sleeps; best for animations and machine-relative timing.
- `clock.measure { }` — measures elapsed execution time of a closure.

### Time-Based Algorithms
- **`debounce(for:clock:)`** — waits for a quiescence period before emitting values; used to rate-limit search queries so the server isn't hit on every keystroke.
- **`chunked(by:)`** — groups elements by a repeating clock interval; used to batch outbound messages into 500 ms windows for efficient server transmission.
- Additional time-aware chunking variants: chunk by count, by time, or by content predicate; errors are rethrown.

### Collection Initializers
- `Array(_:)`, `Dictionary(_:)`, `Set(_:)` initializers that accept a finite `AsyncSequence` and `await` all elements; enables incremental migration of existing collection-based code to Swift concurrency.

## APIs & Frameworks

**Swift Async Algorithms Package** (`swift-async-algorithms`) **[NEW]**
- `zip(_:_:)` — **[NEW]** concurrent zip of two or more `AsyncSequence` bases
- `merge(_:_:)` — **[NEW]** concurrent merge of same-typed `AsyncSequence` bases
- `.debounce(for:clock:)` — **[NEW]** quiescence-based rate limiting on an `AsyncSequence`
- `.chunked(by:)` — **[NEW]** time-interval chunking of an `AsyncSequence`
- `.chunked(count:)` — **[NEW]** count-based chunking
- `.chunked(by:)` (predicate variant) — **[NEW]** content-based chunking

**Swift Standard Library / Swift Concurrency**
- `AsyncSequence` protocol
- `AsyncStream<Element>` — ordered callback-to-async-sequence bridge
- `AsyncChannel<Element>` — **[NEW]** actor-safe channel for sending/receiving values across tasks
- `for try await … in` syntax

**Swift 5.7 Time APIs** **[NEW]**
- `Clock` protocol
- `ContinuousClock` — **[NEW]**
- `SuspendingClock` — **[NEW]**
- `clock.now` — **[NEW]**
- `clock.sleep(until:)` — **[NEW]** async sleep to a deadline
- `clock.measure { }` — **[NEW]** elapsed duration measurement
- `Duration` — **[NEW]** type-safe duration values (`.seconds(_:)`, `.milliseconds(_:)`)

**Collection Initializers (AsyncSequence overloads)** **[NEW]**
- `Array<Element>.init(_: some AsyncSequence)`
- `Dictionary.init(_: some AsyncSequence)`
- `Set<Element>.init(_: some AsyncSequence)`

## Code Highlights

Concurrent zip of video transcoding and preview generation:
```swift
for try await (vid, preview) in zip(videos, previews) {
    try await upload(vid, preview)
}
```

Merging messages from two accounts into one stream:
```swift
for try await message in merge(primaryAccount.messages, secondaryAccount.messages) {
    displayPreview(message)
}
```

Debouncing search input to avoid excessive server calls:
```swift
let queries = searchValues.debounce(for: .milliseconds(300))
for await query in queries {
    let results = try await performSearch(query)
    await channel.send(results)
}
```

Chunking outbound messages into 500 ms batches:
```swift
let batches = outboundMessages.chunked(by: .repeating(every: .milliseconds(500)))
for await batch in batches {
    let data = try encoder.encode(batch)
    try await postToServer(data)
}
```

## Takeaways
- Swift Async Algorithms provides production-ready `zip`, `merge`, `debounce`, and `chunked` operations that replace complex manual concurrency coordination.
- The new `Clock`/`Instant`/`Duration` APIs give time-based algorithms a type-safe, testable foundation; choose `ContinuousClock` for human-relative delays and `SuspendingClock` for machine-relative timing (e.g., animations).
- Collection initializers (`Array`, `Dictionary`, `Set`) accepting `AsyncSequence` make incremental migration from synchronous code straightforward.
- The package is fully open source and under active development at github.com/apple/swift-async-algorithms.

---
_Source: WWDC22 Session 110355 page (abstract, chapter summaries, code samples, and resource links)._
