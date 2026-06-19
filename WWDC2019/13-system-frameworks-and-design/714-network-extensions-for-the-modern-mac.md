# Network Extensions for the Modern Mac
**WWDC19 · Session 714** · [Watch](https://developer.apple.com/videos/play/wwdc2019/714/)

_Platforms:_ macOS Catalina 10.15

## Overview
This session marks the formal deprecation of Network Kernel Extensions (NKEs) on macOS and introduces the full set of user-space replacement APIs that ship in macOS Catalina. Five categories of apps that previously required NKEs are covered: content filter apps (personal firewalls, parental controls, network logging), transparent proxy apps, DNS proxy apps, VPN apps, and virtual machine networking apps. A sixth area covers new APIs for custom low-layer protocols (custom IP protocols and custom Ethernet link-layer protocols) via the Network framework.

The enabling technology across all categories is System Extensions — a new macOS-only extension type that runs independently of any logged-in user, is managed entirely by the OS (no custom installer/uninstaller), and is debuggable with standard Xcode/LLDB tooling. A live demo of a Simple Firewall app shows the complete flow: activating a System Extension via `OSSystemExtensionManager`, registering a content filter with `NEFilterManager`, and implementing `NEFilterDataProvider.handleNewFlow(_:)` with a paused-then-resumed verdict pattern.

## Key Topics
- **System Extensions** — user-space replacement for kernel extensions; packaged inside your app; OS manages lifecycle (start/stop); run without a logged-in user; debuggable with Xcode+LLDB; user prompted once via Security & Privacy preference pane **[NEW macOS Catalina]**
- **Network Kernel Extensions deprecated** — existing NKEs still load in Catalina but will be removed in a future release; migrate all NKE-based apps to the APIs in this session
- **Content filter APIs** — flow-layer (`NEFilterDataProvider`) and packet-layer (`NEFilterPacketProvider`) filtering; `NEFilterFlowObject`; `NEFilterRule` + `NENetworkRule` for traffic scoping; `NEFilterVerdict` including `.pause()` and `resumeFlow(_:with:)` for async decisions
- **Transparent proxy APIs** — `NEAppProxyProvider` subclass in a System Extension; `NETransparentProxyManager` for configuration; `NENetworkRule` specifies which flows to divert; full bidirectional flow handling
- **DNS proxy APIs** — `NEDNSProxyProvider` subclass; `NEDNSProxyManager`; all DNS queries diverted to the proxy for encryption / secure-channel forwarding
- **VPN enhancements** — `NETunnelProviderManager` / `NEPacketTunnelProvider` on macOS; new `includeAllNetworks` flag (traffic drops if VPN is down instead of leaking); `excludeLocalNetworks` for local resource access; Per-App VPN gains `mailDomains`, `calendarDomains`, `contactsDomains` arrays
- **Virtual machine networking** — `vmnet.framework` enhancements: IPv6 in Shared Mode, IP range specification, port forwarding rules; new Bridged Mode (VM appears as first-class LAN device)
- **Custom low-layer protocols** — `NWParameters(customIPProtocolNumber:)` for custom IP protocols; `NWEthernetChannel(on:etherType:)` for custom Ethernet ether types; both in Network framework **[NEW]**

## APIs & Frameworks

### System Extensions (NEW)
- `OSSystemExtensionManager` — `OSSystemExtensionManager.shared.submitRequest(_:delegate:)`
- `OSSystemExtensionRequest.activationRequest(forExtensionWithIdentifier:queue:)`
- `OSSystemExtensionRequestDelegate` — `request(_:didFinishWithResult:)`, `request(_:didFailWithError:)`
- `OSSystemExtensionReplacementAction` — `.cancel` / `.replace`

### NetworkExtension — Content Filter (NEW on macOS)
- `NEFilterManager` — `shared`; `loadFromPreferences(completionHandler:)`; `saveToPreferences(completionHandler:)`; `isEnabled: Bool`; `filterConfiguration: NEFilterProviderConfiguration`
- `NEFilterProviderConfiguration` — `filterSockets: Bool`; `filterPackets: Bool`; `filterDataProviderBundleIdentifier`
- `NEFilterDataProvider` — subclass in System Extension
  - `startFilter(completionHandler:)` — call `apply(_:completionHandler:)` with `NEFilterSettings`
  - `stopFilter(with:completionHandler:)`
  - `handleNewFlow(_ flow: NEFilterFlow) -> NEFilterVerdict` — return `.allow()`, `.drop()`, or `.pause()`
  - `resumeFlow(_:with:)` — resume a paused flow with a final verdict
- `NEFilterPacketProvider` — subclass for packet-layer filtering; receives `NEPacket` objects
- `NEFilterSettings` — `init(rules:defaultAction:)`; `defaultAction: NEFilterAction` (`.allow`, `.drop`, `.filterData`)
- `NEFilterRule` — `init(networkRule:action:)`
- `NENetworkRule` — `init(remoteNetwork:remotePrefix:localNetwork:localPrefix:protocol:direction:)`
  - `NENetworkRuleProtocol`: `.TCP`, `.UDP`, `.any`
  - `NETrafficDirection`: `.inbound`, `.outbound`, `.any`
- `NEFilterFlow` — `NEFilterSocketFlow` (TCP/UDP flows), `NEFilterBrowserFlow` (web flows)
- `NEFilterVerdict` — `.allow()`, `.drop()`, `.pause()`, `.need(rules:)`, `.remediateFlow(withUrlMapKey:issueButtonMapKey:)`

### NetworkExtension — Transparent Proxy (NEW on macOS)
- `NETransparentProxyManager` **[NEW]** — analogous to `NEFilterManager`
- `NEAppProxyProvider` — subclass; `handleNewFlow(_:)` receives `NEAppProxyFlow` objects
- `NEAppProxyTCPFlow` — `readData(completionHandler:)`, `write(_:completionHandler:)`
- `NEAppProxyUDPFlow` — `readDatagrams(completionHandler:)`, `writeDatagrams(_:sentBy:completionHandler:)`

### NetworkExtension — DNS Proxy (NEW on macOS)
- `NEDNSProxyManager` — `shared`; `loadFromPreferences(completionHandler:)`; `saveToPreferences(completionHandler:)`
- `NEDNSProxyProvider` — subclass in System Extension; `handleNewFlow(_:)` for DNS flows

### NetworkExtension — VPN
- `NETunnelProviderManager` — `loadAllFromPreferences(completionHandler:)`
- `NEPacketTunnelProvider` — `packetFlow: NEPacketTunnelFlow`; `setTunnelNetworkSettings(_:completionHandler:)`
- `NETunnelProviderProtocol` — `includeAllNetworks: Bool` **[NEW]** — drop traffic when VPN is unavailable
- `NETunnelProviderProtocol.excludeLocalNetworks: Bool` **[NEW]** — allow LAN access even when `includeAllNetworks` is true
- Per-App VPN — `NEAppRule` — `mailDomains: [String]` **[NEW]**, `calendarDomains: [String]` **[NEW]**, `contactsDomains: [String]` **[NEW]**

### vmnet.framework (Enhancements)
- Shared Mode: IPv6 support **[NEW]**, `vmnet_shared_interface_parameters_t` — IP range, start/end address **[NEW]**, port forwarding rules **[NEW]**
- Bridged Mode **[NEW]** — VM joins local network as if physically attached

### Network framework — Custom Low-Layer Protocols (NEW)
- `NWParameters(customIPProtocolNumber: UInt32)` **[NEW]** — create parameters for a custom IP protocol number (not TCP/UDP/ICMP)
- `NWConnection` — used identically to TCP/UDP connections after init with custom parameters
- `NWEthernetChannel(on: NWInterface, etherType: UInt16)` **[NEW]** — raw Ethernet channel for custom ether types (not 0x0800 IP or 0x86DD IPv6)
  - `start(queue:)`
  - `send(content:to:vlanTag:completion:)`
  - `receiveHandler: ((Data, NWEndpoint, UInt16) -> Void)?`
  - `stateUpdateHandler: ((NWEthernetChannel.State) -> Void)?`

## Code Highlights

```swift
// Activate a System Extension from the main app
let activationReq = OSSystemExtensionRequest.activationRequest(
    forExtensionWithIdentifier: "com.example.SimpleFirewall.Extension",
    queue: .main)
activationReq.delegate = self
OSSystemExtensionManager.shared.submitRequest(activationReq)

// After approval, enable the content filter
func request(_ request: OSSystemExtensionRequest,
             didFinishWithResult result: OSSystemExtensionRequest.Result) {
    let config = NEFilterProviderConfiguration()
    config.filterSockets = true
    config.filterPackets = false
    NEFilterManager.shared().providerConfiguration = config
    NEFilterManager.shared().isEnabled = true
    NEFilterManager.shared().saveToPreferences { _ in }
}
```

```swift
// NEFilterDataProvider — scoped rules + async verdict
override func startFilter(completionHandler: @escaping (Error?) -> Void) {
    let rule = NEFilterRule(
        networkRule: NENetworkRule(
            remoteNetwork: nil, remotePrefix: 0,
            localNetwork: NWHostEndpoint(hostname: "0.0.0.0", port: "8888"),
            localPrefix: 0, protocol: .TCP, direction: .inbound),
        action: .filterData)
    let settings = NEFilterSettings(rules: [rule], defaultAction: .allow)
    apply(settings) { completionHandler($0) }
}

override func handleNewFlow(_ flow: NEFilterFlow) -> NEFilterVerdict {
    // Notify UI asynchronously — pause the flow in the meantime
    notifyUI(about: flow)
    return .pause()
}

func userDecided(flow: NEFilterFlow, allow: Bool) {
    resumeFlow(flow, with: allow ? .allow() : .drop())
}
```

```swift
// Custom IP protocol connection
let params = NWParameters(customIPProtocolNumber: 253)  // IANA unassigned
let conn = NWConnection(host: "192.168.1.10", port: .any, using: params)
conn.start(queue: .main)

// Custom Ethernet channel
let channel = NWEthernetChannel(on: ethernetInterface, etherType: 0x88B5)
channel.stateUpdateHandler = { state in
    if state == .ready { /* start sending */ }
}
channel.receiveHandler = { data, sender, vlan in
    // handle incoming custom-ether-type frames
}
channel.start(queue: .main)
```

## Takeaways
- Network Kernel Extensions are deprecated in macOS Catalina and will be removed entirely in a future release — begin migration now by auditing every NKE your codebase ships.
- System Extensions are the foundation: they run without a logged-in user, are OS-managed, and are debuggable with standard Xcode/LLDB — adopt them first, then layer the appropriate Network Extension API (filter/proxy/DNS/VPN) on top.
- Use `NEFilterSettings` with targeted `NEFilterRule`+`NENetworkRule` objects to scope your content filter to only the traffic types you care about — the default `filterData` action on unmatched flows is expensive and unnecessary.
- The `includeAllNetworks` VPN flag is the correct way to implement a no-leak personal VPN — it drops traffic when the tunnel is unavailable rather than allowing it to bypass the VPN, which is the critical security guarantee users of such apps expect.
- `NWEthernetChannel` and `NWParameters(customIPProtocolNumber:)` fully replace raw socket + kext approaches for custom link-layer and IP-layer protocol implementations, with no kernel code required.

---
_Source: WWDC19 Session 714 page (full transcript, abstract, and resource links)._
