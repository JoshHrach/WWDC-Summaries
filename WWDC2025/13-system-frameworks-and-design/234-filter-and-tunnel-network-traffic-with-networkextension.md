# Filter and tunnel network traffic with NetworkExtension

**Session ID:** 234  
**WWDC Year:** 2025  
**Folder:** `13-system-frameworks-and-design`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/234/

---

## Overview

This session covers new capabilities in the NetworkExtension framework for building privacy-preserving network filtering solutions. The main focus is a new URL filtering architecture that uses Private Information Retrieval (PIR) backed by Bloom filters to let a network extension check URLs against a blocklist without the filter server learning which URLs a device queries. The session also covers Oblivious HTTP (OHTTP) for proxying filter requests, Privacy Pass tokens for anonymous authentication, and updates to the DNS proxy and content filter provider APIs.

---

## Key Topics

- URL content filtering using PIR (Private Information Retrieval) with Bloom filters
- Privacy Pass tokens for authenticating filter queries without device identity
- Oblivious HTTP (OHTTP) as a transport to hide URL queries from the filter server
- New `NEFilterDataProvider` APIs for URL-based decisions
- Updating Bloom filter databases on-device without re-downloading the full list
- `NEDNSProxyProvider` updates for handling DNS-over-HTTPS and DNS-over-TLS
- System Extension vs. App Extension models for macOS and iOS respectively

---

## APIs & Frameworks

- **NetworkExtension** framework – Umbrella framework for VPN, content filtering, DNS proxy, and relay extensions.
- **`NEFilterDataProvider`** – Base class for content filter network extensions; subclass to intercept and allow/deny/redirect network flows.
- **`NEFilterNewFlowVerdict`** – Returned from `handleNewFlow(_:)` to allow, deny, or redirect traffic. New in iOS 26: `.urlBlocked` verdict for URL-specific denials.
- **PIR (Private Information Retrieval)** – **[NEW]** Cryptographic protocol (using homomorphic encryption / Swift Homomorphic Encryption library) allowing on-device lookup of a server-side URL blocklist without the server knowing the queried URL.
- **Bloom filter database** – **[NEW]** Compact probabilistic data structure downloaded to device for fast local URL membership testing before triggering a PIR query; reduces server round-trips.
- **`NEFilterURLInfo`** – **[NEW]** Struct delivered to `NEFilterDataProvider` containing the full URL, host, path, and query of a new HTTP/HTTPS flow.
- **`NEFilterProvider.urlDatabase`** – **[NEW]** Property for associating a local Bloom filter database URL with the filter extension; system handles incremental updates.
- **Privacy Pass** (`PrivacyPass` module) – **[NEW]** Standard (RFC 9576) anonymous authentication tokens; used to authenticate PIR queries to the filter server without a user identifier.
- **Oblivious HTTP (OHTTP)** – **[NEW]** HTTP transport protocol that routes requests through a relay, hiding the client's IP from the filter server; integrated with `NEFilterDataProvider` via `NEFilterOHTTPConfiguration`.
- **`NEFilterOHTTPConfiguration`** – **[NEW]** Configuration type specifying the OHTTP relay URL and the filter server's public key for HPKE encryption.
- **`NEDNSProxyProvider`** – Existing DNS proxy class; updated in iOS 26 / macOS 26 to expose `resolveHostname(_:completionHandler:)` for custom DNS resolution within filter logic.
- **`NEAppProxyProvider`** – App-level proxy provider; no new APIs this year but works alongside the new filter stack.

---

## Code Highlights

Handling a new URL flow with a local Bloom filter check:
```swift
class MyFilterProvider: NEFilterDataProvider {
    override func handleNewFlow(_ flow: NEFilterFlow) -> NEFilterNewFlowVerdict {
        guard let urlFlow = flow as? NEFilterBrowserFlow,
              let url = urlFlow.request?.url else {
            return .allow()
        }
        // Fast local check against Bloom filter
        if bloomFilter.mightContain(url.host ?? "") {
            // Trigger async PIR query for definitive answer
            Task { await checkWithPIR(url: url, flow: flow) }
            return .pause()   // pause flow until PIR resolves
        }
        return .allow()
    }
}
```

Configuring OHTTP for PIR queries:
```swift
let ohttpConfig = NEFilterOHTTPConfiguration(
    relayURL: URL(string: "https://relay.example.com/ohttp")!,
    targetPublicKey: serverPublicKeyData
)
filterManager.providerConfiguration?.ohttpConfiguration = ohttpConfig
```

---

## Takeaways

- The new PIR + Bloom filter architecture lets URL-based content filters query a blocklist without exposing the user's browsing history to the filter server.
- Bloom filters provide fast local pre-filtering; PIR is used only when the Bloom filter signals a possible match, minimizing server queries.
- Privacy Pass tokens authenticate filter queries anonymously, fulfilling anti-abuse requirements without user identity.
- OHTTP hides the device's IP address from the filter server, completing the privacy chain: content is unknown (PIR), identity is unknown (Privacy Pass), and IP is hidden (OHTTP).
- These privacy-preserving technologies are already used in Apple's own Safe Browsing and Communication Safety features.
- Developers building parental controls, enterprise URL filters, or security products should adopt this architecture to avoid becoming a surveillance surface.
