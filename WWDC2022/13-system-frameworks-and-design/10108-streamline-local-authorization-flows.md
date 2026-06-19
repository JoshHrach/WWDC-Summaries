# Streamline local authorization flows
**WWDC22 · Session 10108** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10108/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This session introduces a new high-level authorization API in LocalAuthentication — `LARight` and `LAPersistedRight` — that dramatically simplifies the code needed to protect app-defined resources with Touch ID, Face ID, Apple Watch, or device passcode. The existing `LAContext` + Security framework pattern remains available and recommended for use cases requiring fine-grained, low-level key/ACL access. For most authorization flows, however, the new API reduces multi-step Security framework orchestration to a few async calls.

The session also clarifies the distinction between authentication (verifying user identity) and authorization (verifying that the authenticated user is permitted to perform a specific operation on a resource), and shows how the new API aligns with this model.

## Key Topics

### Authentication vs. Authorization
- **Authentication**: verifying who the user is (e.g., Face ID, Touch ID, passcode).
- **Authorization**: verifying whether the authenticated user is allowed to perform an operation on a resource (e.g., sign with a specific Secure Enclave key).
- Authentication enables authorization: identity must be established before resource access can be granted.

### LAContext — Existing Flexible API
- `LAContext` evaluates user identity, drives Face ID/Touch ID UI, and interfaces with the Secure Enclave.
- Can evaluate `SecAccessControl` ACLs directly via `LAContext.evaluateAccessControl(_:operation:localizedReason:)`.
- Bind an authorized `LAContext` to a `SecItem` query via `kSecUseAuthenticationContext` to reuse the authorization for subsequent operations without re-prompting.
- Best for: apps that need direct access to keys, secrets, and access control lists with fine-grained control.

### LARight — New High-Level API (NEW in iOS 16 / macOS Ventura)
- `LARight` represents a named authorization right that gates access to an app-defined resource.
- Configured with an `LAAuthenticationRequirement`:
  - `.biometry(fallback: .devicePasscode)` — biometry with passcode fallback
  - `.biometryCurrentSet` — only the biometric enrollments currently enrolled at creation time (invalidates if biometrics are re-enrolled)
  - `.devicePasscode` — device passcode only
- **Lifecycle states**: `unknown` → `authorizing` → `authorized` or `notAuthorized`; can return from `authorized` → `notAuthorized` on explicit deauthorize or dealloc.
- Observe state changes:
  - `LARight.state` property (synchronous query)
  - KVO on `state`
  - Combine publisher
  - `NotificationCenter` — `.LAAuthorizationGranted` / `.LAAuthorizationRevoked` — observe all rights from one place
- **Key methods** (all `async throws`):
  - `LARight.checkCanAuthorize()` — fast check if authorization is possible (e.g., biometry enrolled); does not prompt user
  - `LARight.authorize(localizedReason:)` — presents system authorization UI; transitions to `.authorized` on success
  - `LARight.deauthorize()` — invalidates the right; forces re-authorization next time
- Authorization UI is a new system-driven sheet anchored inside the app window, providing context about the origin and purpose of the request.
- Keep a **strong reference** to the `LARight` instance to preserve its authorized state.

### LAPersistedRight — Persistent Authorization (NEW in iOS 16 / macOS Ventura)
- `LAPersistedRight` is an `LARight` backed by a **Secure Enclave key** and an ACL that mirrors the right's authorization requirements.
- Stored using `LARightStore.shared.saveRight(_:identifier:)` — persists the right across app sessions.
- Retrieved via `LARightStore.shared.right(forIdentifier:)`.
- Authorization requirements are **immutable** once saved.
- Key access:
  - `LAPersistedRight.key.publicKey.bytes` — always accessible; export or use for encryption/signature verification
  - `LAPersistedRight.key.sign(_:algorithm:)` — protected; requires prior `.authorize()`
  - `LAPersistedRight.key.decrypt(_:algorithm:)` — protected; requires prior `.authorize()`
- Optional: store a single **immutable secret** alongside the right at creation time; secret is returned only after authorization.
- Useful for: 2FA keys, signing keys, encrypted credential storage.

## APIs & Frameworks

**LocalAuthentication**

`LAContext` (existing)
- `LAContext.evaluateAccessControl(_:operation:localizedReason:)` — evaluate a `SecAccessControl` ACL; `async throws`
- `LAContext.evaluatePolicy(_:localizedReason:)` — evaluate biometry/passcode policy
- `kSecUseAuthenticationContext` — bind authorized context to SecItem query

`LARight` **[NEW]**
- `LARight(requirement: LAAuthenticationRequirement)` — create a right with specified auth requirements
- `LARight.checkCanAuthorize()` — `async throws` — check if authorization is currently possible
- `LARight.authorize(localizedReason: String)` — `async throws` — prompt user and transition to authorized
- `LARight.deauthorize()` — `async` — invalidate the right
- `LARight.state: LARight.State` — `.unknown`, `.authorizing`, `.authorized`, `.notAuthorized`

