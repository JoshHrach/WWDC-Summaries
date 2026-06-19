# Meet Swift OpenAPI Generator
**WWDC23 · Session 10171** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10171/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10, tvOS 17, visionOS 1 (cross-platform Swift; server targets run on Linux)

## Overview
Swift OpenAPI Generator is an open-source Swift Package Plugin that runs at build time to generate type-safe Swift code from an OpenAPI 3.0 document. The generated code is never committed to source control — it is always in sync with the spec file and rebuilt automatically when the spec changes. This enables spec-driven development: both the iOS client and the Swift server start from the same single source of truth.

The plugin generates three categories of code: shared types (from `schemas`), a client (for apps making API calls), and server stubs (for apps implementing the API). The generated `Client` type and server `APIProtocol` protocol are decoupled from any specific HTTP transport library, making it easy to swap transports, write mock clients for testing, and share the generated types across platforms.

## Key Topics

### OpenAPI Documents
- Written in YAML or JSON; describes endpoints, HTTP methods, parameters, request/response schemas, and server URLs
- Machine-readable format enabling code generation, documentation generation, test generation, and runtime validation
- OpenAPI 3.0.3 used in the session's examples
- Each operation has an `operationId` (e.g., `getGreeting`, `getEmoji`) that maps directly to generated Swift function names

### Swift OpenAPI Generator Plugin
- Declared as a build tool plugin in the target's `plugins` array in `Package.swift`
- Expects two files in the target's source directory:
  - `openapi.yaml` (or `.json`) – the OpenAPI document
  - `openapi-generator-config.yaml` – specifies what to generate (`types`, `client`, `server`)
- Generated code is available immediately after build; no manual step required
- Plugin is trusted once via the Xcode "Run Build Tool Plug-ins" security prompt

### Client Generation (iOS/App targets)
- Generated `Client` type initialized with a server URL and a transport
- `Client` conforms to the generated `APIProtocol` protocol
- API calls are async Swift functions with type-safe input/output
- Response is a rich enum covering all documented status codes and content types; `undocumented(statusCode:_:)` case handles unexpected responses
- Parameters are strongly typed nested structs (`Operations.operationId.Input.Query`)

### Server Generation (Server targets)
- Generated `APIProtocol` protocol with one async function per operation
- Implement the protocol to provide business logic only — no routing boilerplate
- Generated `registerHandlers(on:serverURL:)` function wires up the handler to any compatible HTTP server framework
- Demonstrated with Vapor (`swift-openapi-vapor` integration package)

### Transport Abstraction
- Client-side: `swift-openapi-urlsession` provides `URLSessionTransport` for iOS/macOS
- Server-side: `swift-openapi-vapor` provides `VaporTransport`
- Custom transports can be written for other HTTP libraries
- Transport-agnostic design enables testing without network

### Mock Client for Testing
- Make the view/caller generic over `C: APIProtocol`
- Define a `MockClient: APIProtocol` with deterministic test responses
- Inject `MockClient()` in Xcode Previews; production uses `Client` with real transport
- Compiler enforces that all operations are implemented in the mock

### Spec-Driven Development Workflow
- Add a new operation to `openapi.yaml` → rebuild → compiler error on handler → fill in business logic
- Ensures API contract and implementation stay in sync

## APIs & Frameworks

- **Swift OpenAPI Generator** **[NEW]** – Swift Package Plugin (`swift-openapi-generator` on GitHub)
- **Swift OpenAPI Runtime** **[NEW]** – `swift-openapi-runtime`; provides `APIProtocol`, `Operations`, common types
- **URLSession Transport** **[NEW]** – `swift-openapi-urlsession`; `URLSessionTransport` for client apps
- **Vapor Transport** – `swift-openapi-vapor`; `VaporTransport` for server apps
- Generated `Client` type **[NEW]** – conforms to `APIProtocol`; initialized with `serverURL` + `transport`
- Generated `Servers.server1()` – type-safe server URL from the OpenAPI document
- Generated `APIProtocol` protocol **[NEW]** – one function per operation; used for both real and mock implementations
- `registerHandlers(on:serverURL:)` **[NEW]** – generated function that registers server routes
- `Operations.<operationId>.Input` – generated input type; contains `query`, `headers`, `path`, `body` sub-structs
- `Operations.<operationId>.Output` – generated output enum; one case per documented status code + `undocumented(statusCode:_:)`
- `openapi-generator-config.yaml` – plugin configuration; `generate:` list of `types`, `client`, `server`
- **OpenAPI 3.0** specification – YAML/JSON API description format
- **Vapor** – open-source Swift web framework used for server demo
- Swift Package Plugins (`plugins:` in `Package.swift`) – build-time code generation host

## Code Highlights

Plugin configuration file (`openapi-generator-config.yaml`):
```yaml
generate:
  - types
  - client
```

Making a type-safe API call from the app:
```swift
import OpenAPIRuntime
import OpenAPIURLSession

let client = Client(
    serverURL: try! Servers.server1(),
    transport: URLSessionTransport()
)

func updateEmoji(count: Int = 1) async throws {
    let response = try await client.getEmoji(Operations.getEmoji.Input(
        query: Operations.getEmoji.Input.Query(count: count)
    ))
    switch response {
    case let .ok(okResponse):
        switch okResponse.body {
        case .text(let text):
            emoji = text
        }
    case .undocumented(statusCode: let statusCode, _):
        print("Unexpected status: \(statusCode)")
    }
}
```

Mock client for Xcode Previews:
```swift
struct MockClient: APIProtocol {
    func getEmoji(_ input: Operations.getEmoji.Input) async throws -> Operations.getEmoji.Output {
        let count = input.query.count ?? 1
        return .ok(Operations.getEmoji.Output.Ok(body: .text(String(repeating: "🤖", count: count))))
    }
}
```

Server handler with Vapor:
```swift
struct Handler: APIProtocol {
    func getEmoji(_ input: Operations.getEmoji.Input) async throws -> Operations.getEmoji.Output {
        let count = input.query.count ?? 1
        return .ok(Operations.getEmoji.Output.Ok(body: .text(String(repeating: "🐱", count: count))))
    }
}

@main struct CatService {
    static func main() throws {
        let app = Vapor.Application()
        let transport = VaporTransport(routesBuilder: app)
        try Handler().registerHandlers(on: transport, serverURL: Servers.server1())
        try app.run()
    }
}
```

## Takeaways
- Swift OpenAPI Generator eliminates repetitive HTTP request/response boilerplate; the entire encoding, decoding, routing, and error-handling layer is generated from the spec at build time.
- Because the generated `Client` conforms to a protocol, writing a `MockClient` for Xcode Previews and tests is trivial and requires no network or running server.
- The same `openapi.yaml` file can drive both the iOS client and the Swift server, making it the single source of truth for API contracts.
- Adding a new operation to the spec causes an immediate compiler error on the server handler — enforcing that implementations stay in sync with the contract.

---
_Source: WWDC23 Session 10171 page (abstract, chapter summaries, code samples, and resource links)._
