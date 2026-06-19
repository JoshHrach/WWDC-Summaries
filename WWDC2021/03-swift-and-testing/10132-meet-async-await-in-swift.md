# Meet async/await in Swift
**WWDC21 · Session 10132** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10132/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
Swift 5.5 introduces native async/await support, allowing asynchronous code to be written in a linear, straight-line style that is easier to read, safer, and far less error-prone than completion handler-based approaches. The session uses a thumbnail-fetching example to compare a 20-line completion-handler implementation (with five subtle bug opportunities) to a 6-line async/await version.

The core insight is that `async` functions can *suspend* — cooperatively yielding the thread to the system — rather than blocking it. When suspended, the thread is free to perform other work. The system resumes the function when the awaited result is ready. Swift enforces that every path through an async function either returns a value or throws, eliminating the silent-failure bugs common with completion handlers.

The session also covers bridging existing completion-handler and delegate APIs to async/await using the `withCheckedContinuation` and `withCheckedThrowingContinuation` functions, and shows how XCTest natively supports async test functions.

## Key Topics

**async/await Syntax**
Functions, properties (read-only getters only), and initializers can all be marked `async`. `await` marks where the function may suspend. Multiple async calls in one expression need only one `await`. If an async function also throws, use `try await`.

**Suspension Model**
When an async function suspends, it yields the thread to the system (not to its caller). The system may run other work — including work scheduled after the suspension — before resuming. This cooperative threading model is why `await` is syntactically visible: state can change between lines.

**Async Properties**
Read-only property getters can be `async` (and optionally `throws`). Settable properties cannot be async.

**Async Sequences**
`for await ... in` iterates over `AsyncSequence` types, suspending between each element.

**Testing Async Code**
XCTest supports `async throws` test functions natively — no `XCTestExpectation` needed.

**Bridging to Async with Continuations**
`withCheckedContinuation` and `withCheckedThrowingContinuation` wrap completion-handler APIs into async functions. `continuation.resume(returning:)` / `continuation.resume(throwing:)` must be called exactly once. The Swift runtime detects double-resume (fatal error) and missing resume (warning).

**SDK Auto-Generated Async Alternatives**
The Swift compiler automatically provides `async` variants of Objective-C completion-handler APIs. Async delegate methods can also be adopted by dropping the completion handler parameter and returning the value directly.

**Task for Bridging Sync to Async**
`Task { }` creates an unstructured async task from a synchronous context (e.g., in a SwiftUI `.onAppear` closure), allowing async code to be called from synchronous contexts.

## APIs & Frameworks

### Swift Language / Swift Concurrency **[NEW in Swift 5.5]**
- `async` — function/property/initializer modifier enabling suspension
- `await` — expression prefix indicating a potential suspension point
- `try await` — combined for async throwing calls
- `async throws` — marks a function as both async and throwing
- `for await ... in` — async iteration over `AsyncSequence`
- `Task { }` — creates an unstructured async task from sync context **[NEW]**
- `withCheckedContinuation(_:)` — wraps non-throwing completion handler in async **[NEW]**
- `withCheckedThrowingContinuation(_:)` — wraps throwing completion handler in async **[NEW]**
- `CheckedContinuation<T, E>` — type representing a suspended continuation **[NEW]**
  - `.resume(returning:)` — resumes with a value
  - `.resume(throwing:)` — resumes with an error
  - `.resume(with:)` — resumes with a `Result`

### Foundation
- `URLSession.shared.data(for:delegate:)` — new async variant of `dataTask(with:)` **[NEW]**

### UIKit
- `UIImage.byPreparingThumbnail(ofSize:)` — existing async method (now used with `await`)
- `UIImage.thumbnail` (async property in extension) — custom read-only async computed property example

### XCTest
- Async `XCTestCase` test functions (`func testFoo() async throws`) — no expectation needed **[NEW]**

### ClockKit
- `CLKComplicationDataSource.currentTimelineEntry(for:) async -> CLKComplicationTimelineEntry?` — new async delegate alternative **[NEW]**

### Swift Evolution Proposals Referenced
- SE-0296: Async/await
- SE-0297: Concurrency Interoperability with Objective-C
- SE-0300: Continuations for interfacing async tasks with synchronous code
- SE-0310: Effectful read-only properties

## Code Highlights

Async/await version of fetchThumbnail (6 lines vs. 20 with completion handlers):
```swift
func fetchThumbnail(for id: String) async throws -> UIImage {
    let request = thumbnailURLRequest(for: id)
    let (data, response) = try await URLSession.shared.data(for: request)
    guard (response as? HTTPURLResponse)?.statusCode == 200 else { throw FetchError.badID }
    let maybeImage = UIImage(data: data)
    guard let thumbnail = await maybeImage?.thumbnail else { throw FetchError.badImage }
    return thumbnail
}
```

Async property getter:
```swift
extension UIImage {
    var thumbnail: UIImage? {
        get async {
            let size = CGSize(width: 40, height: 40)
            return await self.byPreparingThumbnail(ofSize: size)
        }
    }
}
```

Wrapping a completion-handler API with a checked continuation:
```swift
func persistentPosts() async throws -> [Post] {
    typealias PostContinuation = CheckedContinuation<[Post], Error>
    return try await withCheckedThrowingContinuation { (continuation: PostContinuation) in
        self.getPersistentPosts { posts, error in
            if let error = error { continuation.resume(throwing: error) }
            else { continuation.resume(returning: posts) }
        }
    }
}
```

XCTest with async:
```swift
func testFetchThumbnails() async throws {
    XCTAssertNoThrow(try await self.mockViewModel.fetchThumbnail(for: mockID))
}
```

## Takeaways
- `async/await` eliminates the error-prone, deeply nested completion-handler pyramid; every code path is guaranteed by the compiler to return or throw.
- `await` is a visible signal that app state may change at that point — the thread is not blocked, and other work may run between lines.
- `withCheckedThrowingContinuation` is the correct pattern for bridging existing callback-based (including delegate) APIs to async Swift.
- XCTest, SwiftUI `Task {}`, and hundreds of SDK APIs all have native async support in Swift 5.5 / iOS 15.

---
_Source: WWDC21 Session 10132 page (abstract, chapter summaries, code samples, and resource links)._
