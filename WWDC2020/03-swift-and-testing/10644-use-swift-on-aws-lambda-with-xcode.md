# Use Swift on AWS Lambda with Xcode
**WWDC20 · Session 10644** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10644/)

_Platforms:_ macOS Big Sur 11 (server-side), iOS 14, iPadOS 14 (client apps)

## Overview
The Swift AWS Lambda Runtime library (open-source, available on GitHub at swift-server/swift-aws-lambda-runtime) makes it possible to write serverless functions for AWS Lambda entirely in Swift. Swift's low memory footprint (~6 MB per Lambda), deterministic performance, and fast startup time make it an excellent fit for the event-driven, elastic resource model of serverless computing. Starting May 2020, Swift.org publishes official Swift toolchains for Amazon Linux 2, providing the foundation needed to compile and run Swift code in the Lambda execution environment.

The library exposes two API tiers. The closure-based API is the recommended starting point for most use cases—it requires only a single `Lambda.run` call with a closure that receives context, an input event, and a completion handler. For performance-sensitive scenarios, the protocol-based `EventLoopLambdaHandler` gives direct access to the SwiftNIO `EventLoop`, allowing the Lambda to share its thread with the networking stack and avoid context switches, at the cost of additional complexity and the requirement to never block the event loop.

The session demonstrates local debugging via an Xcode multi-target workspace: one target is the Lambda (Swift Package Manager executable), and another is an iOS SwiftUI app. Setting the `LOCAL_LAMBDA_SERVER_ENABLED=true` environment variable activates a built-in local HTTP simulator, allowing Xcode's debugger to attach to both processes simultaneously and set breakpoints across the client and server code.

## Key Topics
- **Swift AWS Lambda Runtime library** — open-source SPM library; implements the AWS Lambda Runtime API **[NEW open-source project]**
- **Amazon Linux 2 toolchain** — Swift.org publishes toolchains for Amazon Linux 2 (CentOS flavor) enabling Lambda deployment **[NEW as of May 2020]**
- **Closure-based Lambda** — `Lambda.run { context, event, callback in ... }`; simplest API; handles lifecycle automatically
- **`EventLoopLambdaHandler` protocol** — `EventLoopLambdaHandler`; `handle(context:event:) -> EventLoopFuture<Out>`; shares NIO EventLoop with networking; no blocking allowed
- **`Codable`-based payloads** — input/output structs conforming to `Codable` provide automatic JSON serialization/deserialization
- **Local debugging** — `LOCAL_LAMBDA_SERVER_ENABLED=true` environment variable; starts local HTTP server on port 7000; Xcode manages both Lambda and iOS app processes
- **Lifecycle loop** — library polls runtime engine for work, invokes user handler, submits result back; long-lived process enables caching (e.g., reuse connections)
- **Stateless design** — Lambda functions must avoid global mutable state; memory reclaimed between invocations
- **Deployment** — Docker image based on Swift.org Amazon Linux 2 image; compile in container, zip executable + dependencies, upload via AWS CLI
- **AWS API Gateway** — exposes Lambda as HTTP endpoint for client applications
- **Event-based triggers** — S3, SQS, SNS, API Gateway, and custom events supported via extensible integration points

## APIs & Frameworks

**AWSLambdaRuntime (Swift Package)**
- `Lambda.run(_:)` — entry point; takes closure `(Lambda.Context, Event, @escaping (Result<Response, Error>) -> Void) -> Void`
- `Lambda.Context` — context object provided to each invocation; contains logger, event loop, deadline, invocation metadata
- `LambdaHandler` protocol — closure-based protocol variant for struct/class adoption
- `EventLoopLambdaHandler` protocol — performance-sensitive variant
  - `associatedtype In` — input type (Codable or raw ByteBuffer)
  - `associatedtype Out` — output type (Codable or raw ByteBuffer)
  - `func handle(context:event:) -> EventLoopFuture<Out>` — invoked per event; must not block the EventLoop
- `Lambda.run(_ handler: EventLoopLambdaHandler)` — run a protocol-based handler

**SwiftNIO (dependency)**
- `EventLoop` — shared event loop; Lambda handler must be non-blocking when using `EventLoopLambdaHandler`
- `EventLoopFuture<Value>` — async result type

**Swift Standard Library**
- `Codable` — JSON serialization for Lambda payloads; structs conforming to `Codable` are automatically encoded/decoded

**Xcode / SPM tooling**
- Swift Package Manager — Lambda project structure; `.executable` product; `dependencies: [.package(url: "…swift-aws-lambda-runtime…")]`
- Xcode multi-target workspace — debug Lambda and iOS app simultaneously
- Environment variable `LOCAL_LAMBDA_SERVER_ENABLED=true` — activates built-in HTTP simulator (listens on localhost:7000, `/invoke` endpoint)

**AWS tools (referenced)**
- Docker — builds Lambda binary in Amazon Linux 2 container using Swift.org toolchain
- AWS CLI — `scripts/deploy` to package zip and upload; `scripts/test` to invoke remotely
- AWS API Gateway — HTTP endpoint routing to Lambda
- AWS Lambda runtime engine — pulls invocations from queue; hibernates process between events

## Code Highlights

Minimal closure-based Lambda (4 lines):
```swift
import AWSLambdaRuntime

Lambda.run { (_, name: String, callback) in
    callback(.success("Hello, \(name)!"))
}
```

Codable payload Lambda (most common pattern):
```swift
import AWSLambdaRuntime

struct Request: Codable {
    let name: String
    let password: String
}
struct Response: Codable {
    let message: String
}

Lambda.run { (_, request: Request, callback) in
    callback(.success(Response(message: "Hello, \(request.name)!")))
}
```

EventLoop-based Lambda (performance-sensitive):
```swift
import AWSLambdaRuntime
import NIO

struct Handler: EventLoopLambdaHandler {
    typealias In = String
    typealias Out = String

    func handle(context: Lambda.Context, event: String) -> EventLoopFuture<String> {
        context.eventLoop.makeSucceededFuture("Hello, \(event)!")
    }
}
Lambda.run(Handler())
```

Local debugging — add to Xcode scheme environment variables:
```
LOCAL_LAMBDA_SERVER_ENABLED = true
```

## Takeaways
- Swift is well-suited for AWS Lambda due to its small memory footprint (~6 MB), fast startup, and type safety; the open-source Swift AWS Lambda Runtime library provides a complete implementation of the AWS Lambda Runtime API as a Swift package.
- Use the closure-based `Lambda.run` API for most functions; switch to `EventLoopLambdaHandler` only when profiling shows the context-switch overhead is meaningful, and never block the EventLoop.
- Set `LOCAL_LAMBDA_SERVER_ENABLED=true` in an Xcode scheme to simulate the Lambda runtime locally, enabling full Xcode debugger support (breakpoints, LLDB) across both the Lambda and its iOS/macOS client simultaneously.
- Lambda functions must be stateless—avoid global mutable state; package the binary and its dylib dependencies into a zip for deployment, using Docker with the Swift.org Amazon Linux 2 toolchain for a reproducible build.

---
_Source: WWDC20 Session 10644 page (abstract, chapter summaries, code samples, and resource links)._
