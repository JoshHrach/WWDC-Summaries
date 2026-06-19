# Developing Wireless CarPlay Systems
**WWDC17 · Session 717** · [Watch](https://developer.apple.com/videos/play/wwdc2017/717/)

_Platforms:_ iOS 11, CarPlay (automotive head unit firmware)

## Overview
This session targets automotive head unit developers building wireless CarPlay systems. It covers hardware requirements, the two pairing flows (over USB and over Bluetooth), the reconnection logic for every common usage scenario, and architectural considerations for Wi-Fi frequency bands, coexistence with other wireless technologies, and Internet data connectivity.

Wireless CarPlay uses Bluetooth for device discovery and reconnection, and a Wi-Fi access point (preferably 802.11ac on 5 GHz) for all audio, video, and iAP2 data transfer. The session presents a precise sequence of component initialization events, iAP2 message exchanges, and CarPlay Control API calls needed to establish and restore sessions correctly, and clarifies UX rules such as never automatically switching the active CarPlay device mid-session and not reconnecting wirelessly when the user simply unplugs from USB.

## Key Topics

- **Wireless CarPlay user experience** — same experience as wired; first-time pairing either by plugging in USB (simplest) or via Bluetooth; subsequent trips auto-connect without any user action; iPhone can stay in a bag or pocket.
- **Pairing by USB** — plug in triggers out-of-band BT pairing in the background; iOS shows "Enable Wireless CarPlay" prompt; head unit receives BT link key via iAP2 and stores device as a CarPlay BT device; no interruption to the active wired session.
- **Pairing by Bluetooth** — three UI approaches: (1) head unit tells user to find car name on iPhone; (2) head unit lists all BT discovered devices; (3) head unit shows only CarPlay-capable devices (using Apple CarPlay BT EIR); long-press Voice Control button initiates pairing from car side; both car and iPhone must be discoverable.
- **Wireless session establishment** — after BT pairing and iAP2 connect, iPhone requests Wi-Fi credentials; head unit waits for user confirmation before providing credentials; iPhone joins the access point; Bonjour discovery; head unit initiates session via CarPlay Control API; Bluetooth connections to that device are then disconnected.
- **Reconnection scenarios** — five scenarios detailed: phone in pocket (wireless); plug-in after drive starts (stays wireless); phone left plugged in (USB typically wins on restart); plug-in immediately on entry (first available transport wins); multiple devices (last connected reconnects automatically; user switches via native UI).
- **Reconnection logic** — wait for BT + AP operational; verify no active session; connect iAP2 over BT; wait for wireless CarPlay enabled notification; wait for Bonjour discovery; issue connect command; iPhone chooses wired or wireless based on availability.
- **Session disconnect handling** — explicit user unplug or native UI disconnect: no reconnect needed. Phone out of range or Wi-Fi lost: fast reconnect via CarPlay Control API, then fall back to BT reconnect.
- **Multi-device management** — device selector in native UI shows all paired/plugged CarPlay devices; use CarPlay logo to indicate active device; never indicate wired vs. wireless to user; do not auto-switch mid-session.
- **Notification rules** — show subtle notification only for first iPhone connection if CarPlay won't be immediately visible; show notification when second iPhone plugs in via USB; no notifications for AP join/leave or BT reconnect events.

## APIs & Frameworks

This session targets head unit firmware developers, not iOS app developers. Platform-level components and protocols referenced:

**Hardware Requirements**
- Bluetooth Core Specification — service discovery, iAP2 protocols, CarPlay EIR advertisement
- Wi-Fi access point — Wi-Fi Alliance certification required; 802.11ac on 5 GHz recommended; must support Apple Device Information Element and Interworking Information Element
- GNSS receiver + vehicle speed sensor — required for dead-reckoning location (critical for wireless as phone may be out of sight)

**Protocols and APIs (head unit firmware)**
- `iAP2` — Apple Accessory Protocol 2; carries Bluetooth link keys, device transport identifiers, CarPlay enable/disable notifications, Wi-Fi credentials, and Internet connectivity state
- `CarPlay Control API` — head unit API to initiate and manage CarPlay sessions
- `Bonjour` (mDNS/DNS-SD) — used by CarPlay for device discovery once iPhone is on the access point
- Apple CarPlay Bluetooth EIR — Extended Inquiry Response field advertising CarPlay support; used to filter CarPlay-capable devices in pairing UI

**Wireless Architecture Guidelines**
- 5 GHz band strongly recommended; 2.4 GHz supported but requires disabling all Bluetooth while CarPlay session is active
- Multiple access points must use different channels if operating in the same band; must offer identical services if sharing SSID/password
- Hidden networks not recommended for CarPlay
- Internet connectivity state communicated via Apple Device IE and Networking IE notifications; transient cell coverage loss does not need to be reported

**No iOS app-level APIs** are covered in this session. The audience is automotive system engineers.

## Code Highlights

No code samples. The session focuses on firmware-level protocol sequences and hardware architecture.

Key initialization sequence for wireless CarPlay:
1. On ignition: initialize USB (role swap + NCM), BT, Wi-Fi, networking, Bonjour, CarPlay communication plugin.
2. Check last connected device; verify CarPlay is still enabled.
3. Connect iAP2 over BT; wait for wireless CarPlay enabled notification.
4. Wait for Bonjour discovery; call CarPlay Control API connect.
5. iPhone selects wired or wireless transport; session starts; restore last user mode (last screen, last audio source).

## Takeaways

- Always support USB-initiated out-of-band pairing as the primary pairing method — it requires the least user effort and is the most reliable path to first wireless session.
- Use 5 GHz Wi-Fi exclusively where possible; 2.4 GHz requires disabling all Bluetooth during the CarPlay session, which breaks other BT profiles.
- Never automatically switch the active CarPlay device mid-session — changing the active device must be an explicit user action via the native UI.
- Reconnect behavior must respect the user's intention: a USB unplug or native UI disconnect requires no reconnect; a network-level loss of connectivity requires a fast reconnect attempt.

---
_Source: WWDC17 Session 717 page (abstract, transcript, and resource links)._
