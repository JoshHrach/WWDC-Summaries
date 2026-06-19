# Advances in Networking, Part 1
**WWDC19 · Session 712** · [Watch](https://developer.apple.com/videos/play/wwdc2019/712/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session covers four major networking advances in iOS 13: Low Data Mode (a user-controlled per-network data-conservation signal), Combine integration with URLSession, native WebSocket support in both URLSession and Network.framework, and significant improvements to network mobility (Wi-Fi Assist and Multipath TCP). All improvements are available exclusively through the high-level networking APIs — URLSession and Network.framework — and not through raw sockets.

The session is paired with Part 2 (session 713), which covers additional networking APIs. The recommended API hierarchy is: URLSession for HTTP and WebSocket, Network.framework for custom protocols and server-side WebSocket, and NetworkExtension for VPN/content filters.

## Key Topics

### Low Data Mode **[NEW]**
- A new user preference (set per Wi-Fi SSID or per cellular SIM) signaling to apps that they should minimize data usage on the current network.
- System effects: background discretionary tasks deferred; Background App Refresh disabled.
- App-side adoption: reduce image quality, eliminate pre-fetching, synchronize less frequently, mark tasks discretionary, disable auto-play video.
- Rule: never block user-initiated requests due to Low Data Mode — only reduce quality of background/proactive operations.
- Correlates to the network path's `isConstrained` property.
- `URLRequest.allowsConstrainedNetworkAccess = false` — request fails with `URLError.networkUnavailableReason == .constrained` when on a Low Data Mode network; app can fall back to a lower-quality fetch.
- `URLSessionConfiguration.allowsConstrainedNetworkAccess = false` — session-wide opt-out.
- `NWPathMonitor` / `NWConnection.currentPath.isConstrained` — check constrained status in Network.framework.
- `NWParameters.prohibitConstrainedPaths = true` — block Network.framework connections on constrained networks.
- Prefer `isConstrained` over `isExpensive` (cellular proxy) or interface-type checks; `isExpensive` is the next-best alternative and futureproofs against new interface types.

### Combine in URLSession **[NEW]**
- `URLSession.DataTaskPublisher` **[NEW]** — a single-value publisher that wraps a data task; conforms to `Publisher` with output `(data: Data, response: URLResponse)` and failure `URLError`.
- Created via `URLSession.dataTaskPublisher(for:)` (URL or URLRequest overloads).
- Integrates with existing URLSession configuration, authentication delegates, and metrics.
- Key operators for networking patterns:
  - `tryCatch` — catch a publisher failure and replace it with a fallback publisher (e.g., Low Data Mode fallback fetch).
  - `tryMap` — transform output and throw on invalid HTTP status codes.
  - `retry(_:)` — resubscribe the upstream publisher N times on failure; use sparingly (low count, idempotent requests only).
  - `replaceError(with:)` — replace failure with a placeholder value.
  - `receive(on:)` — switch to main queue for UI updates.
  - `assign(to:on:)` — bind output directly to a property.
- Storing the subscription as `AnyCancellable` and calling `cancel()` in `prepareForReuse()` eliminates the classic cell-reuse image placement bug.

### WebSocket Support **[NEW]**
- WebSocket provides full-duplex communication over a single HTTP connection; works through proxies, CDNs, and firewalls on standard HTTP ports.
- `URLSessionWebSocketTask` **[NEW]** — Foundation-level WebSocket client built on Network.framework:
  - Created via `URLSession.webSocketTask(with:)`.
  - Call `.resume()` to start the HTTP Upgrade handshake (101 Switching Protocols).
  - `send(_:completionHandler:)` — send a `.string(String)` or `.data(Data)` message.
  - `receive(completionHandler:)` — asynchronously receive the next complete message; call recursively to read a stream of messages.
  - `sendPing(pongReceiveHandler:)` — measure round-trip time.
  - Inherits URLSession configuration, cookie storage, and authentication delegates.
  - New metrics exposed via `URLSessionTaskMetrics` for WebSocket connections.
- `NWConnection` / `NWListener` with WebSocket protocol **[NEW]** — Network.framework WebSocket for both client and server:
  - `NWProtocolWebSocket.Options()` — create WebSocket protocol options; add to `NWParameters.defaultProtocolStack`.
  - `NWConnection.send(content:contentContext:isComplete:completion:)` with `NWConnection.ContentContext` containing `NWProtocolWebSocket.Metadata` — send a framed WebSocket message.
  - Supports partial message receive (specify min/max bytes).
  - Allows custom headers (cookies, auth) via `NWProtocolWebSocket.Options`.
  - Enables server-side WebSocket via `NWListener`.

### Network Mobility Improvements
- **Wi-Fi Assist (iOS 13 enhancements):** cross-layer mobility detection now aggregates signal quality data from Wi-Fi/cellular firmware, and flow progress feedback from Network.framework and URLSession; feeds decisions back to both lower layers (signal improvement efforts) and higher layers (flow recovery). Flows stuck on poor Wi-Fi now recover automatically without user turning Wi-Fi off.
  - Available automatically to all apps using URLSession or Network.framework — no code changes required.
  - Avoid `SCNetworkReachability` pre-flight checks; use `allowsExpensiveNetworkAccess = false` to steer flows away from cellular instead.
- **Multipath TCP (expanded in iOS 13):** Apple Maps and Apple Music now use Multipath TCP (`handover` mode) to seamlessly move flows from Wi-Fi to cellular when walking away from a Wi-Fi access point.
  - Available to third-party apps via `URLSessionConfiguration.multipathServiceType` (`.handover` or `.interactive`) and `NWParameters.multipathServiceType`.
  - Requires server-side Multipath TCP configuration.

## APIs & Frameworks

**Foundation — URLSession**
- `URLRequest.allowsConstrainedNetworkAccess: Bool` **[NEW]** — default `true`; set `false` to fail on Low Data Mode networks
- `URLRequest.allowsExpensiveNetworkAccess: Bool` **[NEW]** — default `true`; set `false` to avoid cellular/hotspot
- `URLSessionConfiguration.allowsConstrainedNetworkAccess: Bool` **[NEW]**
- `URLSessionConfiguration.allowsExpensiveNetworkAccess: Bool` **[NEW]**
- `URLError.networkUnavailableReason` **[NEW]** — `.constrained`, `.expensive`, `.cellular`
- `URLSession.dataTaskPublisher(for:) -> DataTaskPublisher` **[NEW]** — Combine publisher
- `URLSession.DataTaskPublisher` **[NEW]** — `Publisher` with output `(Data, URLResponse)`, failure `URLError`
- `URLSessionWebSocketTask` **[NEW]** — WebSocket task
- `URLSessionWebSocketTask.Message` **[NEW]** — `.string(String)`, `.data(Data)`
- `URLSession.webSocketTask(with:) -> URLSessionWebSocketTask` **[NEW]**
- `URLSessionWebSocketTask.send(_:completionHandler:)` **[NEW]**
- `URLSessionWebSocketTask.receive(completionHandler:)` **[NEW]**
- `URLSessionWebSocketTask.sendPing(pongReceiveHandler:)` **[NEW]**
- `URLSessionConfiguration.multipathServiceType` — `.handover`, `.interactive` (existing, expanded adoption)

**Network.framework**
- `NWPath.isConstrained: Bool` **[NEW]** — true when on Low Data Mode network
- `NWPath.isExpensive: Bool` — existing; true for cellular and Personal Hotspot
- `NWParameters.prohibitConstrainedPaths: Bool` **[NEW]**
- `NWProtocolWebSocket.Options` **[NEW]** — WebSocket protocol configuration
- `NWProtocolWebSocket.Metadata` **[NEW]** — per-message WebSocket frame metadata (opcode, fin)
- `NWParameters.multipathServiceType` — expanded (existing API)

## Code Highlights

```swift
// Low Data Mode adaptive image loading with Combine
func adaptiveImagePublisher(url: URL, lowDataURL: URL) -> AnyPublisher<UIImage, Error> {
    var request = URLRequest(url: url)
    request.allowsConstrainedNetworkAccess = false  // fail fast on Low Data Mode

    return URLSession.shared.dataTaskPublisher(for: request)
        .tryCatch { error -> URLSession.DataTaskPublisher in
            guard let urlError = error as? URLError,
                  urlError.networkUnavailableReason == .constrained else { throw error }
            return URLSession.shared.dataTaskPublisher(for: lowDataURL)
        }
        .tryMap { data, response -> UIImage in
            guard let http = response as? HTTPURLResponse, http.statusCode == 200,
                  let image = UIImage(data: data) else {
                throw URLError(.badServerResponse)
            }
            return image
        }
        .receive(on: DispatchQueue.main)
        .eraseToAnyPublisher()
}

// Cancel subscription on cell reuse to prevent wrong-image bug
class MenuItemCell: UITableViewCell {
    var cancellable: AnyCancellable?
    override func prepareForReuse() {
        super.prepareForReuse()
        cancellable?.cancel()
    }
}
```

```swift
// URLSessionWebSocketTask — client
let task = URLSession.shared.webSocketTask(with: URL(string: "wss://example.com/socket")!)
task.resume()

func readMessage() {
    task.receive { result in
        switch result {
        case .success(let message):
            switch message {
            case .string(let text): print("Received: \(text)")
            case .data(let data): print("Received data: \(data)")
            }
            readMessage()  // chain to receive next message
        case .failure(let error):
            print("WebSocket error: \(error)")
        }
    }
}
readMessage()

task.send(.string("Hello, server!")) { error in
    if let error { print("Send error: \(error)") }
}
```

```swift
// Network.framework WebSocket server (NWListener)
let wsOptions = NWProtocolWebSocket.Options()
let parameters = NWParameters.tls
parameters.defaultProtocolStack.applicationProtocols.insert(wsOptions, at: 0)

let listener = try NWListener(using: parameters, on: 8080)
listener.newConnectionHandler = { connection in
    connection.start(queue: .main)
    // send framed WebSocket message:
    let metadata = NWProtocolWebSocket.Metadata(opcode: .text)
    let context = NWConnection.ContentContext(identifier: "ws", metadata: [metadata])
    connection.send(content: "price updated".data(using: .utf8),
                    contentContext: context, isComplete: true,
                    completion: .idempotent)
}
listener.start(queue: .main)
```

## Takeaways
- Adopt `allowsConstrainedNetworkAccess = false` with a Low Data Mode fallback rather than querying network state proactively — this pattern works with caches, avoids race conditions, and is the recommended iOS 13 pattern.
- `URLSession.DataTaskPublisher` combined with `tryCatch` / `tryMap` / `retry` / `receive(on:)` eliminates most boilerplate in networking code and fixes cell-reuse races via `AnyCancellable` cancellation in `prepareForReuse`.
- `URLSessionWebSocketTask` is the recommended WebSocket client API; use Network.framework's `NWListener` with WebSocket protocol options for server-side or peer-to-peer use cases.
- Avoid `SCNetworkReachability` pre-flight checks — use `allowsExpensiveNetworkAccess`/`allowsConstrainedNetworkAccess` flags instead; Wi-Fi Assist and Multipath TCP handle mobility automatically for URLSession and Network.framework users.

---
_Source: WWDC19 Session 712 page (transcript, abstract, and resource links)._
