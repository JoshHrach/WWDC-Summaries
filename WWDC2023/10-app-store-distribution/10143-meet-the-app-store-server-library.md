# Meet the App Store Server Library
**WWDC23 · Session 10143** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10143/)

_Platforms:_ iOS, macOS, tvOS, watchOS, visionOS (server-side library; supports Swift, Java, Node.js, Python backends)

## Overview
The App Store Server Library is a new open-source, multi-language library that simplifies server-side integration with App Store Commerce APIs. Launched in beta at WWDC 2023, the library ships in four languages — Swift, Java, Node.js, and Python — and provides ready-made implementations of the most complex App Store server tasks: generating JSON Web Tokens (JWTs) for the App Store Server API, verifying JWS-signed transaction data, extracting transaction IDs from legacy app receipts, and generating subscription promotional offer signatures.

The library directly addresses the common pain points of adopting App Store Server API and App Store Server Notifications v2: JWT construction, certificate chain validation, OCSP revocation checking, and JWS signature verification. It also provides a migration path away from the now-deprecated `verifyReceipt` endpoint.

## Key Topics

### Four Core Capabilities
1. **App Store Server API client** – handles JWT creation so callers can invoke any of the dozen API endpoints without implementing OAuth/JWT boilerplate themselves
2. **Signed data verification (`SignedDataVerifier`)** – validates JWS transactions, renewal info, and server notifications against Apple's certificate chain including OCSP revocation checks
3. **Receipt transaction extraction (`ReceiptUtility`)** – extracts a transaction ID from a Base64-encoded legacy app receipt, enabling migration to `GetTransactionHistory` without calling `verifyReceipt`
4. **Promotional offer signature generation** – signs subscription promotional offer requests using the in-app purchase private key

### App Store Server API Client
- Configured with issuer ID, key ID, private key (downloaded once from App Store Connect → Users & Access → Keys → In-App Purchase), bundle ID, and environment (`sandbox` / `production`)
- Instantiate `AppStoreServerAPIClient`; use it to call any endpoint
- Session demonstrates calling the **Request a Test Notification** endpoint, which triggers the App Store server to POST a `TEST` notification to the developer's configured URL
- Recommended workflow: start with sandbox/TestFlight; rotate private keys immediately if compromised; check Apple Root CA updates regularly

### JWS Signed Data Verification
- JWS consists of three Base64 URL-encoded sections separated by periods: header, payload, signature
- Header contains `alg` (always `ES256`) and `x5c` (certificate chain array)
- Certificate chain: leaf → Apple Worldwide Developer Relations intermediate CA → Apple Root CA
- Verification steps:
  1. Verify each certificate is signed by the prior certificate in the chain
  2. Validate certificate dates, formatting, and purpose OIDs:
     - Leaf: Mac App Store Receipt Signing OID
     - Intermediate: Apple Worldwide Developer Relations OID
     - Root: must exactly match a downloaded Apple Root CA certificate
  3. Perform OCSP revocation check (RFC 6960) on each certificate
  4. Verify the JWS signature against the leaf certificate's public key
  5. Confirm `appAppleId`, `bundleId`, and `environment` fields match the expected app
- `SignedDataVerifier` performs all steps automatically; configure with the set of Apple Root CA certificates, bundle ID, app Apple ID (optional in sandbox), and whether to perform online OCSP checks
- `onlineChecks: true` for freshly received notifications; `false` for historical data where certs may have expired

### Receipt Migration to App Store Server API
- `verifyReceipt` is now **deprecated**; all new features ship only in JWS signed data (StoreKit 2, App Store Server API, Server Notifications v2)
- `ReceiptUtility.extractTransactionIdFromAppReceipt(_:)` extracts a transaction ID directly from a Base64-encoded receipt — no network round trip required
- The extracted ID (which may be any transaction ID, not necessarily the original) can now be passed to `GetTransactionHistory`; the endpoint was updated to accept any transaction ID, not just original transaction IDs
- Pattern: receive receipt from client → extract ID → call `GetTransactionHistory` with paging; store the revision token to avoid re-fetching the full history each time

### Transaction History Filtering
- `TransactionHistoryRequest` accepts filters: `productType` (e.g., `CONSUMABLE`), `excludeRevoked`, sort order
- Paginate using `revision` token from each response until `hasMore` is `false`
- Decode paged transactions with `SignedDataVerifier` to get strongly typed transaction data

