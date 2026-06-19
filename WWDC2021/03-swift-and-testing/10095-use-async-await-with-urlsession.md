# Use async/await with URLSession
**WWDC21 · Session 10095** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10095/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session demonstrates how to adopt Swift concurrency in URLSession using async/await and `AsyncSequence`. The session opens by showing a common completion-handler-based networking function and identifying three bugs it contains: inconsistent main-queue dispatch, a missing early return that causes the completion handler to be called twice on error, and a missing nil check on `UIImage` initialization. Converting to async/await eliminates all three bugs by making control flow linear and letting the compiler enforce error handling.

New async methods on `URLSession` — `data`, `upload`, `download`, and `bytes` — provide direct equivalents to existing convenience methods but suspend the current concurrency context without blocking a thread. For incremental streaming scenarios, the new `bytes` method returns a `URLSession.AsyncBytes` value that conforms to `AsyncSequence`, allowing the response body to be consumed line-by-line or byte-by-byte as data arrives.

The session also covers task-specific delegates: because the new async methods do not expose the underlying `URLSessionTask`, a per-call delegate argument lets you supply authentication and other callbacks scoped to that single request, avoiding the need to multiplex logic in a shared session delegate.

## Key Topics

### New Async Convenience Methods
`URLSession.data(from:)`, `URLSession.upload(for:fromFile:)`, and `URLSession.download(from:)` are drop-in async replacements for their completion-handler counterparts. `download` no longer auto-deletes the temporary file — the caller must move or delete it. All three support Swift's native `Task` cancellation model.

### Streaming with URLSession.AsyncBytes
`URLSession.bytes(from:)` returns immediately after receiving response headers and exposes the body as `URLSession.AsyncBytes`. The `lines` property on `AsyncBytes` wraps it in an `AsyncLineSequence`, enabling `for try await line in bytes.lines` patterns ideal for newline-delimited JSON or server-sent events.

### Task-Specific Delegates
All four async methods accept an optional `delegate:` argument of type `URLSessionTaskDelegate`. The delegate is strongly retained by the task until it completes or fails. If both the session delegate and task delegate implement the same method, the task delegate wins. Task-specific delegates are not supported in background `URLSession` configurations.

### Cancellation
Swift `Task` cancellation propagates to the underlying `URLSessionTask`. Cancelling the `Task` handle cancels any in-flight network request that the async method is currently awaiting.

## APIs & Frameworks

- **Foundation / URLSession**
- `URLSession.data(from: URL) async throws -> (Data, URLResponse)` **[NEW]**
- `URLSession.data(for: URLRequest, delegate:) async throws -> (Data, URLResponse)` **[NEW]**
- `URLSession.upload(for: URLRequest, fromFile: URL, delegate:) async throws -> (Data, URLResponse)` **[NEW]**
- `URLSession.upload(for: URLRequest, from: Data, delegate:) async throws -> (Data, URLResponse)` **[NEW]**
- `URLSession.download(from: URL, delegate:) async throws -> (URL, URLResponse)` **[NEW]**
- `URLSession.download(for: URLRequest, delegate:) async throws -> (URL, URLResponse)` **[NEW]**
- `URLSession.bytes(from: URL, delegate:) async throws -> (URLSession.AsyncBytes, URLResponse)` **[NEW]**
- `URLSession.bytes(for: URLRequest, delegate:) async throws -> (URLSession.AsyncBytes, URLResponse)` **[NEW]**
- `URLSession.AsyncBytes` **[NEW]** — `AsyncSequence` of `UInt8` for incremental response body consumption
- `URLSession.AsyncBytes.lines` **[NEW]** — `AsyncLineSequence` wrapper for line-by-line iteration
- `NSURLSessionTask.delegate` property **[NEW]** — Objective-C task-specific delegate
- `URLSessionTaskDelegate` protocol — task-level authentication and event callbacks
  - `urlSession(_:task:didReceive:completionHandler:)` — authentication challenge callback (now with async overload)
- `URLSession.AuthChallengeDisposition` — `.useCredential`, `.cancelAuthenticationChallenge`, `.performDefaultHandling`
- `URLCredential(user:password:persistence:)` — credential construction
- `URLAuthenticationChallenge` / `URLProtectionSpace`
- `NSURLAuthenticationMethodHTTPBasic` — protection space auth method constant
- `Task { }` — Swift concurrency task creation
- `Task.cancel()` — cancels underlying URLSessionTask
- `AsyncSequence` — protocol; `for try await` iteration
- `AsyncLineSequence` — line-splitting `AsyncSequence` transformer
- `JSONDecoder().decode(_:from:)` — JSON decoding within async loop
- `HTTPURLResponse` / `statusCode` — response validation

## Code Highlights

**Async data fetch — linear, compiler-checked:**
```swift
func fetchPhoto(url: URL) async throws -> UIImage {
    let (data, response) = try await URLSession.shared.data(from: url)
    guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
        throw WoofError.invalidServerResponse
    }
    guard let image = UIImage(data: data) else {
        throw WoofError.unsupportedImage
    }
    return image
}
```

**Streaming server-sent events line by line:**
```swift
let (bytes, response) = try await URLSession.shared.bytes(from: eventStreamURL)
guard let httpResponse = response as? HTTPURLResponse,
      httpResponse.statusCode == 200 else {
    throw WoofError.invalidServerResponse
}
for try await line in bytes.lines {
    let metadata = try JSONDecoder().decode(PhotoMetadata.self, from: Data(line.utf8))
    await updateFavoriteCount(with: metadata)
}
```

**Task-specific authentication delegate:**
```swift
class AuthenticationDelegate: NSObject, URLSessionTaskDelegate {
    func urlSession(_ session: URLSession, task: URLSessionTask,
                    didReceive challenge: URLAuthenticationChallenge) async
    -> (URLSession.AuthChallengeDisposition, URLCredential?) {
        if challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodHTTPBasic {
            let (user, pass) = try await signInController.promptForCredential()
            return (.useCredential, URLCredential(user: user, password: pass, persistence: .forSession))
        }
        return (.performDefaultHandling, nil)
    }
}
// Usage:
let (data, response) = try await URLSession.shared.data(from: url,
    delegate: AuthenticationDelegate(signInController: signInController))
```

## Takeaways

- The new async URLSession methods (`data`, `upload`, `download`, `bytes`) eliminate entire categories of completion-handler bugs — missing returns, threading errors, and unhandled optionals — by making the Swift compiler enforce correctness.
- `URLSession.AsyncBytes` and its `lines` property enable elegant incremental response streaming with `for try await` loops, ideal for server-sent events and newline-delimited JSON feeds.
- Task-specific delegates allow scoping authentication and metric callbacks to individual requests without complicating shared session delegate logic.
- Swift `Task` cancellation integrates directly with URLSession, so cancelling a `Task` propagates to the underlying network request automatically.

---
_Source: WWDC21 Session 10095 page (abstract, chapter summaries, code samples, and resource links)._
