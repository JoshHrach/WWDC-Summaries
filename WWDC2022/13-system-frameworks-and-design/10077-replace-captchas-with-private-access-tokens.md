# Replace CAPTCHAs with Private Access Tokens
**WWDC22 · Session 10077** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10077/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
Private Access Tokens are a new cryptographic mechanism, introduced in iOS 16 and macOS Ventura, that allow servers to verify a client request comes from a legitimate Apple device with a signed-in Apple ID — without tracking or fingerprinting the client, and without displaying CAPTCHAs. The technology is built on the IETF Privacy Pass working group standard and uses RSA Blind Signatures so that tokens are "unlinkable": a server receiving a token can verify it is valid but cannot identify who sent it or correlate multiple requests from the same user.

The client side requires no app code changes when using URLSession or WebKit. The adoption work is entirely server-side: select a token issuer, issue HTTP authentication challenges with the `PrivateToken` scheme, and validate tokens using the issuer's public key.

## Key Topics

### Why CAPTCHAs Are Problematic
- Poor user experience: interrupts flow, some are inaccessible to users with disabilities or language barriers.
- Privacy risk: servers use IP addresses and fingerprinting to risk-score clients; this is at odds with Safari, Mail Privacy Protection, and iCloud Private Relay.
- Still needed: bots, account fraud, and automated attacks remain real threats.

### How Private Access Tokens Work
1. **Client accesses server** — server responds with HTTP 401 and an authentication challenge using the new `PrivateToken` scheme, specifying a trusted token issuer.
2. **Client sends blinded request to iCloud attester** — the token request is "blinded" (cannot be linked back to the server challenge) before leaving the device.
3. **iCloud attester performs device attestation** — verifies device credentials from the Secure Enclave, verifies Apple ID is in good standing, and performs rate-limiting checks.
4. **Attester forwards to token issuer** — the issuer (e.g., Fastly, Cloudflare) receives the request; it does not know the client's identity, only that the iCloud attester approved it.
5. **Issuer signs the token** — returns a signed blinded token.
6. **Client unblinds the token** — transforms the signed blinded token into a final token that the server can verify with the issuer's public key.
7. **Client presents token to server** — the server validates the signature but cannot identify or track the client.

### Cryptographic Properties
- **RSA Blind Signatures** — the client blinds the token request so the issuer's signature cannot be linked to any specific network transaction.
- **Unlinkable** — the server receiving a validated token cannot determine which client sent it or correlate tokens from the same user across time.
- **Rate limiting** — performed at the iCloud attester level; abnormal request patterns (bot farms, compromised devices) are detected without tracking individuals.
- **One-time use** — tokens should be used once; servers should track presented tokens or require tokens to sign a unique server-issued nonce to prevent replay attacks.

### Server-Side Adoption
1. **Select a token issuer** — must be a large service handling requests for at least hundreds of servers (to prevent issuer identity from becoming a tracking vector). Current issuers in iOS 16/macOS Ventura beta: **Fastly** and **Cloudflare**. Additional issuers can register at `register.apple.com`.
2. **Issue HTTP authentication challenges** — send `WWW-Authenticate: PrivateToken issuer=<issuer-url>` in responses when client validation is desired. Challenges must come from first-party domains (not embedded third-party domains).
3. **Integrate with existing providers** — existing CAPTCHA/fraud-prevention providers may automatically issue challenges within their scripts.
4. **Validate returned tokens** — use the issuer's public key to verify token signatures.
5. **Prevent replay attacks** — track used tokens or require that each token sign a unique nonce included in the server challenge.
6. **Keep legacy fallback** — older clients won't respond to the challenge; the 401 response is the fallback for those clients. Authentication must be optional and non-blocking for page loads.

### App-Side Integration
- **No code changes required** when using `URLSession` or `WebKit` — the system automatically responds to `PrivateToken` challenges in the foreground.
- Requires iOS 16 / macOS Ventura and a signed-in Apple ID (used only for attestation, never shared with the server).
- If token fetching fails (app in background, no Apple ID), URLSession surfaces the 401 HTTP response so the app can handle it gracefully (e.g., fall back to CAPTCHA or retry).

## APIs & Frameworks

**URLSession (Foundation)**
- `URLSession` — **[NEW behavior]** automatically handles `WWW-Authenticate: PrivateToken` challenges when app is in foreground; no API changes required
- HTTP 401 response — received by URLSession when token fetch fails; app should handle gracefully

**WebKit**
- `WKWebView` — automatically handles `PrivateToken` authentication challenges; no changes required

**HTTP Authentication Scheme (Server-Side)**
- `WWW-Authenticate: PrivateToken issuer=<issuer-url>` — **[NEW]** server-to-client challenge header
- `Authorization: PrivateToken token=<base64-token>` — client-to-server token response header

**RSA Blind Signatures (underlying cryptography)**
- Standardized in IETF Privacy Pass working group
- Token issuers use RSA-PSS blind signing; Apple's Secure Enclave provides client device attestation

**Platform Requirements**
- iOS 16 / iPadOS 16 / macOS Ventura
- Apple ID signed in on device (used for attester verification only)
- Token issuer: Fastly and Cloudflare available in beta; more issuers via `register.apple.com`

## Code Highlights
No app-side code changes are required. The complete flow happens automatically through URLSession and WebKit.

Server challenge example (HTTP response header):
```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: PrivateToken issuer="https://issuer.example.com"
```

Client response (sent automatically by URLSession):
```
GET /resource HTTP/1.1
Authorization: PrivateToken token=<base64url-encoded-token>
```

Handling token challenge failure in URLSession (Swift):
```swift
// URLSession delivers the 401 response if token fetching fails
// (e.g., app is backgrounded, no Apple ID signed in)
let (data, response) = try await URLSession.shared.data(from: url)
if let httpResponse = response as? HTTPURLResponse,
   httpResponse.statusCode == 401 {
    // Handle: show fallback CAPTCHA, prompt user, or retry
}
```

## Takeaways
- Private Access Tokens eliminate CAPTCHAs for users on iOS 16 and macOS Ventura by using device-level attestation via iCloud, RSA Blind Signatures, and the IETF Privacy Pass standard — without tracking or identifying users.
- The entire adoption burden is on the server: select a trusted issuer (Fastly or Cloudflare in beta), issue `WWW-Authenticate: PrivateToken` challenges, and validate token signatures with the issuer's public key.
- Apps using URLSession or WebKit require zero code changes; the OS handles the full token request-and-response flow automatically when the app is in the foreground.
- Tokens are one-time-use; implement replay protection by tracking used tokens or requiring tokens to sign a server-issued nonce.
- The issuer must serve at least hundreds of server operators so that choosing a specific issuer cannot be used to infer which websites a user visits.

---
_Source: WWDC22 Session 10077 page (abstract, transcript, and related session links)._
