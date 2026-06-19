# Introducing Car Keys
**WWDC20 · Session 10006** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10006/)

_Platforms:_ iOS 14, watchOS 7

## Overview
Car Keys is a new feature in iOS 14 that allows users to store digital car keys in Apple Wallet and use them to lock, unlock, and start their vehicles using iPhone or Apple Watch via NFC. The system is designed for automakers and operates fully offline for core transactions, with server connectivity only required for remote key management operations like sharing and revocation.

The architecture is security-first: cryptographic key pairs are generated and permanently stored in the Secure Element on the device, meaning the private key never leaves the hardware. All protocols use standard AES and elliptic-curve algorithms, and the system is built on Public Key Infrastructure (PKI). Apple's Enhanced Contactless Protocol (ECP) enables automatic, seamless reader selection without user input.

Digital key sharing is integrated with Messages, allowing owners to share keys with friends or family at configurable access levels (e.g., "Unlock and Drive" vs. speed-restricted "Access and Drive Restricted"). Keys can be managed remotely, revoked instantly even when the device is offline, and survive device upgrades gracefully. Future support for Ultra Wideband (UWB) will enable passive entry without removing the phone from a pocket.

## Key Topics

### Owner Pairing
- Establishes a secure association between the owner's iPhone and the car via NFC
- Requires the user to prove vehicle ownership using automaker-defined criteria
- Initiated via an automaker app or a welcome email; the user places iPhone on the NFC reader in the dashboard
- Uses an elliptic-curve PAKE (Password Authenticated Key Exchange) protocol for secure channel creation
- Car key appears in Apple Wallet upon successful pairing

### Transactions (Unlock / Engine Start)
- Cars must provide at least one NFC reader in the door handle and one in the dashboard
- Fast transaction: used for door unlock; uses pre-computed symmetric keys for speed
- Standard transaction: used for engine start; includes mutual authentication and key exchange for subsequent fast transactions
- Express Mode: works without Face ID or passcode; can be disabled for stronger security
- All transactions are fully offline; Apple receives no transaction telemetry

### Key Sharing
- Owner initiates sharing via Messages; no automaker server required for the invite flow
- Friend device creates a new key pair; public key certificate chain is sent back to owner via Apple Identity Service (IDS)
- Owner issues a signed confirmation attestation; friend presents it to the car on first use
- Apple and the automaker do not see sharing details

### Key Management
- Owners can revoke shared keys from their device or the car's UI
- Revoked keys stop working immediately, even if the revoked device is offline
- iCloud Lost Mode locks the car key applet on the Secure Element
- Device upgrades: pair new iPhone with car; old key is removed, shared keys remain valid
- Remote deletion supported via iCloud

### Security Architecture
- Elliptic-curve key pair generated in the Secure Element; private key never exported
- Public key exported in an X.509 certificate for authenticity verification
- Single Secure Element applet hosts all car keys for memory efficiency
- Instance CA per automaker: ties device identity to automaker without exposing it to Apple
- Forward secrecy in transactions; no device identifiers sent in fast transactions

### Radio Technologies
- NFC (ISO 7816): standard reader in door handle and dashboard
- Apple Enhanced Contactless Protocol (ECP) **[NEW]**: auto-selects the correct Wallet pass before transaction begins; works for both car and payment readers
- Ultra Wideband (UWB): upcoming passive entry support via secure ranging (spec in draft form at time of WWDC20)

### Server Integration (Automaker)
- Automaker server connects to Apple's backend per environment (test/production)
- Apple provides car key root certificate for cross-signing; automaker returns external CA, root, and encryption/verification certificates
- Server interfaces required: register key, remotely revoke keys, send device notifications
- Wallet pass artwork/template provided via Apple portal; pass creation is automatic (no code required from automaker)
- Automaker apps need entitlement; APIs available to automakers only via PassKit

## APIs & Frameworks

- **PassKit**
  - Automaker app APIs for car key management (automaker-entitlement required) **[NEW]**
  - Wallet pass integration for car key display (automated pass creation via Apple portal)
- **NFC / Core NFC**
  - NFC reader communication (door handle and dashboard readers)
  - Apple Enhanced Contactless Protocol (ECP) **[NEW]** — automatic reader/pass selection
- **Secure Element**
  - Car key applet on Secure Element — hosts all car keys; implements transactions and mailboxes
  - Instance CA — per-automaker certificate authority on the Secure Element
- **Cryptographic primitives used (standard algorithms, not Apple-specific APIs)**
  - ECDH (elliptic-curve Diffie-Hellman) key exchange
  - ECDSA (elliptic-curve digital signatures) for authentication
  - AES encryption for transaction data
  - Elliptic-curve PAKE protocol for owner pairing
  - X.509 certificate chains for key attestation and trust
- **Apple Identity Service (IDS)**
  - Used for encrypted peer-to-peer key sharing messages between devices (not exposed as a public API)
- **iCloud**
  - Remote key deletion and Lost Mode integration

## Code Highlights

No developer-facing Swift/Objective-C code samples were shown in this session. The session focused on the architectural and cryptographic protocols intended for automaker integration teams. Automaker app developers should refer to PassKit documentation and the Apple MFi program materials for implementation specifics.

Enrollment path for automakers:
1. Join the Car Connectivity Consortium (CCC) and obtain the Digital Key Specification (v2.0 for NFC, v3.0 draft for UWB)
2. Enroll in the Apple MFi program for hardware specs, performance targets, and server integration details
3. Cross-sign Apple's car key root certificate; submit automaker certificates to Apple
4. Implement required server interfaces (key registration, revocation, device notifications)
5. Submit automaker app with PassKit car key entitlement

## Takeaways
- Car Keys is a system-level feature requiring automaker hardware and server integration, not a general third-party SDK; access requires the Apple MFi program and CCC membership.
- All transaction cryptography runs offline on the Secure Element, and Apple never sees when or where a user drives.
- Apple's Enhanced Contactless Protocol (ECP) enables seamless reader disambiguation on the device side — automakers must implement the ECP reader-side protocol in their NFC hardware.
- UWB-based passive entry (no need to remove phone from pocket) is on the roadmap as CCC Digital Key Specification v3.0.

---
_Source: WWDC20 Session 10006 page (abstract and transcript)._
