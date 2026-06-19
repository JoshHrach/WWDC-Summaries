# Build Robust and Resumable File Transfers
**WWDC23 · Session 10006** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10006/)

_Platforms:_ iOS 17, macOS Sonoma 14, watchOS 10, tvOS 17

## Overview
Large file transfers are inherently fragile: Wi-Fi drops, servers go down, and users navigate away from apps. This session explains how URLSession can handle these scenarios gracefully using HTTP-level resumable protocols for both downloads and uploads, so that interrupted transfers pick up where they left off rather than restarting from scratch.

The session covers the HTTP protocol mechanics behind resumable downloads (range requests, ETags) and the newly standardized resumable uploads protocol (Upload-Incomplete, 104 informational responses, PATCH with Upload-Offset). iOS 17 introduces brand-new URLSession API for resumable upload tasks, mirroring the long-standing download resumption API.

For server-side developers using Swift on Server, the session shows how to wrap any SwiftNIO HTTP/2 server with the new `NIOResumableUpload` package in just a few lines. The session closes with best practices for background URLSession—discretionary scheduling, Low Data Mode handling, and expected byte count hints.

## Key Topics

### Resumable Downloads (HTTP Range Requests)
The server advertises `Accept-Ranges: bytes` and supplies an `ETag`. When a download is interrupted, the client sends a range request with `Range` and `If-Range` headers referencing the ETag; the server responds with `206 Partial Content` to continue delivery.

### Pausing and Resuming Downloads with URLSession
`URLSessionDownloadTask.cancelByProducingResumeData()` returns an opaque `Data` blob containing the ETag, partial file location, and other metadata. Passing this blob to `session.downloadTask(withResumeData:)` creates a new task that continues from the interruption point. On failure, `URLError.downloadTaskResumeData` exposes the same blob from the thrown error.

Requirements: GET method only; server must send `Accept-Ranges`, `ETag` or `Last-Modified`; the partial file must not have been evicted by the system.

### Resumable Uploads (iOS 17 NEW)
Upload tasks now support the same `cancelByProducingResumeData()` / `uploadTask(withResumeData:)` pattern as download tasks. URLSession auto-detects server support using the `Upload-Incomplete` header. If the server does not support resumable uploads, the request falls back to a plain upload transparently. On transient network interruptions where the server is still reachable, URLSession automatically resumes uploads without any app-level code.

### Resumable Uploads Protocol (HTTP Draft)
1. Client sends request with `Upload-Incomplete: ?0`.
2. Server responds with a `104` informational response containing a `Location` resume URL.
3. If interrupted, client sends `HEAD` to the resume URL to get `Upload-Offset`.
4. Client resumes with `PATCH` to the resume URL starting at the acknowledged offset.

### NIOResumableUpload (SwiftNIO)
Adding the `NIOResumableUpload` package and wrapping an existing `ChannelHandler` in `HTTPResumableUploadHandler` gives any SwiftNIO HTTP/2 server full resumable upload support.

### Informational Responses
`URLSessionTaskDelegate.urlSession(_:task:didReceiveInformationalResponse:)` is new in iOS 17, exposing 1xx responses (102, 103, 104) to app code. URLSession handles the 104 automatically for upload resumption.

### Background URLSession
Background sessions schedule tasks outside the app process, surviving suspension and termination. They automatically retry or resume interrupted transfers. Key configuration properties: `isDiscretionary`, `allowsConstrainedNetworkAccess`, `earliestBeginDate`, `countOfBytesClientExpectsToSend`, `countOfBytesClientExpectsToReceive`.

## APIs & Frameworks

**Foundation / URLSession**
- `URLSession` — session object for network tasks
- `URLSessionDownloadTask` — download task with file-level resume support
- `URLSessionUploadTask` — upload task, now with resume support **[NEW]**
- `URLSessionTask.cancelByProducingResumeData()` async — pauses task and returns resume data **[NEW for uploads]**
- `URLSession.downloadTask(withResumeData:)` — creates a resumable download continuation
- `URLSession.uploadTask(withResumeData:)` **[NEW]** — creates a resumable upload continuation
- `URLError.downloadTaskResumeData` — resume data extracted from a download failure error
- `URLError.uploadTaskResumeData` **[NEW]** — resume data extracted from an upload failure error
- `URLSessionConfiguration.background(withIdentifier:)` — background session configuration
- `URLSessionConfiguration.isDiscretionary` — allow system to schedule tasks opportunistically
- `URLSessionConfiguration.allowsConstrainedNetworkAccess` — opt out of Low Data Mode transfers
- `URLSessionTask.earliestBeginDate` — delay task scheduling
- `URLSessionTask.countOfBytesClientExpectsToSend` — scheduling hint
- `URLSessionTask.countOfBytesClientExpectsToReceive` — scheduling hint
- `URLSessionTaskDelegate.urlSession(_:task:didReceiveInformationalResponse:)` **[NEW]** — 1xx response callback
- `HTTPURLResponse` — HTTP response including informational responses

**NIOResumableUpload (Swift Package)**
- `NIOResumableUpload` — new open-source package **[NEW]**
- `HTTPResumableUploadContext(origin:)` — configures resume URL generation **[NEW]**
- `HTTPResumableUploadHandler(context:handlers:)` — channel handler wrapping existing handlers **[NEW]**

**HTTP Types (open-source)**
- `HTTPRequest`, `HTTPResponse` — shared request/response types across app and server **[NEW]**

## Code Highlights

Pausing and resuming a download:
```swift
let downloadTask = session.downloadTask(with: request)
downloadTask.resume()

guard let resumeData = await downloadTask.cancelByProducingResumeData() else { return }
let newTask = session.downloadTask(withResumeData: resumeData)
newTask.resume()
```

Resuming an upload after error:
```swift
do {
    let (data, response) = try await session.upload(for: request, fromFile: fileURL)
} catch let error as URLError {
    guard let resumeData = error.uploadTaskResumeData else { return }
}
```

Background session configuration:
```swift
let configuration = URLSessionConfiguration.background(withIdentifier: "com.example.app")
configuration.isDiscretionary = true
configuration.allowsConstrainedNetworkAccess = false
let session = URLSession(configuration: configuration, delegate: self, delegateQueue: nil)

let backgroundTask = session.uploadTask(with: url, fromFile: fileURL)
backgroundTask.earliestBeginDate = .now.addingTimeInterval(60 * 60)
backgroundTask.countOfBytesClientExpectsToSend = 500 * 1024
backgroundTask.countOfBytesClientExpectsToReceive = 200
```

## Takeaways
- iOS 17 brings resumable upload tasks to URLSession with the same API pattern as long-standing resumable downloads.
- The underlying HTTP resumable uploads protocol is being standardized at the IETF; URLSession detects server support automatically and falls back gracefully.
- SwiftNIO servers can add resumable upload support in three lines using the new `NIOResumableUpload` package.
- Background URLSession with `isDiscretionary`, `allowsConstrainedNetworkAccess`, and scheduling hints maximizes efficiency for large or non-urgent transfers.

---
_Source: WWDC23 Session 10006 page (abstract, chapter summaries, code samples, and resource links)._
