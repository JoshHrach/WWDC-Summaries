# Explore Verifiable Health Records
**WWDC21 · Session 10089** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10089/)

_Platforms:_ iOS 15

## Overview
iOS 15 extends the Health Records feature to support SMART Health Cards — cryptographically signed verifiable health records based on the FHIR standard. Users can import these records (via connected providers, `.smart-health-cards` file download, or QR code scan) into the Health app, and apps with a special entitlement can query for and verify them using the new `HKVerifiableClinicalRecordQuery` API combined with CryptoKit for signature validation.

This session covers the SMART Health Card data format (JSON Web Signature / JWS wrapping FHIR bundles), the new HealthKit query API with its per-sample authorization model, and a step-by-step walkthrough of fetching an issuer's public keys from a `.well-known` endpoint and verifying the JWS signature with CryptoKit's `P256.Signing`.

## Key Topics

**SMART Health Cards and JWS**
A SMART Health Card bundles a FHIR Patient resource and one or more clinical resources (e.g., Immunization) into a credential payload, then signs it as a JSON Web Signature (JWS). The JWS header contains the signing algorithm (ES256), a public key thumbprint (kid), and a compression algorithm (DEF). The payload includes the issuer URL, issued date, optional expiration, and the FHIR bundle. The issuer publishes their public keys at `{issuer}/.well-known/jwks.json`.

**HKVerifiableClinicalRecordQuery**
New in iOS 15, this query class uses a per-sample authorization model: instead of requesting type-level access upfront, each query execution displays a picker where the user selects exactly which records to share. Authorization is one-time per query execution and does not grant persistent access. Requires a special HealthKit entitlement for Verifiable Health Records.

**Signature Verification with CryptoKit**
After retrieving the JWS from an `HKVerifiableClinicalRecord`, apps can verify authenticity by: parsing the compact serialized JWS, decompressing the DEF-compressed payload, downloading the issuer's JWKS from the `.well-known` endpoint, selecting the correct key by thumbprint, and calling `P256.Signing.PublicKey.isValidSignature(_:for:)`.

**Privacy Design**
Records stored in the Health app are encrypted when the phone is locked. The SMART Health Card specification uses minimal data profiles. The entitlement application process includes additional obligations, and the per-sample authorization gives users granular control.

## APIs & Frameworks

- **HealthKit** — extended in iOS 15
- `HKVerifiableClinicalRecordQuery` **[NEW]** — queries for verifiable health records
  - `init(recordTypes: [String], predicate: NSPredicate?, resultsHandler: ...)` 
  - `recordTypes` — string array specifying required FHIR resource types in the record
  - Results returned in a per-execution authorization sheet (not persistent)
- `HKVerifiableClinicalRecord` **[NEW]** (subclass of `HKSample`)
  - `var subject: HKVerifiableClinicalRecordSubject`
  - `var issuerIdentifier: String`
  - `var issuedDate: Date`
  - `var expirationDate: Date?`
  - `var itemNames: [String]`
  - `var jwsRepresentation: Data` — the raw JWS compact serialization
- `HKHealthStore` — used to execute `HKVerifiableClinicalRecordQuery`
- `NSPredicate` + `HKQuery.predicateForVerifiableClinicalRecords(withRelevantDateWithin:)` convenience constructor **[NEW]**
- HealthKit entitlement: `com.apple.developer.healthkit.background-delivery` (Verifiable Health Records variant) — requires application
- **CryptoKit** — used for signature verification
  - `P256.Signing.PublicKey` — ES256 public key type
  - `P256.Signing.PublicKey.isValidSignature(_:for:)` — JWS signature verification
- **SMART Health Cards** specification — `https://smarthealth.cards`
  - JSON Web Signature (JWS) compact serialization format
  - `.smart-health-cards` file extension
  - `.well-known/jwks.json` — issuer public key endpoint (JWKS)
  - ES256 signature algorithm
  - DEF (DEFLATE) payload compression
  - FHIR R4 (`Bundle` resource containing `Patient` + clinical resources)
- **Combine** — used for async URLSession fetch of JWKS
- `URLSession.dataTaskPublisher(for:)` — downloads issuer public keys
- **HL7 FHIR** (R4) — underlying health data standard

## Code Highlights

Query for verifiable health records:
```swift
import HealthKit
let healthStore = HKHealthStore()
let recordTypes = ["https://smarthealth.cards#immunization"]
let predicate = HKQuery.predicateForVerifiableClinicalRecords(withRelevantDateWithin: DateInterval(...))
let query = HKVerifiableClinicalRecordQuery(recordTypes: recordTypes, predicate: predicate) { query, samples, error in
    guard let samples = samples else { return }
    // samples contains user-selected HKVerifiableClinicalRecord instances
}
healthStore.execute(query)
```

Verifying the JWS signature with CryptoKit (outline):
```swift
// 1. Parse compact serialization: header.payload.signature (Base64URL-encoded parts)
// 2. Decompress DEF-compressed payload
// 3. Fetch issuer JWKS from {issuer}/.well-known/jwks.json
// 4. Select key matching header's "kid" thumbprint
// 5. Verify:
let signingInput = "\(encodedHeader).\(encodedPayload)".data(using: .utf8)!
let isValid = try publicKey.isValidSignature(signature, for: signingInput)
```

## Takeaways

- `HKVerifiableClinicalRecordQuery` uses a unique per-sample, per-execution authorization model — each query shows a picker and returns only what the user explicitly selects that time.
- The JWS signature can be verified entirely on-device using CryptoKit's `P256.Signing` and the issuer's public keys from their `.well-known/jwks.json` endpoint.
- Accessing verifiable health records requires a special HealthKit entitlement application (not just the standard HealthKit permission).
- The feature is built on open standards: SMART Health Cards spec, FHIR R4, JWS (RFC 7515), and JWKS (RFC 7517).

---
_Source: WWDC21 Session 10089 page (abstract, chapter summaries, code samples, and resource links)._
