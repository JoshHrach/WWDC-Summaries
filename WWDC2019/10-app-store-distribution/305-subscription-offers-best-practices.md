# Subscription Offers Best Practices
**WWDC19 · Session 305** · [Watch](https://developer.apple.com/videos/play/wwdc2019/305/)

_Platforms:_ iOS 13, macOS Catalina 10.15

## Overview
Subscription Offers are a StoreKit feature that lets developers offer discounted pricing to existing or former subscribers for a specified duration — giving apps a powerful tool to reduce churn and win back lapsed subscribers. Unlike introductory offers, Subscription Offers can be redeemed multiple times per customer, and you control when and to whom they are displayed.

This session covers the complete implementation: creating offers and private keys in App Store Connect, generating cryptographic signatures on a server, using the new StoreKit APIs to present and purchase offers, understanding App Store eligibility rules, and designing effective distribution and anti-churn strategies.

## Key Topics

### App Store Connect Setup
- Create offers in App Store Connect under Features > In-App Purchases > any auto-renewable subscription > Subscription Prices > Create Promotional Offer.
- Each offer has a human-readable name and a unique **offer identifier** (product code) used to reference it in code.
- Offer types: pay-as-you-go, pay-up-front, or free trial — same as introductory offers.
- Create a **private key** under Users and Access > Keys > Subscriptions. App Store Connect generates a **key ID**. The private key can only be downloaded once — after download it is deleted from Apple servers permanently.
- A single key is valid across your entire developer account; multiple keys can be used to segment apps or for key rotation.

### Cryptographic Signature Generation (Server-Side)
- Every Subscription Offer transaction requires a server-generated cryptographic signature to prevent unauthorized redemptions.
- **Never generate signatures on-device** — the private key must remain on a secure server.
- Signature payload components (concatenated with Unicode U+2063 separator):
  1. App bundle identifier
  2. Key ID
  3. Product identifier
  4. Offer identifier
  5. Application username (one-way salted hash — never plain text)
  6. Nonce (random UUID v4 — prevents replay attacks)
  7. Timestamp (milliseconds since Unix Epoch)
- Signature algorithm: ECDSA with SHA-256, output in DER format, base-64 encoded.
- Timestamps expire after 24 hours — generate signatures just before presenting the offer, not in advance.
- For key rotation: update server logic to use the new key ID; no app update required.

### StoreKit API

**Retrieving Offer Details:**
- `SKProduct.discounts` — new array of `SKProductDiscount` objects for Subscription Offers **[NEW]**
- `SKProductDiscount.identifier` — the offer identifier string (nil for introductory offers)
- `SKProductDiscount.type` — new enum distinguishing introductory vs. subscription offers **[NEW]**
- Note: introductory offers still appear only on `SKProduct.introductoryPrice`, not in `discounts`.

**Making a Purchase with an Offer:**
- Create `SKPaymentDiscount(identifier:keyIdentifier:nonce:signature:timestamp:)` using values from the server response **[NEW]**
- Set `SKMutablePayment.paymentDiscount` **[NEW]**
- Set `SKMutablePayment.applicationUsername` (the hashed user ID used when generating the signature)
- Add payment to `SKPaymentQueue.default()`

**New Error Codes (`SKError.Code`):**
- `.invalidOfferIdentifier` — offer not found or disabled in App Store Connect **[NEW]**
- `.invalidOfferPrice` — pay-as-you-go offer price not lower than the base subscription price **[NEW]**
- `.invalidSignature` — signature doesn't validate against the key/payload **[NEW]**
- `.missingOfferParams` — required payload field is missing or empty (e.g., missing `applicationUsername`) **[NEW]**

### Eligibility Rules
- App Store requirement: customer must be an existing or previous subscriber to any auto-renewable subscription in your app.
- You can layer additional business rules on top (e.g., churned subscribers only, subscribers on a specific tier).
- Use server-to-server notifications (e.g., `DID_CHANGE_RENEWAL_STATUS`) to detect churn events and trigger offer delivery in real time.
- Eligibility logic lives on your server — no App Store validation of your custom rules.

### Distribution Strategies
- Present offers in-app when you detect churn signals (e.g., auto-renew disabled notification, subscription lapse event).
- Use push notifications, email, or in-app prompts to surface offers to lapsed subscribers.
- Deep link directly to an offer in your app using a custom URL scheme or Universal Link.
- Do not pre-generate signatures — generate fresh ones at the moment of presentation for security and freshness.

### Anti-Churn Strategies
- Target at-risk subscribers before they cancel (use billing retry events and renewal status changes).
- Use pay-as-you-go offers (e.g., 3 months at 50%) to lower the psychological barrier to staying subscribed.
- Use pay-up-front offers to deliver bundled value (e.g., 3 months + in-app content) for game and app bundles.
- Win-back campaigns: target former subscribers with relevant offers, using known preferences from their subscription history.

## APIs & Frameworks

### StoreKit **[NEW/UPDATED]**
- `SKProduct.discounts: [SKProductDiscount]` — subscription offer list **[NEW]**
- `SKProductDiscount.identifier: String?` — offer identifier **[NEW]**
- `SKProductDiscount.type: SKProductDiscount.Type` — introductory vs subscription offer **[NEW]**
- `SKPaymentDiscount` — holds offer payload + signature for a purchase **[NEW]**
  - `init(identifier:keyIdentifier:nonce:signature:timestamp:)` **[NEW]**
- `SKMutablePayment.paymentDiscount: SKPaymentDiscount?` **[NEW]**
- `SKMutablePayment.applicationUsername: String?` — hashed user ID
- `SKPaymentQueue.default().add(_:)` — submits the offer purchase
- `SKProductsRequest` — fetches product details including discounts
- `SKError.Code.invalidOfferIdentifier` **[NEW]**
- `SKError.Code.invalidOfferPrice` **[NEW]**
- `SKError.Code.invalidSignature` **[NEW]**
- `SKError.Code.missingOfferParams` **[NEW]**

### Server-Side (Language-Agnostic)
- ECDSA signature generation with SHA-256
- DER output format, base-64 encoded for transport
- Private key in PEM format
- UUID v4 nonce generation
- Millisecond-precision Unix timestamp

## Code Highlights

Creating an `SKPaymentDiscount` from server response:
```swift
let paymentDiscount = SKPaymentDiscount(
    identifier: offerIdentifier,
    keyIdentifier: keyID,
    nonce: UUID(uuidString: nonce)!,
    signature: signature,
    timestamp: NSNumber(value: timestamp)
)
```

Attaching the discount to a payment:
```swift
let payment = SKMutablePayment(product: skProduct)
payment.applicationUsername = hashedUsername
payment.paymentDiscount = paymentDiscount
SKPaymentQueue.default().add(payment)
```

Server-side signature payload (Node.js pseudocode):
```javascript
const payload = [
  bundleId, keyId, productId, offerId, appUsername, nonce, timestamp
].join('⁣');

const sign = crypto.createSign('SHA256');
sign.update(payload);
const signature = sign.sign(privateKey, 'base64');
```

## Takeaways
- Subscription Offers give you precise control over who gets an offer and when — build server-side business logic to target at-risk or churned subscribers.
- Always generate signatures on a secure server immediately before presenting an offer; never on-device and never cached more than 24 hours.
- The application username (hashed user ID) must match between signature generation and the `SKMutablePayment` — forgetting it causes `missingOfferParams`.
- Use `SKProduct.discounts` to fetch and display offer details; use `SKPaymentDiscount` + `SKMutablePayment.paymentDiscount` to complete the purchase.

---
_Source: WWDC19 Session 305 page (abstract, chapter summaries, code samples, and resource links)._
