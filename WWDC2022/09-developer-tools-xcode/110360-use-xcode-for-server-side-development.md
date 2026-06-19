# Use Xcode for Server-Side Development
**WWDC22 · Session 110360** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110360/)

_Platforms:_ macOS Ventura 13

## Overview
This session, presented by a Swift team engineer, demonstrates how to extend an iOS application into the cloud using Swift on the server — all from within Xcode. Starting from a minimal "Hello World" web service, it walks through running a Swift server locally for development, calling it from an iOS app via `URLSession`, deploying to Heroku, and then building a complete Food Truck example that shares `Codable` model types across the server and client in a shared Swift Package.

The central message: Swift is a general-purpose language useful on both sides of the client-server boundary. An Xcode workspace can contain both an iOS app target and a Swift server package, and shared packages eliminate data model duplication and serialization risk.

## Key Topics

### Server Apps as Swift Packages
Server-side Swift applications are modeled as Swift packages with an `.executableTarget`. Adding a web framework dependency (e.g., Vapor) provides routing, request/response handling, and JSON encoding on the wire. The `@main` annotation designates the program entry point, and the `Application` type from Vapor bootstraps the HTTP server.

### Local Development in Xcode
With the server scheme selected and "My Mac" as destination, clicking Run launches the HTTP server locally (default: `127.0.0.1:8080`). Console output shows the listening address. The iOS app simulator can call `localhost` directly. Both schemes can live in the same Xcode workspace, making it easy to run the server and app side-by-side during development.

### Calling the Server from iOS
A lightweight client struct wraps each API endpoint as an `async throws` method, using `URLSession.shared.data(for:)` for async HTTP requests and `JSONDecoder` for response parsing. The SwiftUI `.task { }` modifier triggers the async fetch at view load time, storing the result in `@State`.

### Deployment to Cloud Providers
Swift server apps deploy to any Linux-based cloud provider (AWS, Google Cloud, Azure, Heroku, and others) with Swift support. Heroku's git-push-to-deploy workflow with the community-maintained Swift buildpack provides a simple entry point. After `git push heroku main`, Heroku compiles the Swift package remotely and deploys the binary.

### Swift Actors for Thread-Safe Storage
When the server's `Storage` type gains mutable state (loading a donut menu from a JSON file), it is converted to an `actor` to prevent data races. The `load()` and `listDonuts()` methods are then called from `async` contexts, with Swift's concurrency model handling synchronization automatically — no locks or dispatch queues required.

### Sharing Code Between Client and Server
Duplicate model definitions on client and server are eliminated by extracting a `Shared` library package added to the Xcode workspace. Both the server and iOS app targets declare it as a dependency. Model types conform to `Codable` (for JSON serialization) and are defined once; the server also requires `Content` (Vapor's encoding protocol) at response sites.

### SwiftPM Resources
Static data files (e.g., `menu.json`) are bundled with an executable target using `resources: [.copy("menu.json")]` in `Package.swift`. `Bundle.module.path(forResource:ofType:)` locates the file at runtime, and `FileManager.default.contents(atPath:)` + `JSONDecoder` loads it.

## APIs & Frameworks

**Swift Package Manager**
- `.executableTarget(name:dependencies:resources:)` — declares a runnable server target
- `resources: [.copy("filename")]` — bundles static files into the package
- `Bundle.module.path(forResource:ofType:)` — accesses bundled resources at runtime

**Swift Concurrency**
- `async throws` functions — async HTTP handlers and storage methods
- `actor` — thread-safe storage type; replaces manual locking
- `@main` — designates program entry point

**URLSession (iOS client)**
- `URLSession.shared.data(for: URLRequest)` — async HTTP request
- `JSONDecoder` — decodes `Codable` server responses

**SwiftUI (iOS client)**
- `@State` — stores fetched server data
- `.task { }` — async fetch triggered at view appearance

**Vapor (third-party, open source)**
- `Application()` — Vapor HTTP server bootstrap
- `app.get(_:use:)` / `app.post(_:use:)` — route registration
- `Request` — per-request context
- `Content` protocol — marks a type as JSON-encodable in a Vapor response

## Code Highlights

Minimal Swift server with Vapor:
```swift
import Vapor

@main
public struct MyServer {
    public static func main() async throws {
        let webapp = Application()
        webapp.get("greet", use: Self.greet)
        webapp.post("echo", use: Self.echo)
        try webapp.run()
    }

    static func greet(request: Request) async throws -> String { "Hello from Swift Server" }
    static func echo(request: Request) async throws -> String { request.body.string ?? "" }
}
```

iOS client calling the server:
```swift
struct MyServerClient {
    let baseURL = URL(string: "http://127.0.0.1:8080")!

    func greet() async throws -> String {
        let (data, _) = try await URLSession.shared.data(for: URLRequest(url: baseURL.appendingPathComponent("greet")))
        return String(data: data, encoding: .utf8) ?? ""
    }
}
```

Thread-safe actor storage loading from a bundled JSON resource:
```swift
actor Storage {
    var donuts = [Model.Donut]()
    let jsonDecoder: JSONDecoder

    func load() throws {
        guard let path = Bundle.module.path(forResource: "menu", ofType: "json") else { throw Errors.menuFileNotFound }
        guard let data = FileManager.default.contents(atPath: path) else { throw Errors.failedLoadingMenu }
        self.donuts = try jsonDecoder.decode([Model.Donut].self, from: data)
    }

    func listDonuts() -> [Model.Donut] { donuts }
}
```

## Takeaways
- A single Xcode workspace can contain an iOS app and a Swift server package, enabling local end-to-end development and debugging without any additional tooling.
- Swift actors provide safe, ergonomic concurrency for server-side mutable state with no manual locking or dispatch queue management.
- Sharing `Codable` model types in a standalone Swift package eliminates duplication between client and server and ensures serialization stays in sync.
- Swift server apps deploy to any Linux cloud provider; additional deployment guides and database driver options are documented at [swift.org/server](https://www.swift.org/server/).

---
_Source: WWDC22 Session 110360 page (abstract, transcript, and code samples)._