## APIs & Frameworks

- **App Store Server Library** **[NEW]** – open-source multi-language library (GitHub: `apple/app-store-server-library-swift`, `-python`, `-node`, `-java`)
- `AppStoreServerAPIClient` **[NEW]** – client class; handles JWT generation; initialized with issuerId, keyId, privateKey, bundleId, environment
- `SignedDataVerifier` **[NEW]** – verifies JWS transactions, renewal info, and server notifications; validates certificate chain + OCSP + signature + app/environment fields
- `ReceiptUtility` **[NEW]** – `extractTransactionIdFromAppReceipt(_:)` for migrating off `verifyReceipt`
- Promotional offer signature generator **[NEW]** – signs offers using in-app purchase private key
- **App Store Server API** – server-to-server REST API; `GetTransactionHistory`, `GetTestNotification`, `GetTestNotificationStatus`, and ~10 additional endpoints
- `GetTransactionHistory` endpoint – now accepts any transaction ID (not only original) **[UPDATED]**
- `TransactionHistoryRequest` **[NEW]** – filter/sort object for `GetTransactionHistory`; `productType`, `excludeRevoked`, `sort`
- **App Store Server Notifications v2** – signed JWS payloads POSTed to developer server URL; notification type `TEST`
- JWS (JSON Web Signature) – signed data format; `JWSTransaction`, `JWSRenewalInfo`, `appTransaction`
- OCSP (Online Certificate Status Protocol, RFC 6960) – certificate revocation checking
- Apple Root CA / Apple Certificate Authority – root of trust; download from `apple.com/certificateauthority`
- JWT (JSON Web Token) – authentication token for App Store Server API; algorithm ES256
- `verifyReceipt` endpoint – **[DEPRECATED]** in 2023; replaced by App Store Server API

## Code Highlights

Configure the API client (Java example from session):
```java
AppStoreServerAPIClient client = new AppStoreServerAPIClient(
    privateKey,
    keyId,
    issuerId,
    bundleId,
    Environment.SANDBOX
);
SendTestNotificationResponse response = client.requestTestNotification();
String testNotificationToken = response.getTestNotificationToken();
```

Create a `SignedDataVerifier` and decode a notification:
```java
Set<X509Certificate> rootCerts = loadRootCertificates(); // from Apple PKI
SignedDataVerifier verifier = new SignedDataVerifier(
    rootCerts,
    bundleId,
    /*appAppleId=*/ null,          // null acceptable in sandbox
    Environment.SANDBOX,
    /*onlineChecks=*/ true
);
ResponseBodyV2DecodedPayload payload =
    verifier.verifyAndDecodeNotification(signedPayload);
```

Migrate from `verifyReceipt` using `ReceiptUtility`:
```java
ReceiptUtility receiptUtility = new ReceiptUtility();
Optional<String> transactionId =
    receiptUtility.extractTransactionIdFromAppReceipt(base64EncodedReceipt);

if (transactionId.isPresent()) {
    TransactionHistoryRequest request = new TransactionHistoryRequest()
        .productType(List.of(ProductType.CONSUMABLE))
        .excludeRevoked(true)
        .sort(Order.DESCENDING);

    TransactionHistoryResponse response = null;
    List<String> transactions = new ArrayList<>();
    do {
        String revision = response != null ? response.getRevision() : null;
        response = client.getTransactionHistory(
            transactionId.get(), revision, request);
        transactions.addAll(response.getSignedTransactions());
    } while (response.isHasMore());
}
```

## Takeaways
- The App Store Server Library eliminates the most error-prone parts of App Store server integration — JWT construction, X.509 certificate chain validation, OCSP revocation checking, and JWS signature verification — reducing them to a few lines of configuration.
- `verifyReceipt` is deprecated; use `ReceiptUtility` to extract a transaction ID from existing receipts and migrate to `GetTransactionHistory` without requiring app updates from users.
- Always verify signed data server-side before unlocking content; after verification, additionally check `bundleId`, `appAppleId`, and `environment` to confirm the data targets the correct app.
- Store only the `revision` token from `GetTransactionHistory` responses — do not re-fetch the full history; page through from the stored revision token on subsequent requests.

---
_Source: WWDC23 Session 10143 page (abstract, chapter summaries, transcript, and resource links)._
