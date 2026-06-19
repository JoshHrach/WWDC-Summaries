# Verify Identity Documents on the Web
**WWDC25 · Session 232** · [Watch](https://developer.apple.com/videos/play/wwdc2025/232/)

_Platforms:_ iOS 26, iPadOS 26, Safari 26

## Overview
This session introduces the W3C Digital Credentials API, implemented in Safari 26, which allows websites to request verified identity credentials from identity documents stored in Wallet (such as driver's licenses in the ISO 18013-5 / mdoc format). It also covers the new `IdentityDocumentServices` framework that enables apps to register identity documents as credential providers, and explains how the system mediates credential requests while preserving user privacy and consent.

## Key Topics

### W3C Digital Credentials API
The Digital Credentials API is a browser-level API (`navigator.credentials.get({ digital: ... })`) that lets websites request specific fields from identity documents. The browser/OS mediates the request — the user chooses which identity document to present and reviews exactly which fields will be disclosed. No credential data reaches the website without explicit user approval. Requests and responses use the mdoc/ISO 18013-5 format.

### mdoc / ISO 18013-5
ISO 18013-5 defines the mobile document (mdoc) format used for mobile driver's licenses and other government identity documents. An mdoc is a CBOR-encoded credential with namespaced data elements (e.g., `org.iso.18013.5.1` namespace for driving license data). Verifiers request specific fields by namespace and element name.

### IdentityDocumentServices Framework
`IdentityDocumentServices` is a new iOS 26 framework that allows apps to register identity documents as credential providers. When a website makes a Digital Credentials request, the system presents a picker showing registered identity documents. The framework's registration store (`IdentityDocumentProviderRegistrationStore`) is where apps register supported document types.

### IdentityDocumentProviderRegistrationStore
The registration store is the root entry point for document providers. Apps conform to the `IdentityDocumentProvider` protocol and use the store to register their documents. At request time, the system calls into the registered provider to fulfill the credential request.

### MobileDocumentRegistration
`MobileDocumentRegistration` is used to register an mdoc (mobile document) in the identity document provider registration store. It specifies the document type and the fields the document can present.

### ISO18013MobileDocumentRequestContext
When the system routes a credential request to the app's identity document provider, it delivers an `ISO18013MobileDocumentRequestContext` that describes the verifier's request: which fields are being requested, the verifier's identity (certificate chain), and any reader engagement/session transcript data. The app validates the request, generates the cryptographic response, and returns the appropriate mdoc fields.

## APIs & Frameworks

**IdentityDocumentServices** (new framework, iOS 26) **[NEW]**
- `IdentityDocumentProviderRegistrationStore` **[NEW]** — root entry point for registering identity document providers
- `MobileDocumentRegistration` **[NEW]** — registers a mobile document (mdoc) with the system credential store
- `IdentityDocumentProvider` protocol **[NEW]** — app extension protocol for handling credential presentation requests
- `ISO18013MobileDocumentRequestContext` **[NEW]** — request context for ISO 18013-5 mdoc credential requests; provides verifier identity, requested data elements, session transcript

**Safari / WebKit (iOS 26 / macOS Tahoe 26)**
- W3C Digital Credentials API **[NEW]** — `navigator.credentials.get({ digital: { requests: [...] } })` — browser-mediated identity credential requests
- mdoc/ISO 18013-5 request and response format support **[NEW]**

## Code Highlights

```javascript
// Web: request specific fields from an mdoc
const result = await navigator.credentials.get({
  digital: {
    requests: [{
      protocol: "org.iso.18013.5.1.mdoc",
      data: {
        documentType: "org.iso.18013.5.1.mDL",
        nameSpaces: {
          "org.iso.18013.5.1": {
            "family_name": true,
            "given_name": true,
            "birth_date": true,
            "age_over_21": true
          }
        }
      }
    }]
  }
});
// result.data contains the cryptographically signed mdoc response
```

```swift
// App: register a mobile document as a credential provider
import IdentityDocumentServices

let store = IdentityDocumentProviderRegistrationStore()
let registration = MobileDocumentRegistration(
    documentType: "org.iso.18013.5.1.mDL",
    supportedFields: [...]
)
try await store.register(registration)
```

```swift
// App Extension: handle a credential request
func handleRequest(_ context: ISO18013MobileDocumentRequestContext) async throws -> Data {
    // Validate verifier certificate chain
    // Select requested fields from the stored mdoc
    // Generate CBOR-encoded signed response
    return signedMdocResponse
}
```

## Takeaways
- Websites can use the W3C Digital Credentials API in Safari 26 to request age verification, identity fields, or license status directly from Wallet — without building custom identity pipelines.
- The system mediates all credential requests: users see a picker and consent screen before any data is disclosed to the website.
- Apps that hold government identity documents (mobile driver's licenses) should implement `IdentityDocumentProvider` and register via `IdentityDocumentProviderRegistrationStore` to surface their documents in the system credential picker.
- Use `ISO18013MobileDocumentRequestContext` to validate verifier identity (certificate chain) and cryptographically sign only the requested fields — never send the full mdoc.

---
_Source: WWDC25 Session 232 page (abstract, chapter summaries, code samples, and resource links)._
