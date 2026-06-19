# Explore the Swift on Server Ecosystem
**WWDC24 · Session 10216** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10216/)

_Platforms:_ macOS, Linux (server)

## Overview
Swift powers critical Apple cloud services — iCloud Keychain, Photos, Notes, App Store processing pipelines, SharePlay file sharing, and the new Private Cloud Compute service — handling millions of requests per second. This session surveys the Swift on Server ecosystem by building a practical event-management service from scratch using popular open-source packages: Swift OpenAPI Generator, Vapor, PostgresNIO, swift-log, swift-metrics, and swift-distributed-tracing.

The Swift Server Workgroup (founded 2016, the oldest Swift workgroup) governs package incubation through Sandbox → Incubating → Graduated maturity levels, ensuring quality and reducing duplication across the ecosystem.

## Key Topics

### Why Swift for Server
Swift offers C-like performance with a low memory footprint (ARC instead of garbage collection), predictable resource consumption, fast startup time, strong typing, optionals, memory safety, and first-class structured concurrency — all of which eliminate data races and make distributed systems more robust.

### Build a Service
The demo builds an `EventService` using Vapor as the HTTP server and Swift OpenAPI Generator to define and serve the API. The service has two endpoints: `GET /events` (list all events) and `POST /events` (create a new event). The OpenAPI YAML document describes both operations, and the SPM build plugin generates all Swift types and routing glue.

### Swift OpenAPI Generator
An SPM build plugin generates Swift server and client code from an `openapi.yaml` document. The `EventAPI` target uses the `OpenAPIGenerator` plugin; the `EventService` target conforms to the generated `APIProtocol`. The `swift-openapi-vapor` package provides the `VaporTransport` that bridges the generated protocol to Vapor's router.

### Database Drivers: PostgresNIO 1.21
PostgresNIO (maintained by Vapor and Apple) gains a **new `PostgresClient`** in version 1.21 with a completely new async interface, a built-in connection pool leveraging structured concurrency, resilience against intermittent network failures, and row prefetching via `AsyncSequence`. Queries use Swift string interpolation to produce parameterized queries — safe from SQL injection by construction. `PSQLError.serverInfo` provides detailed error information while keeping it out of the public description to prevent accidental schema leakage.

### Observability
Three pillars: logging (`swift-log`), metrics (`swift-metrics`), and distributed tracing (`swift-distributed-tracing`). All three APIs are backend-agnostic; bootstrap them at startup (LoggingSystem first, then MetricsSystem, then InstrumentationSystem). The session instruments `listEvents` with a log, a counter increment, and a tracing span around the database query.

### Ecosystem Exploration
Find packages at swift.org/packages (server category), Swift Package Index, and the Swift Server Workgroup incubation list. The ecosystem covers networking, database drivers, observability, message streaming, and more.

## APIs & Frameworks

**Swift OpenAPI Generator** (`apple/swift-openapi-generator`)
- SPM `OpenAPIGenerator` build plugin — generates types from `openapi.yaml`
- `APIProtocol` — generated server-side conformance protocol
- `Operations.<operationId>.Input` / `.Output` — generated request/response types
- `swift-openapi-vapor` — `VaporTransport` bridging OpenAPI to Vapor **[companion package]**
- `swift-openapi-runtime` — `OpenAPIRuntime` module used by generated code

**Vapor** (`vapor/vapor`)
- `Vapor.Application.make()` async factory **[updated]**
- `VaporTransport(routesBuilder:)` — from `swift-openapi-vapor`
- `service.registerHandlers(on:serverURL:)` — generated registration method
- `application.execute()` — starts HTTP server

**PostgresNIO** (`vapor/postgres-nio`) 1.21
- `PostgresClient` **[NEW]** — new async interface with built-in connection pool
  - `PostgresClient(configuration:)` init
  - `PostgresClient.Configuration(host:username:password:database:tls:)`
  - `postgresClient.run()` async — starts the client (runs until done)
  - `postgresClient.query(_:)` async — returns `AsyncSequence` of rows with prefetching
  - `rows.decode(_:)` — strongly-typed row decoding
