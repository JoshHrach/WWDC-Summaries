# Explore UWB-Based Car Keys
**WWDC21 · Session 10084** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10084/)

_Platforms:_ iOS 15, watchOS 8 (iPhone and Apple Watch with U1 chip)

## Overview
Building on the Car Keys foundation from WWDC20, this session introduces Ultra Wideband (UWB)-based passive entry for digital car keys. With UWB, the car can precisely locate an iPhone or Apple Watch and automatically unlock/lock the car as the user approaches or walks away, without requiring the user to take the device out of their pocket or bag.

The session is aimed at automotive OEMs and MFi partners implementing UWB car key support. It covers the underlying hardware/software stack, the secure-ranging protocol, remote keyless entry (RKE) controls, vehicle personalization, and practical engineering guidance for system integration (transceiver selection, time synchronization, localization algorithms).

## Key Topics

**Passive Entry with UWB**
The U1 chip in iPhone and Apple Watch enables precise spatial localization using Ultra Wideband. The car defines virtual zones around the vehicle; as a user (and their device) enters each zone, the car triggers the appropriate action (welcome lights, door unlock, engine start). The car uses multiple UWB and BLE transceivers for 360-degree coverage. UWB passive entry works even in iPhone power reserve mode.

**Secure Ranging Protocol**
UWB ranging uses a three-packet "poll-response-poll" exchange. Each packet carries a Scrambled Time Stamp (STS) — a cryptographically generated, time-bounded token that prevents replay and relay attacks. Session keys are derived per-connection inside the device's Secure Element and used to generate UWB ranging keys and random rotating identifiers for BLE and UWB to prevent device tracking.

**Remote Keyless Entry (RKE) Controls**
BLE-based remote actions (lock, unlock, preheat, horn, etc.) work outside UWB range and are triggered from Wallet. RKE uses a challenge/signature protocol: the device requests a challenge, generates a device signature, and the car verifies before executing the action. RKE is standardized at the Car Connectivity Consortium.

**Vehicle Personalization**
Because digital keys are strongly tied to individual users, cars can use UWB trajectory data (which key, which door) to automatically configure personal settings (seat position, temperature, etc.) without requiring the user to select a profile.

**System Integration Guidance**
Key engineering areas: transceiver selection (link budget, antenna diversity, 3D ToF accuracy), transceiver placement for 360-degree coverage, low-latency bus architecture (ECU-to-transceiver), time synchronization (tens-of-microsecond precision for minimal scan windows), transceiver synchronization (sharing timing across transceivers), and localization algorithm development (multi-lateration, trajectory tracking, inside/outside cabin detection).

## APIs & Frameworks

- **Car Keys / Wallet** (iOS 15, watchOS 8) — extended from WWDC20
- **U1 chip** with Ultra Wideband (UWB) — hardware requirement
- **Secure Element** — stores car keys, derives session-specific UWB ranging keys **[NEW]**
- **Bluetooth LE (BLE)** — communication channel for authentication, session management, RKE, timing anchor
- **UWB Secure Ranging Protocol** **[NEW]** — three-packet (poll-response-poll) exchange
  - Scrambled Time Stamp (STS) — cryptographically generated, time-bounded ranging token **[NEW]**
- **Car Connectivity Consortium (CCC)** — cross-platform specification body for digital car key standard
- **MFi Program** — required for automakers integrating with iPhone/Apple Watch car key
- Virtual zone model: welcome zone, unlock zone, lock zone **[NEW]**
- Power Reserve mode support for passive entry **[NEW]**
- Remote keyless entry (RKE) protocol:
  - Challenge request / device signature / car verification flow
  - Actions: lock, unlock, preheat cabin, horn honk, fuel/battery state query
  - Triggered via Wallet app

## Code Highlights

No Swift/Objective-C code samples provided in this session. This session targets automotive OEM engineers. Car key integration requires enrollment in the Apple MFi Program and adherence to the Car Connectivity Consortium specification.

Development workflow recommendation:
1. UWB interoperability testing (using Apple-provided tools, with step-by-step crypto enablement)
2. BLE layer integration (connection management, owner pairing)
3. Secure-ranging management
4. Remote keyless entry (RKE) actions

## Takeaways

- UWB passive entry enables hands-free car unlock/lock using precise spatial localization via the U1 chip — no user interaction required.
- The secure-ranging protocol with Scrambled Time Stamps (STS) provides strong protection against relay and replay attacks, a key weakness of traditional RF key fobs.
- System latency (BLE connection + authentication + UWB ranging setup) must complete before the user reaches the door handle — high-performance crypto processors and low-latency bus architectures are critical.
- Automakers must join the Car Connectivity Consortium and enroll in the Apple MFi Program to implement UWB car key support.

---
_Source: WWDC21 Session 10084 page (abstract, chapter summaries, code samples, and resource links)._
