# Build Real-Time Apps and Services with gRPC and Swift
**WWDC26 · Session 265** · [Watch](https://developer.apple.com/videos/play/wwdc2026/265/)

_Platforms:_ iOS, iPadOS, macOS, Swift on Server (Linux)

## Overview
This session introduces gRPC as a first-class networking option for Swift developers building real-time, bidirectional streaming features. gRPC is a CNCF-standard remote procedure call framework that uses Protocol Buffers to define APIs as typed, version-safe function calls rather than hand-crafted HTTP endpoints. The official `grpc-swift` package provides a modern, Swift-concurrency-based runtime that integrates naturally with `async`/`await` and `AsyncSequence`.

The session is structured around a complete end-to-end demo: a SwiftKart go-karting iOS app that retrieves a race schedule from a Swift server and then streams live kart positions and race standings using a bidirectional RPC. Both the client and server are written in Swift, sharing the same `.proto` schema, and the server is ultimately containerized and deployed to a cloud platform.

A key design principle throughout is lifecycle management: managing the gRPC client connection as a shared app-level object, disconnecting when the app backgrounds, and propagating errors correctly through the SwiftUI environment.

## Key Topics

### What is gRPC
gRPC is an open-source RPC framework backed by CNCF. APIs are defined in `.proto` files using Protocol Buffers (edition 2024 syntax). The framework generates Swift types for all messages and stubs for client and server code. All communication runs over HTTP/2, supporting four call patterns: unary, server-streaming, client-streaming, and bidirectional-streaming.

### Setting Up Xcode
Add three Swift packages:
1. `grpc-swift` — core runtime (`GRPCCore`)
2. `grpc-swift-nio-transport` — HTTP/2 transport layer (`GRPCNIOTransportHTTP2`)
3. `grpc-swift-protobuf` — Protobuf message codegen integration (`SwiftProtobuf`)

A `grpc-swift-proto-generator-config.json` file in the target folder tells the `GRPCProtobufGenerator` build plugin which Swift artifacts to generate (clients, servers, messages). The plugin runs automatically during the Xcode build, meaning generated files never need to be checked in.

### Defining a Unary RPC (`ListRaces`)
The `.proto` file defines a `service` with `rpc` declarations and `message` types. Field numbers (not names) are used in the binary encoding, making messages roughly half the size of equivalent JSON. Protobuf Well Known Types (`google.protobuf.Timestamp`, `google.protobuf.Duration`) are available via import.

### Managing the gRPC Client Lifecycle
Rather than creating a new `GRPCClient` per view, a shared `ClientManager` (`@Observable`, `Sendable`) holds the client behind a `Mutex<State>`. It exposes a `withClient(body:)` method, connects lazily on first use, and calls `beginGracefulShutdown()` when the SwiftUI `scenePhase` transitions to `.background`. Inject it into the SwiftUI environment as a single instance.

### Protobuf Binary Format
Fields are identified by number, not name, so wire encoding is compact. The session demonstrates manually constructing a `Race` message and calling `serializedBytes()` to see the binary output — approximately 50% smaller than equivalent JSON, important for mobile data usage and service-to-service traffic.

### Bidirectional Streaming (`FollowRace`)
Adding `stream` keyword before both the request and response types in the `.proto` file defines a bidirectional streaming RPC. On the server, the handler receives an `RPCAsyncSequence<FollowRaceRequest, Error>` for incoming messages and an `RPCWriter<FollowRaceResponse>` for writing outgoing messages. A `withThrowingTaskGroup` handles concurrent read/write loops: one task processes live event data and writes responses, while the main task consumes the request stream to dynamically update subscribed event types. On the client, `kart.followRace(requestStream:onResponse:)` takes two closures — one to write request messages and one to iterate response messages.

### Deploying the Service
The Swift server is containerized using a multi-stage `Containerfile`: a `swift:latest` builder stage compiles with `swift build -c release`, then a `swift:slim` runtime stage copies only the binary. The container is pushed to a registry and deployed to Google Cloud Run with `--use-http2` to preserve HTTP/2 end-to-end. The client's `makeTransport()` is updated to target the deployed DNS hostname and enable TLS.

## APIs & Frameworks

**gRPC Swift packages**
- **[NEW]** `GRPCCore` — `GRPCClient`, `GRPCServer`, `RPCError`, `ServerContext`, `RPCAsyncSequence`, `RPCWriter`
- **[NEW]** `GRPCNIOTransportHTTP2` — `HTTP2ClientTransport.TransportServices` (`.http2NIOTS`), `HTTP2ClientTransport.Posix` (`.http2NIOPosix`)
- **[NEW]** `SwiftProtobuf` — `Message`, `serializedBytes()`, `google.protobuf.Timestamp`, `google.protobuf.Duration`
- **[NEW]** `GRPCProtobufGenerator` — Xcode build plugin for `.proto` → Swift code generation
- **[NEW]** `grpc-swift-proto-generator-config.json` — build plugin configuration

**Client API**
- `withGRPCClient(transport:body:)` — creates a scoped gRPC client
- `GRPCClient<Transport>` — reusable client handle
- `client.runConnections()` — async task that maintains the transport connection
- `client.beginGracefulShutdown()` — initiates graceful disconnect
- `SwiftKartService.Client(wrapping:)` — generated service-specific client wrapper
- `.http2NIOTS(target:transportSecurity:)` — NIO Transport Services transport (iOS/macOS)
- `.http2NIOPosix(address:transportSecurity:)` — POSIX NIO transport (Linux server)
- `TransportSecurity.tls` / `.plaintext`
- `NetworkAddress.ipv4(host:port:)` / `.dns(host:)`

**Server API**
- `GRPCServer(transport:services:)` — server constructor
- `server.serve()` — runs the server (async, long-lived)
- `SwiftKartService.SimpleServiceProtocol` — generated simple (unary) service protocol
- `RPCAsyncSequence<Request, Error>` — incoming streaming request sequence
- `RPCWriter<Response>` — outgoing streaming response writer
- `RPCError(code:message:)` — structured RPC error (e.g., `.unimplemented`)

**SwiftUI integration**
- `@Observable` on `ClientManager` for environment propagation
- `Mutex<State>` (`Synchronization` framework) for thread-safe state mutation
- `@Environment(ClientManager.self)` injection
- `.onChange(of: scenePhase)` for lifecycle-driven disconnect

**Protobuf message patterns**
- `message`, `service`, `rpc`, `stream`, `repeated`, `oneof`, `enum` — `.proto` keywords
- `edition = "2024"` — modern proto edition
- `google.protobuf.Timestamp` / `google.protobuf.Duration` Well Known Types

## Code Highlights

**Minimal unary call:**
```swift
try await withGRPCClient(
    transport: .http2NIOTS(address: .ipv4(host: "127.0.0.1", port: 8080), transportSecurity: .tls)
) { client in
    let kart = SwiftKartService.Client(wrapping: client)
    let response = try await kart.listRaces(ListRacesRequest())
}
```

**Bidirectional streaming call (client side):**
```swift
try await kart.followRace { requestStream in
    for await showLeaderboard in stream {
        var message = FollowRaceRequest()
        message.raceName = race.name
        message.eventTypes = showLeaderboard ? [.kartLocations, .standings] : [.kartLocations]
        try await requestStream.write(message)
    }
} onResponse: { responseStream in
    for try await message in responseStream.messages {
        await handleEvent(message.event!)
    }
}
```

**Server lifecycle:**
```swift
let server = GRPCServer(
    transport: .http2NIOPosix(address: .ipv4(host: "127.0.0.1", port: 8080), transportSecurity: .plaintext),
    services: [Service()]
)
try await server.serve()
```

## Takeaways
- Add the three gRPC Swift packages and configure `grpc-swift-proto-generator-config.json` to automate Swift code generation from `.proto` files at build time — no manual codegen step required.
- Lift the `GRPCClient` into a shared `ClientManager` with `Mutex`-backed state and disconnect on `scenePhase == .background` to avoid unnecessary connection overhead.
- Use bidirectional streaming RPCs for real-time features: one request stream lets the client dynamically adjust its subscription, while the response stream pushes server-initiated updates.
- Containerize the Swift server with a multi-stage `Containerfile` and deploy with `--use-http2` on any HTTP/2-capable cloud platform.

---
_Source: WWDC26 Session 265 page (abstract, chapter summaries, code samples, and resource links)._
