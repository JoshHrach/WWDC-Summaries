# Verify App Dependencies with Digital Signatures
**WWDC23 · Session 10061** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10061/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10 (Xcode 15 feature)

## Overview
Xcode 15 introduces automatic dependency signature verification for XCFrameworks, making supply chain security a first-class development feature. The session explains the risks of third-party SDK dependencies, how code signing works using developer certificates and CDHash-based signatures, and how Xcode now automatically records and re-verifies the signing identity of every XCFramework added to a project on each build.

The feature addresses supply chain attacks — where a dependency is tampered with during distribution — by cryptographically binding the binary and its metadata (including privacy manifest files) to the developer's identity. When a signature becomes invalid, the identity changes, or a certificate is revoked, Xcode raises a build error with a clear description and offers to remove the compromised framework from the project.

SDK authors are encouraged to start signing their XCFrameworks using the `codesign` command-line tool with an Apple Developer Program certificate and a secure timestamp, which enables Xcode to automatically validate new certificate generations and provides the highest level of trust for SDK consumers.

## Key Topics

### How Code Signing Works
- A **CDHash** (Code Directory hash) is generated from the compiled binary.
- The hash is signed with the developer's private key (represented by a developer certificate containing a public key).
- An optional **secure timestamp** (attested by Apple) is embedded to prove when the signature was created.
- If any file in the XCFramework changes after signing, the signature is invalidated.
- The signature resides in the `_CodeSignature` directory within the XCFramework bundle.
- Protects all files inside the XCFramework, including privacy manifest files.

### Xcode 15 Dependency Signature Verification (NEW)
- Xcode's Inspector shows a **Signature** section for XCFrameworks, displaying certificate author and identity type.
- **First use**: Xcode records the signing identity (Apple Developer Program certificate or self-signed certificate SHA-256 fingerprint).
- **Subsequent builds**: Xcode automatically re-verifies that the identity has not changed.
- Three identity tiers:
  1. **Apple Developer Program**: Apple validates certificate validity (including revocation) and uniqueness; automatic trust for new certificates from same developer.
  2. **Self-signed**: Xcode compares the SHA-256 certificate fingerprint; manual verification with SDK author required for fingerprint changes.
  3. **Unsigned**: noted in the Inspector; no automatic verification.
- **Build errors** are raised when: signature is invalid, identity has changed, framework was signed after certificate expiry, or ADP certificate was revoked by Apple.
- When identity changes, Xcode shows expected vs actual identity details and offers to remove the framework.

### For App Developers
- Adding a signed XCFramework: the Inspector's Signature view shows team details immediately.
- A mismatch (e.g., Apple Developer Program cert replaced by self-signed cert) causes a build failure automatically.
- Developers can accept a legitimate identity change (e.g., SDK ownership transfer) or reject it and remove the framework.
- Encourage SDK vendors to sign their distributions to get maximum protection.

### For SDK Authors
- Use **Apple Distribution** certificate for public SDK releases; **Apple Development** certificate for test distributions.
- Enterprise Program members: **iOS Distribution** or **App Development** certificates.
- Self-signed certificates: responsible for publishing and sharing the certificate fingerprint with SDK clients.
- Sign using the `codesign` tool; include `--timestamp` for an Apple-attested secure timestamp.
- Signing can be applied to already-published frameworks without a new build.
- Automate signing via a build script to ensure every release is signed.

## APIs & Frameworks

- `codesign` command-line tool — signs XCFrameworks using a developer identity
- `--timestamp` flag **[key for XCFramework signing]** — embeds Apple-attested secure timestamp in signature
- `--sign` flag — specifies the signing identity (e.g., "Apple Distribution: ...")
- `_CodeSignature` directory — within XCFramework bundle, contains the code signature
- CDHash (Code Directory hash) — cryptographic hash of the compiled binary used in code signing
- **Xcode 15 Inspector "Signature" section** **[NEW]** — displays signing identity info for XCFrameworks
- **Xcode 15 automatic XCFramework signature verification** **[NEW]** — records and re-verifies identity on every build
- Apple Developer Program certificate — highest trust; validated and revocable by Apple
- Apple Distribution certificate — for publicly distributed SDKs
- Apple Development certificate — for test/debug SDK distributions
- Self-signed certificate — lower trust; requires manual fingerprint verification
- SHA-256 certificate fingerprint — used to track self-signed certificate identity across builds
- Privacy manifests (`.xcprivacy`) — protected by XCFramework code signature; referenced in session 10060

## Code Highlights

```bash
# Sign an XCFramework with an Apple Developer Program identity (includes secure timestamp)
codesign --timestamp -v --sign "Apple Distribution: Truck to Table (UA527FUGW7)" BirdFeeder.xcframework

# Verify a signature manually
codesign --verify --verbose BirdFeeder.xcframework

# Display signing details
codesign --display --verbose=4 BirdFeeder.xcframework
```

## Takeaways
- Xcode 15 automatically records and re-verifies XCFramework signing identities on every build, turning supply chain security from a manual burden into an automatic safeguard.
- Apple Developer Program identities provide the strongest protection: Apple validates the certificate, handles revocation, and automatically trusts certificate renewals.
- SDK authors should sign XCFrameworks using `codesign --timestamp --sign "Apple Distribution: ..."` and ideally automate this in their build pipeline.
- Self-signed certificates are supported for developers outside the ADP, but require manual fingerprint verification by SDK consumers when the certificate changes.

---
_Source: WWDC23 Session 10061 page (abstract, chapter summaries, code samples, and resource links)._
