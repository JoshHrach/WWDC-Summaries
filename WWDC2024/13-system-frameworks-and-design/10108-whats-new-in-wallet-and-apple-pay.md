# What's New in Wallet and Apple Pay
**WWDC24 · Session 10108** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10108/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, watchOS 11

## Overview
Two major areas of improvement in 2024: **Apple Pay on the web** (third-party browser support via QR-code scan flow, disbursement/funds-transfer on web, and a new `applePayCapabilities` API replacing browser detection) and **Wallet event ticket enhancements** (new "poster event ticket" style with rich artwork, venue maps, event guide, weather, music playlists, and automatic Live Activity).

## Key Topics

### Apple Pay in Third-Party Browsers
iOS 18 extends Apple Pay to any browser via a QR-code-based flow. When a user on a non-Safari browser clicks the Apple Pay button during checkout, a scannable code appears on screen. The user scans it with their iPhone camera and completes payment using Face ID — the same underlying payment system as a Safari transaction. The merchant receives the same payment response data as a standard Safari web transaction, with no additional code changes required for the payment handling.

**Required steps for developers:**
1. Import the Apple Pay JS SDK version 1.2.0 or higher in the `<head>` of the HTML document
2. Use the JavaScript SDK's Apple Pay button (not a CSS-implemented variant) — CSS buttons do not support non-Safari browser flows

### `applePayCapabilities` API
`canMakePaymentsWithActiveCard()` is deprecated. The new `applePayCapabilities()` API returns an object with `paymentCredentialStatus`:
- `paymentCredentialsAvailable` — device has an active qualifying card; show Apple Pay as the first payment option
- `paymentCredentialsUnavailable` — capable device but no eligible card; do not show Apple Pay button
- `paymentCredentialStatusUnknown` — Apple Pay is supported but card info unavailable (non-Safari or non-Apple device); show Apple Pay button, ordering at developer's discretion
- `applePayUnsupported` — do not show Apple Pay button

`canMakePayments()` (without active card check) remains unchanged and valid.

### Funds Disbursement on the Web
Funds transfer (paying out to users) is now available on the web in iOS 18 and macOS 15. Previously limited to in-app. The web API uses `disbursementRequest` inside the `modifiers` list of a `PaymentRequest`.

Key fields:
- `requiredRecipientContactFields` — contact info needed from the user
- `additionalLineItems` — includes a total deduction summary, an optional instant funds fee item (type `instantFundsOutFee`), and the disbursement amount (type `disbursement`)
- `supportsInstantFundsOut` — add to `merchantCapabilities` to offer instant transfer option with fees

