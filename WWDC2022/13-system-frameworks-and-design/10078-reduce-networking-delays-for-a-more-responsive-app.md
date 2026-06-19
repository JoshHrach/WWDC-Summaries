# Reduce networking delays for a more responsive app
**WWDC22 · Session 10078** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10078/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This session from Apple's networking team explains the root cause of perceived app slowness — queuing delay at the network's slowest link, known as "bufferbloat" — and provides a three-layer framework for reducing it: client-side protocol adoption, server-side buffer tuning, and network-level L4S (Low Latency, Low Loss, Scalable Throughput) adoption. Studies show that increasing bandwidth provides diminishing returns beyond about 4 Mbps, while every 20 ms reduction in latency yields a linear improvement in page load time across all network activity.

Key developer actions are all within reach using URLSession and Network.framework with no custom protocol work required.

## Key Topics

### Why Latency Beats Bandwidth
- Total time-to-first-response = (round-trip time × number of round trips). TLS 1.2 over TCP requires 4 round trips before the first response packet arrives.
- Bufferbloat: the slowest network link typically has a large queue. New packets from your app wait behind stale queued packets, inflating every round-trip.
- Bandwidth increase past ~4 Mbps produces almost no improvement in page load time; every 20 ms latency reduction yields linear improvement.
- These results apply to all network activity — API calls, image loads, streaming video seeks, real-time communications.

### Client-Side Optimizations

**Adopt modern protocols (no code changes needed beyond using URLSession/Network.framework)**
- HTTP/3 over QUIC: median request completion is ~55% that of HTTP/1. Enable on server; URLSession and Network.framework use it automatically.
- TLS 1.3: reduces handshake round trips compared to TLS 1.2. Enabled automatically.
- IPv6: better routing efficiency; no app changes needed.

**Connection migration (handover)**
- Eliminates stalls when the device switches between Wi-Fi and cellular.
- `URLSessionConfiguration.multipathServiceType = .handover`
- `NWParameters.multipathServiceType = .handover`

**QUIC datagrams (NEW in iOS 16 / macOS Ventura)**
- For app-defined protocols over UDP: QUIC datagrams react to network congestion (unlike plain UDP), keeping RTT low and reducing packet loss.
- Set `NWProtocolQUIC.Options.isDatagram = true` and specify `maxDatagramFrameSize`.
- Only one datagram flow per QUIC connection.

### Server-Side Buffer Tuning
- Large server-side buffers cause "head-of-line blocking": a new packet queues behind stale data in oversized buffers.
- Recommended buffer reductions for Apache Traffic Server:
  - **TCP**: set `proxy.config.net.sock_notsent_lowat` to 131072 (128 KB); enable `TCP_NODELAY`, `TCP_FASTOPEN`, `TCP_NOTSENT_LOWAT` via `sock_option_flag_in = 73`.
  - **TLS**: enable dynamic record sizes (`proxy.config.ssl.max_record_size = -1`).
  - **HTTP/2**: reduce `default_buffer_water_mark` to 32768 (32 KB) and `write_buffer_block_size` to 262144 (256 KB).
- Diagnose server buffering with the `networkQuality` tool: run against Apple's default server for baseline, then against your own server. A lower RPM (Round Trips Per Minute) score indicates server-side buffering.

### Measuring Responsiveness
- **`networkQuality` CLI** (macOS) — `networkQuality -s -C https://myserver.example.com/config`; configure your server as a target for accurate server-specific measurements.
- **Waveform Bufferbloat test** — `waveform.com/tools/bufferbloat`
- **Open-source Go implementation** — `github.com/network-quality/goresponsiveness`
- **Ookla Speedtest app** — now shows RTT; divide 60,000 by RTT ms to get RPM score.

