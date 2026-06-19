# Reduce Network Delays for Your App
**WWDC21 · Session 10239** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10239/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
Network delays remain one of the largest sources of perceived sluggishness in apps. While CPU speed and bandwidth continue to improve, the speed of light is a hard ceiling on round-trip latency. This session explains the concept of bufferbloat — excessive packet buffering that inflates real-world latency by 30x or more over idle ping times — and how Apple measures network responsiveness using a new RPM (round-trips per minute) metric available in iOS 15 Developer settings and the `networkquality` command-line tool on macOS.

The session then walks through the modern networking protocols that each shave one or more round trips off connection establishment, potentially reducing time-to-first-byte from 2.4 seconds to 0.5 seconds on real-world congested networks. It also covers how to use traffic classification APIs so background prefetching does not starve foreground user activity.

## Key Topics

### Understanding Network Delays
- **Bufferbloat**: oversized network buffers don't improve throughput — they add delay. Idle ping may be 20 ms, but under load it can balloon to 600 ms.
- New **Responsiveness** developer setting in iOS 15: measures round-trips per minute (RPM) under working conditions, not idle.
- `networkquality` CLI tool on macOS provides the same measurement.
- App responsiveness is inversely proportional to the number of network round trips — reduce round trips for the biggest gains.

### Modern Protocols — Round-Trip Savings
- **HTTP/3 & QUIC** (enabled by default in iOS 15/macOS Monterey via URLSession): reduces to 2 round trips vs. 4 for TLS 1.2/TCP; QUIC early data can further reduce to 1.
- **TCP Fast Open**: sends idempotent app data with the TCP SYN, saving 1 round trip. Must only be used with idempotent (replay-safe) requests.
- **TLS 1.3** (default since iOS 13.4): removes 1 round trip vs. TLS 1.2; TLS 1.3 early data can send idempotent requests with the handshake.
- **Multipath TCP** interactive mode: preserves connection continuity across network switches (e.g., Wi-Fi to cellular), saving multiple handshake round trips.

### Traffic Classification
- Marking non-user-initiated transfers as `background` frees the network queue for foreground data.
- iOS 15/macOS Monterey: improved congestion control algorithms for background transfers; background traffic now completes in nearly the same time as default traffic.
- Background `URLSessionConfiguration` continues transfers when the app is suspended; set `isDiscretionary = true` for time-insensitive tasks.

### Server-Side Considerations
- Servers should use the TCP `not_sent_low_watermark` socket option to reduce send-buffer delay.
- All listed protocols require server-side support; verify provider readiness.

## APIs & Frameworks

- `URLSession` / `URLSessionConfiguration` — automatically uses HTTP/3 & QUIC in iOS 15+ **[NEW default]**
- `NWConnection` — Network.framework connection object
- `NWParameters` — connection parameter configuration
- `NWParameters.allowFastOpen` — enables TCP Fast Open **[NEW]**
- `NWConnection.send(content:completion:)` with `.idempotent` completion — sends data with TCP handshake **[NEW]**
- `NWParameters.tcp` options `.enableFastOpen` — TLS-over-TCP fast open variant
- `NWParameters.multipathServiceType` — set to `.interactive` for Multipath TCP low-latency mode
- `URLSessionConfiguration.multipathServiceType` — `.interactive` for Multipath TCP
- `URLRequest.networkServiceType` — set to `.background` for non-user-initiated foreground transfers
- `NWParameters.serviceClass` — set to `.background` for NWConnection background traffic
- `URLSessionConfiguration.background(withIdentifier:)` — background session for out-of-process transfers
- `URLSessionConfiguration.isDiscretionary` — allows system to pick optimal transfer time
- `NWParameters` (QUIC) — `NWParameters(quic:)` with ALPN configuration for custom QUIC usage
- `connectx()` BSD socket function with `CONNECT_DATA_IDEMPOTENT | CONNECT_RESUME_ON_READ_WRITE` flags — TCP Fast Open via sockets
- Network Link Conditioner (Developer tool) — simulates real-world network conditions for testing

## Code Highlights

TCP Fast Open via Network.framework:
```swift
parameters.allowFastOpen = true
let connection = NWConnection(to: endpoint, using: parameters)
connection.send(content: initialData, completion: .idempotent)
connection.start(queue: myQueue)
```

Multipath TCP for low-latency interactive traffic:
```swift
let configuration = URLSessionConfiguration.default
configuration.multipathServiceType = .interactive
```

Marking prefetch requests as background (app in foreground):
```swift
var request = URLRequest(url: myURL)
request.networkServiceType = .background
```

Background URLSession for out-of-process transfers:
```swift
let config = URLSessionConfiguration.background(withIdentifier: "MySession")
config.isDiscretionary = true
let session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
```

## Takeaways
- The biggest wins come from reducing the number of round trips, not raw throughput; adopt HTTP/3, TLS 1.3, and TCP Fast Open for server-backed apps.
- iOS 15 enables HTTP/3 and QUIC by default for URLSession — existing apps benefit automatically if their servers support it.
- Always classify non-user-initiated transfers as `.background` to protect the foreground experience, especially on congested shared networks.
- Test with Network Link Conditioner under realistic congested conditions — apps that look fine on fast Wi-Fi can feel broken at 600 ms round-trip latency.

---
_Source: WWDC21 Session 10239 page (abstract, chapter summaries, code samples, and resource links)._
