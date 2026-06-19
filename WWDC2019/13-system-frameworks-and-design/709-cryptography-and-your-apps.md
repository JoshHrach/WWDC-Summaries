# Cryptography and Your Apps
**WWDC19 · Session 709** · [Watch](https://developer.apple.com/videos/play/wwdc2019/709/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session has two parts. The first surveys the system-level security features Apple provides that developers should use before reaching for custom cryptography: Data Protection file attributes, Keychain, LocalAuthentication, CloudKit private database encryption, TLS via Network framework and URLSession, and SecTrust certificate validation. The second and larger part introduces Apple CryptoKit — a new, Swift-first cryptographic framework that provides safe, high-level access to peer-reviewed cryptographic primitives backed by Apple's `corecrypto` library.

The recurring theme is that "don't roll your own crypto" applies at multiple levels. Developers should use system frameworks for common scenarios, and when custom cryptography is necessary, CryptoKit's Swift API design makes it hard to misuse primitives while also providing high performance through hand-tuned assembly in `corecrypto`.

## Key Topics

- **System security features (use these first):** Complete file protection (`NSDataWritingAtomic + .completeFileProtection`), Keychain with iCloud Keychain sync, LocalAuthentication for biometric/passcode gating, CloudKit private database for encrypted cross-device data, TLS 1.3 via Network framework and URLSession (with App Transport Security), SecTrust for certificate validation.
- **Apple Watch authentication on macOS** — Users can now authenticate with a double-click on Apple Watch on macOS; two new LocalAuthentication policies: biometrics-or-Watch, and Watch-only. **[NEW]**
- **SecTrust new asynchronous function** — A single new function in SecTrust combines async validation with rich error information for debugging and application-level error handling. **[NEW]**
- **Apple CryptoKit** — A new Swift framework providing strongly typed, memory-safe, hard-to-misuse cryptographic APIs. **[NEW]**
- **Hash functions** — `SHA256`, `SHA384`, `SHA512`; one-shot and incremental (update/finalize) APIs; collision-resistant digests.
- **Authenticated encryption** — `AES.GCM` (and `ChaChaPoly`); sealed box concept combining nonce + ciphertext + authentication tag; prevents padding oracle attacks by combining auth and encryption in one API.
- **Signatures** — `P256`, `P384`, `P521` (ECDSA); `Curve25519` (EdDSA); sign with private key, verify with public key.
- **Key agreement** — ECDH key agreement via `P256.KeyAgreement`, `P384.KeyAgreement`, `P521.KeyAgreement`, `Curve25519.KeyAgreement`.
- **Message authentication codes** — `HMAC<SHA256>`, `HMAC<SHA384>`, `HMAC<SHA512>`; constant-time equality via `==`.
- **Secure Enclave integration** — `SecureEnclave.P256.Signing.PrivateKey`; access control policies (device unlock, biometric presence); LocalAuthentication context for user-facing reason string.
- **Insecure module** — `Insecure.MD5`, `Insecure.SHA1` for legacy interoperability.
- **Performance** — Built on `corecrypto` with hand-tuned assembly per microarchitecture; FIPS validated; side-channel resistant; secret values auto-zeroized on dealloc via ARC.

## APIs & Frameworks

### Apple CryptoKit **[NEW]**
- `SymmetricKey(size:)` — generate a cryptographically secure symmetric key; auto-zeroized on release
- `SHA256`, `SHA384`, `SHA512` — hash functions conforming to `HashFunction` protocol
- `SHA256.hash(data:)` — one-shot hashing
- `SHA256()`, `hasher.update(data:)`, `hasher.finalize()` — incremental hashing
- `AES.GCM.seal(_:using:nonce:)` — authenticated encryption **[NEW]**
- `AES.GCM.open(_:using:)` — authenticated decryption **[NEW]**
- `AES.GCM.SealedBox` — container for nonce + ciphertext + tag; init with `combined:` for wire format
- `ChaChaPoly.seal(_:using:nonce:)` / `ChaChaPoly.open(_:using:)` **[NEW]**
- `P256.Signing.PrivateKey()` — generate ECDSA private key
- `P256.Signing.PrivateKey.publicKey` — extract public key
- `P256.Signing.PrivateKey.signature(for:)` — produce ECDSA signature
- `P256.Signing.PublicKey.isValidSignature(_:for:)` — verify ECDSA signature
- `P384.Signing`, `P521.Signing` — ECDSA variants
- `Curve25519.Signing.PrivateKey` / `.PublicKey` — EdDSA
- `P256.KeyAgreement.PrivateKey`, `P384.KeyAgreement`, `P521.KeyAgreement`, `Curve25519.KeyAgreement` — ECDH
- `HMAC<SHA256>.authenticationCode(for:using:)` — HMAC generation
- `HMAC<SHA256>.isValidAuthenticationCode(_:authenticating:using:)` — constant-time HMAC verification
- `SecureEnclave.isAvailable` — check Secure Enclave availability **[NEW]**
- `SecureEnclave.P256.Signing.PrivateKey(accessControl:authenticationContext:)` — Secure Enclave key generation **[NEW]**
- `Insecure.MD5`, `Insecure.SHA1` — legacy hash support

### Security / LocalAuthentication
- `NSDataWritingOptions.completeFileProtection` — strongest file protection class
- `SecItemAdd` / `SecItemCopyMatching` — Keychain read/write
- `kSecAttrAccessibleWhenUnlocked` / `kSecAttrSynchronizable` — Keychain protection attributes
- `LAPolicy.deviceOwnerAuthenticationWithBiometricsOrWatch` — new macOS policy **[NEW]**
- `LAPolicy.deviceOwnerAuthenticationWithWatch` — new macOS policy **[NEW]**
- `SecTrustEvaluateAsyncWithError(_:_:_:)` — new async certificate evaluation with detailed error **[NEW]**

### Network / URLSession (TLS)
- `NWParameters.tls` — TLS connection with Network framework
- `URLSession` with `https://` endpoint — uses App Transport Security + TLS 1.3 by default
- TLS 1.3 — used by default on Apple platforms; no code change required if server supports it

### CloudKit
- `CKAsset` — file-backed encrypted asset
- `CKRecord` in private database — encrypted at rest and in transit

## Code Highlights

One-shot authenticated encryption with AES-GCM:

```swift
let key = SymmetricKey(size: .bits256)
let sealedBox = try AES.GCM.seal(plaintext, using: key)
let decrypted = try AES.GCM.open(sealedBox, using: key)
```

ECDSA signature with Secure Enclave and access control:

```swift
guard SecureEnclave.isAvailable else { /* fallback */ }

let accessControl = SecAccessControlCreateWithFlags(
    nil,
    kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
    [.privateKeyUsage, .userPresence],
    nil
)!

let context = LAContext()
context.touchIDAuthenticationAllowableReuseDuration = 10
context.localizedReason = "Authorize $10 transfer to Bob"

let privateKey = try SecureEnclave.P256.Signing.PrivateKey(
    accessControl: accessControl,
    authenticationContext: context
)
let signature = try privateKey.signature(for: transactionData)
```

Incremental hashing:

```swift
var hasher = SHA256()
while let chunk = inputStream.read() {
    hasher.update(data: chunk)
}
let digest = hasher.finalize()
```

## Takeaways

- Reach for system features first: Data Protection, Keychain, TLS via URLSession/Network framework, and CloudKit handle the most common security scenarios with strong defaults and hardware backing.
- Apple CryptoKit's Swift type system prevents entire classes of bugs (wrong key size, missing authentication, insecure nonce reuse) that plague raw C crypto APIs.
- Always store private keys in the Keychain or Secure Enclave — never in UserDefaults or in-memory variables that outlive their use.
- Deploying TLS 1.3 on your servers requires no code changes in your app; it is used automatically when the server advertises support.

---
_Source: WWDC19 Session 709 page (abstract, full transcript, and resource links)._