`LAPersistedRight` **[NEW]**
- `LARightStore.shared.saveRight(_ right: LARight, identifier: String)` — `async throws -> LAPersistedRight`
- `LARightStore.shared.right(forIdentifier: String)` — `async throws -> LAPersistedRight`
- `LARightStore.shared.removeRight(forIdentifier: String)` — `async throws`
- `LAPersistedRight.key: LAPrivateKey`
  - `.sign(_ data: Data, algorithm: SecKeyAlgorithm)` — `async throws -> Data` (requires authorization)
  - `.decrypt(_ data: Data, algorithm: SecKeyAlgorithm)` — `async throws -> Data` (requires authorization)
  - `.publicKey: LAPublicKey`
    - `.bytes` — `async throws -> Data` (always allowed)
    - `.encrypt(_ data: Data, algorithm: SecKeyAlgorithm)` — `async throws -> Data` (always allowed)

`LAAuthenticationRequirement` **[NEW]**
- `.biometry(fallback:)` — biometric; fallback can be `.devicePasscode` or `.none`
- `.biometryCurrentSet` — biometry limited to currently enrolled set (invalidates on re-enrollment)
- `.devicePasscode` — device passcode only

**Security framework (existing, for low-level use)**
- `SecItemCopyMatching` — query keychain items; use `kSecReturnAttributes` to retrieve ACL
- `SecKeyCreateSignature` — sign data with a private key reference
- `SecAccessControl` — ACL bound to a key item via `SecAttrAccessControl`
- `kSecAttrTokenIDSecureEnclave` — flag for Secure Enclave-backed keys

## Code Highlights

LAContext pattern — authorize and sign with Secure Enclave key:
```swift
// 1. Retrieve the ACL
let query: [String: Any] = [
    kSecClass as String: kSecClassKey,
    kSecAttrTokenID as String: kSecAttrTokenIDSecureEnclave,
    kSecAttrApplicationTag as String: "com.example.app.key",
    kSecReturnAttributes as String: true,
]
var item: CFTypeRef?
SecItemCopyMatching(query as CFDictionary, &item)
let accessControl = (item as! NSDictionary)[kSecAttrAccessControl]

// 2. Evaluate the ACL
let context = LAContext()
try await context.evaluateAccessControl(accessControl as! SecAccessControl,
                                        operation: .useKeySign,
                                        localizedReason: "Authentication required")

// 3. Retrieve key with bound context (no re-prompt)
let keyQuery: [String: Any] = [
    kSecClass as String: kSecClassKey,
    kSecAttrTokenID as String: kSecAttrTokenIDSecureEnclave,
    kSecAttrApplicationTag as String: "com.example.app.key",
    kSecReturnRef as String: true,
    kSecUseAuthenticationContext as String: context
]
var keyItem: CFTypeRef?
SecItemCopyMatching(keyQuery as CFDictionary, &keyItem)
let privateKey = keyItem as! SecKey
// 4. Sign
let signature = SecKeyCreateSignature(privateKey, algorithm, blob, &error) as Data
```

LARight — login gate (new high-level API):
```swift
func login() async {
    self.loginRight = LARight(requirement: .biometry(fallback: .devicePasscode))
    do {
        try await loginRight.checkCanAuthorize()
    } catch {
        navigateTo(section: .public); return
    }
    do {
        try await loginRight.authorize(localizedReason: "Log in to access your profile")
        navigateTo(section: .protected)
    } catch {
        showError(.authenticationRequired)
    }
}

func logout() async {
    await loginRight.deauthorize()
}
```

LAPersistedRight — generate 2FA key pair and sign a challenge:
```swift
// On first run: create and persist the right
func generateClientKeys() async throws -> Data {
    let login2FA = LARight(requirement: .biometryCurrentSet)
    let persisted2FA = try await LARightStore.shared.saveRight(login2FA, identifier: "2fa")
    return try await persisted2FA.key.publicKey.bytes  // export public key to server
}

// On subsequent sign requests
func signChallenge(_ challenge: Data, algorithm: SecKeyAlgorithm) async throws -> Data {
    let persisted2FA = try await LARightStore.shared.right(forIdentifier: "2fa")
    try await persisted2FA.authorize(localizedReason: "Biometric authentication required")
    return try await persisted2FA.key.sign(challenge, algorithm: algorithm)
}
```

## Takeaways
- `LARight` and `LAPersistedRight` are new high-level abstractions in LocalAuthentication (iOS 16 / macOS Ventura) that reduce multi-step `LAContext` + Security framework authorization flows to a handful of `async throws` calls.
- Use `LARight` for transient session-scoped authorization gates (login required, protected view); use `LAPersistedRight` when you need a stable Secure Enclave key pair that persists across app sessions (2FA, signing, encrypted storage).
- `biometryCurrentSet` binds the right to the exact set of biometric enrollments at creation time; it automatically becomes invalid if the user re-enrolls or changes biometrics — appropriate for security-sensitive operations like 2FA.
- Always hold a strong reference to an `LARight` to preserve its authorized state; deallocation transitions the right to `notAuthorized`.
- Stick with `LAContext` + Security framework when you need low-level access control list evaluation, multi-operation key reuse within a session, or use cases that aren't well-modeled by a single named right.

---
_Source: WWDC22 Session 10108 page (abstract, transcript, code samples, and resource links)._
