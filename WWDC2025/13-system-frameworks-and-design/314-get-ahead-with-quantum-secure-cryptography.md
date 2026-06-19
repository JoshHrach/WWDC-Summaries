# Get ahead with quantum-secure cryptography

**Session ID:** 314  
**WWDC Year:** 2025  
**Folder:** `13-system-frameworks-and-design`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/314/

---

## Overview

This session explains why post-quantum cryptography (PQC) matters now — specifically the "harvest now, decrypt later" attack where adversaries collect encrypted traffic today to decrypt once quantum computers exist — and introduces the new quantum-secure algorithms added to Apple's CryptoKit framework in iOS 26, iPadOS 26, macOS 26, and watchOS 12. The session covers two NIST-standardized post-quantum algorithms: ML-KEM (key encapsulation, replacing RSA/ECDH for key exchange) and ML-DSA (digital signatures, replacing ECDSA). It also demonstrates hybrid construction patterns that combine classical and post-quantum algorithms for defense-in-depth.

---

## Key Topics

- The "harvest now, decrypt later" threat model and why PQC adoption is urgent
- NIST PQC standardization: ML-KEM (FIPS 203) and ML-DSA (FIPS 204)
- ML-KEM (Module Lattice Key Encapsulation Mechanism): replacing RSA/ECDH in key exchange
- ML-DSA (Module Lattice Digital Signature Algorithm): replacing ECDSA for signatures
- Hybrid key exchange: combining X25519 with ML-KEM-768
- Hybrid signatures: combining Ed25519 with ML-DSA-65
- Migrating existing protocols to hybrid or pure PQC schemes
- iMessage and Apple's own adoption of PQC (PQ3 protocol)

---

## APIs & Frameworks

- **CryptoKit** framework (`import CryptoKit`) – Apple's high-level cryptography framework; expanded with PQC algorithms in iOS 26 / macOS 26.
- **`MLKEM768`** – **[NEW]** (CryptoKit, iOS 26) ML-KEM key encapsulation mechanism with security parameter 768 (NIST FIPS 203). Provides `KeyPair`, `EncapsulationKey`, `DecapsulationKey`, and `Ciphertext` types.
- **`MLKEM1024`** – **[NEW]** ML-KEM variant with 1024-bit security parameter; higher security at larger key/ciphertext size.
- **`MLKEM768.KeyPair.generate()`** – Generates a new ML-KEM key pair on-device.
- **`MLKEM768.EncapsulationKey.encapsulate()`** – Returns `(ciphertext: Ciphertext, sharedSecret: SymmetricKey)`; sender calls this with the recipient's public key.
- **`MLKEM768.DecapsulationKey.decapsulate(ciphertext:)`** – Returns `SymmetricKey`; recipient calls with the sender's ciphertext to recover the shared secret.
- **`MLDSA65`** – **[NEW]** (CryptoKit, iOS 26) ML-DSA digital signature algorithm with security parameter 65 (NIST FIPS 204). Provides `PrivateKey`, `PublicKey`, `sign(_:)`, and `isValidSignature(_:for:)`.
- **`MLDSA87`** – **[NEW]** ML-DSA variant with parameter 87; higher security.
- **Hybrid KEM construction** – **[NEW]** Combine `Curve25519.KeyAgreement` (X25519) with `MLKEM768` by separately deriving two shared secrets and combining them via HKDF; secure if either algorithm remains unbroken.
- **`HKDF`** (existing CryptoKit) – Used to combine classical and PQC shared secrets in hybrid constructions.
- **`ChaChaPoly` / `AES.GCM`** (existing CryptoKit) – Symmetric encryption using the hybrid-derived key; unchanged.

---

## Code Highlights

ML-KEM key exchange (sender and recipient):
```swift
import CryptoKit

// Recipient generates a key pair and shares the encapsulation key
let recipientKeyPair = MLKEM768.KeyPair.generate()
let encapKey = recipientKeyPair.encapsulationKey

// Sender encapsulates a shared secret
let (ciphertext, senderSharedSecret) = try encapKey.encapsulate()

// Recipient decapsulates to recover the shared secret
let recipientSharedSecret = try recipientKeyPair.decapsulationKey.decapsulate(
    ciphertext: ciphertext
)
// senderSharedSecret == recipientSharedSecret — use as symmetric key
```

ML-DSA signing and verification:
```swift
import CryptoKit

let signingKey = MLDSA65.PrivateKey()
let message = Data("Hello, post-quantum world".utf8)
let signature = try signingKey.signature(for: message)

let isValid = signingKey.publicKey.isValidSignature(signature, for: message)
print(isValid) // true
```

Hybrid KEM combining X25519 and ML-KEM:
```swift
// Derive two independent shared secrets and combine via HKDF
let classicalSecret = try Curve25519.KeyAgreement.PrivateKey()
    .sharedSecretFromKeyAgreement(with: recipientClassicalPublicKey)
let (mlkemCiphertext, pqcSecret) = try mlkemEncapKey.encapsulate()

let combinedKey = HKDF<SHA512>.deriveKey(
    inputKeyMaterial: classicalSecret + pqcSecret,
    outputByteCount: 32
)
```

---

## Takeaways

- "Harvest now, decrypt later" is an active threat; attackers are collecting today's encrypted traffic expecting future quantum computers to decrypt it. Migration to PQC is time-sensitive for long-lived secrets.
- ML-KEM replaces RSA/ECDH for key exchange; ML-DSA replaces ECDSA for signatures. Both are NIST-standardized (FIPS 203/204) and available in CryptoKit starting in iOS/macOS 26.
- Hybrid constructions (X25519 + ML-KEM, Ed25519 + ML-DSA) are the recommended migration path: they remain secure against classical attacks if PQC is broken, and secure against quantum attacks if classical is broken.
- CryptoKit abstracts away the complex lattice math; the API surface is similar to existing key agreement and signing APIs.
- Apple already uses PQC in iMessage (PQ3 protocol) and is incrementally rolling it out across OS-level protocols.
- Start by auditing which parts of your protocol stack use asymmetric key exchange for long-lived secrets and prioritize those for PQC migration.