- `PSQLError` — error type; intentionally omits detailed info in description
  - `PSQLError.serverInfo` — dictionary of server-sent details
  - `PSQLError.serverInfo[.message]` — the concrete Postgres error message
- SQL injection safety via Swift string interpolation into parameterized queries **[key behavior]**

**swift-log** (`apple/swift-log`)
- `Logger(label:)` — create a logger
- `logger.info(_:metadata:)` — structured logging with metadata
- `LoggingSystem.bootstrap(_:)` — select logging backend at startup

**swift-metrics** (`apple/swift-metrics`)
- `Counter(label:)` — create a counter
- `counter.increment()` — increment the counter
- `MetricsSystem.bootstrap(_:)` — select metrics backend (e.g., Prometheus)

**swift-distributed-tracing** (`apple/swift-distributed-tracing`)
- `withSpan(_:) { span in ... }` async — create a tracing span
- `InstrumentationSystem.bootstrap(_:)` — select tracing backend (e.g., OpenTelemetry)

**Concurrency Patterns**
- `withThrowingDiscardingTaskGroup { group in ... }` — run `PostgresClient` and Vapor concurrently
- `group.addTask { ... }` — child tasks in the group
- `AsyncSequence` row iteration for database results

## Code Highlights

OpenAPI-generated service with Vapor transport:
```swift
@main
struct Service {
    static func main() async throws {
        let application = try await Vapor.Application.make()
        let transport = VaporTransport(routesBuilder: application)
        let service = Service()
        try service.registerHandlers(on: transport, serverURL: URL(string: "/api")!)
        try await application.execute()
    }
}
extension Service: APIProtocol {
    func listEvents(_ input: Operations.listEvents.Input) async throws -> Operations.listEvents.Output {
        return .ok(.init(body: .json(events)))
    }
}
```

PostgresNIO with structured concurrency and async row decoding:
```swift
let rows = try await self.postgresClient.query("SELECT name, date, attendee FROM events")
var events = [Components.Schemas.Event]()
for try await (name, date, attendee) in rows.decode((String, String, String).self) {
    events.append(.init(name: name, date: date, attendee: attendee))
}
```

SQL injection-safe parameterized INSERT via string interpolation:
```swift
try await self.postgresClient.query(
    """
    INSERT INTO events (name, date, attendee)
    VALUES (\(event.name), \(event.date), \(event.attendee))
    """
)
```

Observability: logging, metrics, and tracing together:
```swift
let logger = Logger(label: "ListEvents")
logger.info("Handling request", metadata: ["operation": "\(Operations.listEvents.id)"])
Counter(label: "list.events.counter").increment()
return try await withSpan("database query") { span in
    let rows = try await postgresClient.query("SELECT name, date, attendee FROM events")
    return try await .ok(.init(body: .json(decodeEvents(rows))))
}
```

Running PostgresClient and Vapor server concurrently:
```swift
try await withThrowingDiscardingTaskGroup { group in
    group.addTask { await postgresClient.run() }
    group.addTask { try await application.execute() }
}
```

## Takeaways
- Swift is production-proven at Apple scale (millions of req/s, Private Cloud Compute); its performance, safety, and structured concurrency make it compelling for server workloads.
- The new `PostgresClient` in PostgresNIO 1.21 provides built-in connection pooling with structured concurrency — replace raw connection management with it; SQL injection safety via string interpolation is a standout ergonomic win.
- Use `swift-openapi-generator` as an SPM plugin to generate all request/response types and routing from a single `openapi.yaml`; it works with both Vapor and Hummingbird transports.
- Bootstrap observability (swift-log → swift-metrics → swift-distributed-tracing) at the very start of `main()` so no events are lost; all three APIs are backend-agnostic.

---
_Source: WWDC24 Session 10216 page (abstract, chapter summaries, code samples, and resource links)._