Error types: `unsupportedCard` (card can't receive funds), `recipientContactInvalid` (contact data issue).

### Merchant Category Codes (MCC)
Merchants can now specify their ISO 7945 Merchant Category Code in a payment request. The payment sheet filters out ineligible cards automatically, improving transaction success rates and user experience.

### Wallet Pass Enhancements — Poster Event Ticket
NFC event tickets get a new "poster event ticket" visual style with focused event artwork, a structured footer showing seating info, and below-pass tiles for maps and event guide.

**Pass requirements:**
- `pass.json` must include semantic tags (machine-readable metadata) for event data
- Specify `preferredStyleVersions: ["posterEventTicket"]` in `pass.json` to opt into the new style (followed by `"eventTicket"` for backward compatibility)
- Signed with NFC entitlement for contactless entry
- New assets: `artwork` image (event-focused visual); `secondaryLogo` (shown in footer)
- Legacy fields (`primaryFields`, `secondaryFields`, `auxiliaryFields`, `headerFields`) still required for older OS versions

**Semantic tags used:**
- `eventStartDate` / `eventEndDate` — shown as date/time labels on the pass; date range for multi-day events
- `seats` dictionary — up to four elements of entry/seating info in the footer; or `admissionLevel` if no assigned seat
- `venueLocation` (latitude/longitude) — powers maps tile and weather forecast in event guide
- `venueName` — fallback for venue location
- `performerNames` / artist IDs — enable music playlist integration in event guide
- Semantic `url` tags — create tappable tiles linking to app experiences (food ordering, merchandise, bag policy, etc.)

**Event guide features** (appear below the pass automatically):
- Maps tile linking to Maps app for venue navigation
- Weather forecast for event day and location
- Venue details including venue map and freeform queue labels
- Music playlist link (from performer names or event name)
- Action tiles linking to app experiences via semantic URL tags

**Live Activity**: Passes with the new style automatically start a Live Activity on iPhone and Apple Watch at event time, displaying primary seating and entry info on the lock screen and Smart Stack — driven by the `seats` dictionary, with no additional developer code required.

## APIs & Frameworks

**Apple Pay on the Web (JavaScript)**
- Apple Pay JS SDK v1.2.0 **[NEW]** — required for third-party browser support
- `applePayCapabilities()` **[NEW]** — returns `paymentCredentialStatus`; replaces `canMakePaymentsWithActiveCard()`
- `ApplePaySession` — unchanged; now works in non-Safari browsers via QR code flow **[NEW behavior]**
- `disbursementRequest` in `PaymentRequest` modifiers **[NEW on web]**
  - `supportsInstantFundsOut` merchant capability **[NEW]**
  - `instantFundsOutFee` line item type **[NEW]**
  - `disbursement` line item type **[NEW]**
  - `unsupportedCard` / `recipientContactInvalid` `ApplePayError` types **[NEW]**
- Merchant Category Code (`merchantCategoryCode`) in payment request **[NEW]**

**PassKit (iOS/watchOS)**
- Poster event ticket style (`posterEventTicket` preferred style version) **[NEW]**
- `artwork` and `secondaryLogo` pass bundle assets **[NEW]**
- Semantic tags for event, venue, seating, performer data **[NEW]**
- Automatic Live Activity from event ticket pass **[NEW]**
- `seats` dictionary for structured seating/entry info **[NEW]**
- `admissionLevel` in semantic tags **[NEW]**
- Multi-day event date range (`eventEndDate`) **[NEW]**
- Event guide (maps, weather, music, actions) — automatic from semantic tags **[NEW]**

## Code Highlights

`applePayCapabilities` check (JavaScript):
```javascript
const capabilities = await ApplePaySession.applePayCapabilities("merchant.com.example");
switch (capabilities.paymentCredentialStatus) {
    case "paymentCredentialsAvailable":
        showApplePayFirst();
        break;
    case "paymentCredentialStatusUnknown":
        showApplePayOptionally();
        break;
    case "paymentCredentialsUnavailable":
    case "applePayUnsupported":
        hideApplePay();
        break;
}
```

Disbursement request (JavaScript):
```javascript
const disbursementRequest = {
    requiredRecipientContactFields: ["email"],
    additionalLineItems: [
        { label: "Total Withdrawal", amount: "50.00", type: "final" },
        { label: "Instant Transfer Fee", amount: "0.50", type: "instantFundsOutFee" },
        { label: "You Receive", amount: "49.50", type: "disbursement" }
    ]
};
```

Poster event ticket pass.json snippet:
```json
{
  "preferredStyleVersions": ["posterEventTicket", "eventTicket"],
  "semantics": {
    "eventStartDate": "2024-09-15T19:00-07:00",
    "eventEndDate": "2024-09-15T23:00-07:00",
    "seats": [
      { "description": "Section", "value": "101" },
      { "description": "Row", "value": "G" },
      { "description": "Seat", "value": "12" }
    ],
    "venueLocation": { "latitude": 37.3315, "longitude": -122.0307 },
    "performerNames": ["Artist Name"]
  }
}
```

## Takeaways
- Switch from `canMakePaymentsWithActiveCard()` to the new `applePayCapabilities()` API — it handles non-Safari browsers, non-Apple devices, and gives granular status values for smarter button display decisions.
- Use the JavaScript SDK Apple Pay button (not CSS) and import the SDK in `<head>` — these are required for the third-party browser QR-code flow to work.
- Adopt the "poster event ticket" style in pass.json with complete semantic tags to get the full experience: rich artwork display, event guide, venue maps, weather, music playlists, and automatic Live Activity — all with no additional backend or app code.
- Check with your payment processor for disbursement support before implementing the web disbursement flow.

---
_Source: WWDC24 Session 10108 page (abstract, chapter summaries, and resource links)._
