# Secure your app: threat modeling and anti-patterns
**WWDC20 · Session 10189** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10189/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This session introduces threat modeling as a structured practice for identifying security risks early in the app development cycle. Using a fictional banking app as a running example, the session walks through building a threat model — identifying assets, entry points, threat actors, and potential attack scenarios — and then maps those threats to concrete mitigations available in Apple's platform.

The second half covers security anti-patterns commonly found in real-world iOS apps: insecure data storage, improper authentication, URL scheme hijacking, and other implementation mistakes that undermine security even when the underlying platform is sound.

## Key Topics

### Threat Modeling
- A threat model is a structured inventory of: **assets** (what to protect), **entry points** (how attackers reach the app), **threat actors** (who might attack and why), and **threat scenarios** (how an attack might unfold)
- Assets include: user credentials, financial data, biometric data, session tokens, PII
- Entry points include: network APIs, URL schemes, clipboard, pasteboard, iCloud sync, push notifications, app extensions, and inter-app communication
- Threat actors include: malicious apps on the same device, network attackers (MITM), malicious servers, physical device access

### Anti-Pattern 1: Insecure Data Storage
- Storing sensitive data in `UserDefaults`, unprotected files, or SQLite without file protection
- Fix: store secrets in the **Keychain** (`SecItemAdd`, `kSecAttrAccessible`); for files, use `NSFileProtectionComplete` or `NSFileProtectionCompleteUnlessOpen`
- Do not store passwords or tokens in `UserDefaults` — it is not encrypted
- Avoid logging sensitive data with `print` / `NSLog` — logs are accessible to other processes in some contexts

### Anti-Pattern 2: Weak Authentication
- Relying solely on a PIN or passcode with no attempt limit allows brute-force
- Fix: implement exponential backoff, lockout after N failures, and consider requiring biometric authentication with `LAContext` / Local Authentication framework
- Always validate server-side authentication; local checks are for UX only, not security
- Use `LAPolicy.deviceOwnerAuthenticationWithBiometrics` for biometric-only; `LAPolicy.deviceOwnerAuthentication` for biometric + passcode fallback

### Anti-Pattern 3: Insecure Network Communication
- Disabling App Transport Security (`NSAllowsArbitraryLoads`) without justification exposes traffic to MITM
- Fix: use HTTPS with valid certificates; enable ATS; use `URLSession` with certificate pinning where appropriate
- Validate server certificates; avoid self-signed certs in production
- Use `SecTrustEvaluateWithError` for custom certificate validation

### Anti-Pattern 4: URL Scheme Hijacking
- Custom URL schemes (e.g., `myapp://`) can be registered by any app — a malicious app can intercept OAuth redirect URIs
- Fix: use **Universal Links** (HTTPS-based, domain-verified) instead of custom URL schemes for OAuth and sensitive deep links
- Validate the caller of URL scheme invocations; do not trust `sourceApplication` in older APIs
- For OAuth: use `ASWebAuthenticationSession` which handles the redirect securely in a system browser

### Anti-Pattern 5: Pasteboard and Clipboard Exposure
- Sensitive data (passwords, tokens) placed on `UIPasteboard.general` is readable by any app
- Fix: use a dedicated pasteboard for sensitive data, set a short expiration, or avoid placing sensitive data on the clipboard at all

### Anti-Pattern 6: Improper Access Control for App Extensions
- App extensions share a container with the host app; data in the shared container may be accessible to extensions with weaker permissions
- Fix: use group containers only for data extensions legitimately need; store sensitive data in the Keychain with access group restrictions

### Anti-Pattern 7: Binary Protections
- Shipping without bitcode, without Position Independent Executable (PIE), or with debug symbols embedded weakens resistance to reverse engineering
- Enable **Stack Canaries**, **Address Space Layout Randomization (ASLR)**, and avoid leaving debug symbols in release builds

## APIs & Frameworks

- **Security** framework
  - `SecItemAdd(_:_:)`, `SecItemCopyMatching(_:_:)`, `SecItemUpdate(_:_:)`, `SecItemDelete(_:)` — Keychain operations
  - `kSecAttrAccessible` — Keychain item accessibility: `kSecAttrAccessibleWhenUnlocked`, `kSecAttrAccessibleAfterFirstUnlock`, `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`
  - `kSecAttrAccessGroup` — Keychain access groups for sharing between apps/extensions
  - `SecTrustEvaluateWithError(_:_:)` — custom certificate validation
- **LocalAuthentication** framework
  - `LAContext` — biometric and passcode authentication
  - `LAContext.evaluatePolicy(_:localizedReason:reply:)` — trigger authentication
  - `LAPolicy.deviceOwnerAuthenticationWithBiometrics` — Face ID / Touch ID only
  - `LAPolicy.deviceOwnerAuthentication` — biometric + passcode fallback
  - `LAContext.canEvaluatePolicy(_:error:)` — check availability
- **Foundation**
  - `FileManager` — file protection attributes
  - `URLFileProtection` — `.complete`, `.completeUnlessOpen`, `.completeUntilFirstUserAuthentication`
  - `UserDefaults` — NOT appropriate for sensitive data
  - `UIPasteboard` — `.general` exposes data to all apps; use named pasteboards with expiration
- **AuthenticationServices** framework
  - `ASWebAuthenticationSession` **[NEW / preferred for OAuth]** — secure in-app browser for OAuth flows; handles redirect URI securely
- **URLSession** — HTTPS with ATS; `URLSessionDelegate` for certificate pinning via `urlSession(_:didReceive:completionHandler:)`
- App Transport Security (`Info.plist`) — `NSAppTransportSecurity`, `NSAllowsArbitraryLoads` (avoid), `NSExceptionDomains`
- Universal Links — `apple-app-site-association` file on server; `NSUserActivityTypes` in `Info.plist`
- **App Extensions** — shared app group containers (`FileManager.containerURL(forSecurityApplicationGroupIdentifier:)`)

## Code Highlights

Store a credential securely in the Keychain:
```swift
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: "user@example.com",
    kSecValueData as String: passwordData,
    kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
]
SecItemAdd(query as CFDictionary, nil)
```

Biometric authentication:
```swift
let context = LAContext()
var error: NSError?
if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
    context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics,
                           localizedReason: "Authenticate to access your account") { success, error in
        // handle result on main thread
    }
}
```

Use ASWebAuthenticationSession instead of custom URL scheme for OAuth:
```swift
let session = ASWebAuthenticationSession(url: authURL, callbackURLScheme: nil) { callbackURL, error in
    // system handles redirect securely
}
session.presentationContextProvider = self
session.start()
```

## Takeaways

- Build a threat model before writing security-sensitive code — identifying assets, entry points, and threat actors upfront guides which platform protections to apply.
- Store all secrets in the Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`; never use `UserDefaults` for credentials, tokens, or PII.
- Replace custom URL schemes with Universal Links for deep links and `ASWebAuthenticationSession` for OAuth — both are domain-verified and resistant to hijacking by other apps.
- Validate all authentication server-side; local biometric / PIN checks are UX conveniences, not security boundaries.

---
_Source: WWDC20 Session 10189 page (abstract, chapter summaries, code samples, and resource links)._
