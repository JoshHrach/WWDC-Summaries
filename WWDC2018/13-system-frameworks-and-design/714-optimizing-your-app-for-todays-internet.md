# Optimizing Your App for Today's Internet
**WWDC18 · Session 714** · [Watch](https://developer.apple.com/videos/play/wwdc2018/714/)

_Platforms:_ iOS 12, macOS Mojave 10.14, watchOS 5, tvOS 12

## Overview
This session provides a comprehensive update on networking technologies and URLSession best practices for iOS 12 and macOS Mojave. The first half covers the state of the internet — IPv6 adoption (87% of US mobile carriers), Explicit Congestion Notification (77% of Alexa top-million), Multipath TCP (78% of Apple carrier partners), and upcoming protocols including QUIC and TLS 1.3. The key message is that Apple's high-level APIs — URLSession and the new Network.framework — handle all the heavy lifting so developers should stop using BSD Sockets and raw socket wrappers.

The second half is a deep dive into URLSession best practices organized around four goals: reducing latency (HTTP/2, connection coalescing), maximizing throughput (HTTP compression, cookie management), increasing responsiveness (QoS, `waitsForConnectivity`, new `responsiveData` service type), and making better use of system resources (background sessions, cache control).

## Key Topics

### Internet State of the Union
- IPv6 delivers up to 2x faster TCP connection setup on cellular (e.g., in India: 150 ms at 75th percentile vs. 325 ms for IPv4)
- Explicit Congestion Notification (ECN) enabled by default on iOS/macOS — ensures server support
- Multipath TCP — seamlessly moves connections between Wi-Fi and cellular without reconnect
- TCP Fast Open — embed initial data in TCP handshake to skip a round-trip
- **QUIC** — new IETF transport protocol (successor to TCP); Apple engineers actively contributing; API support coming when standard finalizes
- **TLS 1.3** — improved security and reduced handshake latency; final standard approved, turning on by default later in 2018; test now using the opt-in flag
- **Certificate Transparency** — all newly issued TLS certificates must be logged in public CT logs; Apple will enforce this later in 2018; no app changes needed but server cert authorities must publish

### Networking API Hierarchy
- BSD Sockets: avoid — designed for a world without Wi-Fi, IPv6, or multi-homing
- Third-party socket wrappers: avoid — same limitations
- **Network.framework** **[NEW in iOS 12]** — exposes the same user-space networking stack used internally by URLSession; for custom TCP/UDP protocols
- **URLSession** — recommended high-level API for HTTP/HTTPS and custom TCP via `URLSessionStreamTask`

### Reducing Latency: HTTP/2
- HTTP/1.1 head-of-line blocking: each request must wait for the previous response
- HTTP/2 multiplexes multiple streams over a single connection — requests go out almost immediately
- HTTP/2 header compression reduces request overhead
- No client-side changes needed — enable HTTP/2 on servers and URLSession automatically uses it

### HTTP/2 Connection Coalescing (iOS 12 / macOS Mojave) [NEW]
- URLSession automatically reuses an existing HTTP/2 connection for requests to different hostnames if:
  1. The existing certificate covers the new hostname (e.g., wildcard `*.example.com`)
  2. The new hostname resolves to the same IP address
- Enabled automatically for all URLSession users — no API changes required

### Fewer URLSession Objects
- Each URLSession has its own connection pool; multiple sessions = no connection reuse
- URLSession objects are expensive to create and have non-trivial memory overhead
- Best practice: use one shared URLSession per configuration type

### Maximizing Throughput
- **HTTP Compression** (content-encoding): Gzip (widely supported, fast) and Brotli (best ratio for structured text/HTML; available since iOS 11) — enable on server
- **Cookie hygiene**: cookies are sent on every matching request; use narrow `domain`/`path` attributes; delete when no longer needed; prefer server-side state to minimize client cookies
- HTTP/2 header compression also reduces cookie overhead

### Increasing Responsiveness
- **URLSession QoS**: captures QoS of the queue on which `task.resume()` is called; delegate callbacks respect the same QoS; use `DispatchQueue` with `.background` QoS for non-time-critical fetches
- **`URLSessionConfiguration.networkServiceType`**: classifies traffic for system prioritization
  - New value: `.responsiveData` **[NEW]** — slightly above default; use for time-critical user-initiated network calls (e.g., checkout payment)
  - Maintained across hops on Cisco Fast Lane networks
- **`waitsForConnectivity`** (iOS 11): set on `URLSessionConfiguration`; task waits rather than failing immediately when there is no connectivity; replaces `SCNetworkReachability` preflight
- **`urlSession(_:taskIsWaitingForConnectivity:)`** delegate method — called when waiting; use to show offline UI

### Making Better Use of System Resources
- **Background URL Sessions**: upload/download tasks run out-of-process using system intelligence (battery, CPU, Wi-Fi) to schedule work; downloads continue when app is suspended
- **Cache control**: caching can cause severe flash storage degradation (gigabytes/day seen in practice)
  - Use `URLSession(_:willCacheResponse:completionHandler:)` delegate to selectively cache
  - Use `Cache-Control` server headers to mark non-cacheable responses
  - Do not cache unique/single-use content (e.g., user profile images that won't be revisited)

## APIs & Frameworks

**URLSession / Foundation Networking**
- `URLSession` — high-level networking API; recommended for all HTTP work
- `URLSessionStreamTask` — secure TCP connections for custom protocols
- `URLSessionConfiguration.networkServiceType` — `.default`, `.background`, `.video`, `.voice`, `.callSignaling`, `.responsiveData` **[NEW]**
- `URLSessionConfiguration.waitsForConnectivity` — task waits for connectivity instead of failing
- `urlSession(_:taskIsWaitingForConnectivity:)` — `URLSessionTaskDelegate` method; called when waiting
- `urlSession(_:willCacheResponse:completionHandler:)` — selective caching delegate
- Background `URLSession` (`URLSessionConfiguration.background(withIdentifier:)`)
- HTTP/2 Connection Coalescing **[NEW in iOS 12]** — automatic, no API changes

**Security**
- TLS 1.3 — opt-in flag available in iOS 12 seed; default later in 2018
- Certificate Transparency enforcement — later in 2018; no app changes, but CA must log certificates

**Network.framework (iOS 12 / macOS Mojave)** **[NEW]**
- Low-level user-space networking (see Session 715 for details)
- Alternative to BSD Sockets for custom TCP/UDP protocols

**Diagnostic Tools**
- Network Link Conditioner — simulate 2G, 3G, lossy networks during development
- Wireshark — packet-level network analysis
- tcptrace — visual TCP connection timeline graphs

## Code Highlights

Configuring QoS for a non-time-critical network task:
```swift
let backgroundQueue = DispatchQueue(label: "com.app.background", qos: .background)
backgroundQueue.async {
    let task = URLSession.shared.dataTask(with: url) { data, _, _ in /* ... */ }
    task.resume()  // URLSession captures .background QoS
}
```

Using `waitsForConnectivity` with an offline UI callback:
```swift
let config = URLSessionConfiguration.default
config.waitsForConnectivity = true
let session = URLSession(configuration: config, delegate: self, delegateQueue: nil)

func urlSession(_ session: URLSession, taskIsWaitingForConnectivity task: URLSessionTask) {
    showOfflineUI()
}
```

Marking a payment request as responsive data:
```swift
let config = URLSessionConfiguration.default
config.networkServiceType = .responsiveData
let session = URLSession(configuration: config)
```

## Takeaways
- Enabling HTTP/2 on your server is the single highest-impact change — URLSession automatically uses it with no client code changes, and HTTP/2 Connection Coalescing in iOS 12 further multiplies the benefit.
- Use a single shared `URLSession` per configuration; multiple sessions defeat connection reuse and waste memory.
- Replace `SCNetworkReachability` preflight checks with `waitsForConnectivity` — it correctly handles the race condition between checking and connecting.
- Test with Network Link Conditioner from day one; performance problems in 2G conditions are invisible on LTE until it is too late to fix them.

---
_Source: WWDC18 Session 714 page (abstract, chapter summaries, code samples, and resource links)._