### L4S (Low Latency, Low Loss, Scalable Throughput) — Beta in iOS 16 / macOS Ventura
- L4S is a network-level protocol that eliminates bufferbloat by using **explicit congestion notification** (ECN) signals instead of packet drops; the sender reduces rate immediately, maintaining very short queues.
- Demonstrated screen sharing improvement: remote machine time display lag dropped from several seconds to near-zero synchronization.
- Available as a developer beta: test with HTTP/3 or QUIC apps.
  - iOS 16: Developer Settings → Enable L4S.
  - macOS Ventura: `defaults write -g network_enable_l4s -bool true`
- Server-side: requires a QUIC implementation that supports accurate ECN and a scalable congestion control algorithm (e.g., Prague or CUBIC-ECN).
- L4S requires network infrastructure support; currently requires compatible ISP/router support for end-to-end benefit, but app-side compatibility testing can be done now.

## APIs & Frameworks

**Foundation / URLSession**
- `URLSessionConfiguration.multipathServiceType: URLSessionConfiguration.MultipathServiceType` — `.handover` enables connection migration (Wi-Fi ↔ cellular)

**Network.framework**
- `NWParameters.multipathServiceType: .handover` — connection migration for QUIC/TCP connections
- `NWProtocolQUIC.Options` **[QUIC datagrams NEW in iOS 16]**
  - `.isDatagram: Bool` — enable QUIC datagram mode
  - `.maxDatagramFrameSize: Int` — maximum datagram payload size (up to 65535)
- `NWConnection` — send/receive on datagram flow using QUIC connection

**System Tools**
- `networkQuality` — **[NEW in macOS Monterey]** measures RPM (round trips per minute), latency under load, buffer bloat; `-s` for sequential measurement, `-C <url>` for custom server target
- L4S developer flag (macOS Ventura): `defaults write -g network_enable_l4s -bool true` **[NEW beta]**

**Server Configurations (Apache Traffic Server)**
- `proxy.config.net.sock_notsent_lowat` — TCP not-sent low-water mark
- `proxy.config.net.sock_option_flag_in` — combined TCP socket option flags
- `proxy.config.ssl.max_record_size = -1` — dynamic TLS record sizes
- `proxy.config.http2.default_buffer_water_mark` / `.write_buffer_block_size` — HTTP/2 buffer tuning

## Code Highlights

Enable connection migration for URLSession:
```swift
let configuration = URLSessionConfiguration.default
configuration.multipathServiceType = .handover
```

Enable connection migration for QUIC (Network.framework):
```swift
let parameters = NWParameters.quic(alpn: ["myproto"])
parameters.multipathServiceType = .handover
```

Enable QUIC datagrams:
```swift
let options = NWProtocolQUIC.Options()
options.isDatagram = true
options.maxDatagramFrameSize = 65535
```

Measure server responsiveness from the command line:
```sh
networkQuality -s -C https://myserver.example.com/config
```

Enable L4S on macOS Ventura (developer testing):
```sh
defaults write -g network_enable_l4s -bool true
```

Recommended Apache Traffic Server configuration for low-latency serving:
```
CONFIG proxy.config.net.sock_notsent_lowat INT 131072
CONFIG proxy.config.net.sock_option_flag_in INT 73
CONFIG proxy.config.ssl.max_record_size INT -1
CONFIG proxy.config.http2.default_buffer_water_mark INT 32768
CONFIG proxy.config.http2.write_buffer_block_size   INT 262144
```

## Takeaways
- Bufferbloat — not insufficient bandwidth — is the primary cause of poor perceived app responsiveness; upgrading bandwidth beyond ~4 Mbps rarely improves load times, but reducing latency always does.
- Adopt HTTP/3 on your server: URLSession and Network.framework use it automatically, and measured median request times are roughly half those of HTTP/1.
- Set `multipathServiceType = .handover` to eliminate network stalls when users switch between Wi-Fi and cellular.
- Reduce server-side TCP/TLS/HTTP buffer sizes to near the BDP (bandwidth-delay product) of your connections to eliminate head-of-line blocking from stale queued data.
- Test your HTTP/3 or QUIC app with L4S enabled in Developer Settings today; provide feedback on any compatibility issues before L4S-capable networks are widely deployed.

---
_Source: WWDC22 Session 10078 page (transcript, code samples, and resource links)._
