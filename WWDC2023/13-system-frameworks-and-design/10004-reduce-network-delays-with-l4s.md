# Reduce network delays with L4S
**WWDC23 · Session 10004** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10004/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
L4S (Low Latency, Low Loss, Scalable throughput) is an emerging IETF-standard networking technology that dramatically reduces queue buildup at network bottlenecks, resulting in lower latency, near-zero packet loss, and higher sustained throughput — all simultaneously. This session explains how L4S works, how Apple is rolling it out in iOS 17 and macOS Sonoma, and what developers need to do to ensure their apps benefit from it.

In a demo comparing video calls over a congested network, L4S reduced worst-case round-trip time by 50% (from 45 ms to under 25 ms), reduced packet loss from over 40% to near zero, and virtually eliminated video stalls. Apps using URLSession or Network framework with HTTP/3 or QUIC get L4S automatically with no code changes. HTTP/2 and TCP downloads also gain built-in L4S support in iOS 17 and macOS Sonoma.

## Key Topics

### How L4S Works
Traditional networks build queues at the bottleneck hop until buffer space is exhausted and packets are dropped — damaging both latency and throughput. L4S uses **Explicit Congestion Notification (ECN)** to signal congestion early, before queues grow large:

1. **Sender** marks packets with ECN bits in the IP header, signaling L4S capability.
2. **L4S-capable bottleneck** applies Scalable Queue Management (SQM). When a queue begins forming, it sets a congestion label on ECN-marked packets instead of dropping them.
3. **Receiver** counts congestion-marked packets and echoes the count back to the sender.
4. **Sender** uses this feedback to reduce its sending rate proportionally, preventing large queues from forming.

This three-party collaboration (sender, bottleneck, receiver) keeps queues nearly empty while maximizing throughput.

### App Preparation
- **HTTP/3 or QUIC via URLSession / Network framework:** L4S is automatic — no code changes required.
- **HTTP/2 or TCP:** Built-in L4S support for downloads in iOS 17 / macOS Sonoma — no code changes required.
- **Custom protocols:** Must implement scalable congestion control, ECN validation (detect ECN bleaching), and a mechanism to echo ECN feedback from receiver to sender. Use `ECNProperty` on packet metadata in Network framework, or `setsockopt` / `sendmsg` / `recvmsg` with socket options for raw sockets.

### Server Setup
- **QUIC servers:** Enable ECN and L4S in the QUIC implementation. Ask server/CDN provider for support.
- **TCP servers on Linux:** Follow the instructions at the referenced GitHub page to patch the TCP stack.
- Servers that don't yet fully support L4S can still benefit by enabling ECN receive support.

### L4S Network Requirements
1. Network must not block or zero ECN bits (no "ECN bleaching").
2. Bottleneck hop must support L4S Scalable Queue Management.

### Testing with macOS Internet Sharing
macOS Sonoma's Internet Sharing supports L4S queue management. Using `ifconfig` to throttle the shared interface makes the Mac the bottleneck; traffic through it receives L4S treatment.

### Enabling L4S for Testing
- **iOS 17:** Developer Settings > L4S toggle.
- **macOS Sonoma:** `sudo defaults write -g network_enable_l4s -bool true`.
- L4S rolls out progressively to a random subset of users in production; Developer Settings forces it on.

## APIs & Frameworks

- **Network framework**
  - `ECNProperty` on packet metadata — read/write ECN bits for custom protocols **[NEW support]**
  - Custom protocol support for L4S: scalable congestion control, ECN validation, ECN echo
- **URLSession** — automatic L4S support for HTTP/3, QUIC, and HTTP/2 downloads **[NEW]** (iOS 17, macOS Sonoma)
- **QUIC / HTTP/3** — L4S built in via Network framework **[NEW]**
- **TCP** — L4S built in for downloads in iOS 17 / macOS Sonoma **[NEW]**
- **Explicit Congestion Notification (ECN)** — IETF standard; IP header ECN bits used to signal congestion without packet drop
  - ECN bit values: Not-ECT, ECT(0), ECT(1), CE (Congestion Experienced)
- **Socket-level ECN** — `setsockopt`, `sendmsg`, `recvmsg` system calls for ECN flags in custom socket-based protocols
- **RFC 9330** — IETF specification for L4S architecture and requirements
- **Scalable Queue Management (SQM)** — bottleneck-side algorithm for L4S
- **Internet Sharing** (macOS Sonoma) — now supports L4S queue management **[NEW]**
- **Developer Settings** (iOS 17) — L4S enable toggle for testing **[NEW]**
- `ifconfig` — `tbr` (token bucket rate) option to throttle interface bandwidth for test network setup
- `defaults write -g network_enable_l4s -bool true` — macOS CLI command to enable L4S for testing

## Code Highlights

No Swift/ObjC API changes needed for URLSession or Network framework HTTP apps. For testing:

Throttle Internet Sharing interface to create a bottleneck (replace `en1` with your interface):
```bash
sudo ifconfig en1 tbr 10Mbps
```

Enable L4S on macOS for testing:
```bash
sudo defaults write -g network_enable_l4s -bool true
```

Reverse throttling (set bandwidth to 0 = unlimited):
```bash
sudo ifconfig en1 tbr 0
```

## Takeaways

- Apps using URLSession or Network framework with HTTP/3, QUIC, or HTTP/2/TCP downloads get L4S improvements for free in iOS 17 / macOS Sonoma — no code changes required.
- L4S reduces worst-case latency by ~50%, near-eliminates packet loss, and eliminates video stalls on congested networks, making it transformative for real-time media, gaming, and collaboration apps.
- Test with Developer Settings > L4S enabled on iOS 17, and use macOS Sonoma Internet Sharing (with bandwidth throttling via `ifconfig tbr`) to create a local L4S test network.
- Work with server/CDN providers now to enable ECN and L4S on QUIC and TCP stacks; even enabling ECN receive on servers that don't fully support L4S provides some benefit.

---
_Source: WWDC23 Session 10004 page (abstract, chapter summaries, code samples, and resource links)._
