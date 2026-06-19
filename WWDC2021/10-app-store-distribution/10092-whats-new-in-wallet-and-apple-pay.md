# What's New in Wallet and Apple Pay
**WWDC21 · Session 10092** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10092/)

_Platforms:_ iOS 15, iPadOS 15, watchOS 8

## Overview
This session covers three areas: Wallet platform updates (digital IDs, home keys, multi-pass downloads, automatic pass expiry hiding), a ground-up redesign of the Apple Pay payment sheet in SwiftUI with improved flows for new and returning users, and a set of new PassKit and Apple Pay on the Web APIs for shipping date ranges, read-only pickup addresses, coupon codes, and a new JavaScript Apple Pay button.

## Key Topics

### Wallet Platform Updates
- **Digital IDs**: Driver's licenses and state IDs in Wallet (iOS 15, select US states); protected by Secure Element; accepted at TSA airport checkpoints.
- **Home Keys**: HomeKit-connected lock support — tap to unlock with a pass in Wallet.
- **Multi-Pass Downloads**: Bundle `.pkpass` files into a `.pkpasses` zip archive with the correct MIME type to download all passes in a single tap from Safari.
- **Automatic Pass Expiry Hiding**: Wallet now auto-hides passes that have an `expirationDate` in the past, a `relevantDate` older than one day, or have been voided. Developers should ensure these fields are correctly set in their pass JSON.
- **PKPass Icon Size**: Minimum 38×38 pt @1x icons required for proper display in iOS 15 notification banners.

### Redesigned Apple Pay Experience (SwiftUI)
The Apple Pay payment sheet is rebuilt from the ground up in SwiftUI with a new design. Key improvements: simplified card and address addition for new users; existing users can add another card without leaving the payment flow; redesigned error handling; new summary view showing payment items, discounts, and subtotals; app icon (and web clip icon for websites) displayed on the summary screen.

**Web clip icon**: Place 2x and 3x icon files in the root document folder — Apple Pay retrieves and displays them automatically.

### New JavaScript Apple Pay Button
A new JavaScript web component (`apple-pay-button`) replaces the older image-based button. Supports all current button types and styles, easily customizable via CSS properties (prefixed `apple-pay`). Includes the new "Continue with Apple Pay" button type.

### Shipping Date Ranges (PKShippingMethod)
`PKShippingMethod.dateComponentsRange` **[NEW]** — a `PKDateComponentsRange` specifying start and end `DateComponents`. Uses calendar and time zone to display the correct estimated delivery window or pickup time. Also available in Apple Pay on the Web JavaScript as a date components range dictionary.

### Read-Only Shipping Address
Set a `PKContact` on `PKPaymentRequest.shippingContact` and set `shippingContactEditingMode = .storePickup` (or `.redirectToStore`) with `requiredShippingContactFields` to present a non-editable pickup address to the user. Also available in JavaScript.

### Coupon Code Support
`PKPaymentRequest.supportsCouponCode = true` **[NEW]** — adds a coupon entry field to the payment sheet. `PKPaymentRequest.couponCode` **[NEW]** — pre-populates the field. New delegate method `paymentAuthorizationController(_:didChangeCouponCode:handler:)` **[NEW]** — called whenever the user modifies the coupon code; return updated `PKPaymentSummaryItems` and optional errors. Convenience error initializers: `PKPaymentRequest.paymentCouponCodeInvalidError(localizedDescription:)` and `PKPaymentRequest.paymentCouponCodeExpiredError(localizedDescription:)` **[NEW]**.

### Payment Summary Flexibility
Payment total line now supports additional text: a date for pre-orders or a frequency label for recurring payments.

## APIs & Frameworks

- `PassKit` framework — `PKPassLibrary`, `PKPaymentRequest`, `PKPaymentAuthorizationController`
- `PKShippingMethod.dateComponentsRange` **[NEW]** — `PKDateComponentsRange` with start/end `DateComponents`
- `PKDateComponentsRange` **[NEW]** — encapsulates a start and end `DateComponents` for shipping windows
- `PKPaymentRequest.supportsCouponCode: Bool` **[NEW]** — enables coupon code entry in the sheet
- `PKPaymentRequest.couponCode: String?` **[NEW]** — pre-populates the coupon code field
- `PKPaymentAuthorizationControllerDelegate.paymentAuthorizationController(_:didChangeCouponCode:handler:)` **[NEW]** — coupon code change callback
- `PKPaymentRequestCouponCodeUpdate` **[NEW]** — result type for coupon code handler; carries `summaryItems` and `errors`
- `PKPaymentRequest.paymentCouponCodeInvalidError(localizedDescription:)` **[NEW]** — convenience invalid coupon error
- `PKPaymentRequest.paymentCouponCodeExpiredError(localizedDescription:)` **[NEW]** — convenience expired coupon error
- `PKPaymentRequest.shippingContact: PKContact` — set for read-only pickup address display
- `PKPaymentRequest.shippingContactEditingMode` — `.storePickup`, `.redirectToStore`, `.enabled` **[NEW values]**
- `.pkpasses` file extension / MIME type — multi-pass bundle for web downloads
- Apple Pay JavaScript `apple-pay-button` web component **[NEW]** — new button element
- Apple Pay JavaScript `dateComponentsRange` in shipping method **[NEW]**
- Apple Pay JavaScript `shippingContactEditingMode` **[NEW]**

## Code Highlights

Shipping method with date range:
```swift
var method = PKShippingMethod(label: "Standard Shipping", amount: NSDecimalNumber(string: "0.00"))
method.identifier = "standard"
let calendar = Calendar.current
let start = calendar.dateComponents([.calendar, .year, .month, .day],
                                     from: calendar.date(byAdding: .day, value: 3, to: Date())!)
let end = calendar.dateComponents([.calendar, .year, .month, .day],
                                   from: calendar.date(byAdding: .day, value: 5, to: Date())!)
method.dateComponentsRange = PKDateComponentsRange(start: start, end: end)
```

Enabling coupon codes:
```swift
request.supportsCouponCode = true
request.couponCode = "FESTIVAL"  // pre-populate if known
```

Coupon code delegate callback:
```swift
func paymentAuthorizationController(_ controller: PKPaymentAuthorizationController,
                                     didChangeCouponCode couponCode: String,
                                     handler completion: @escaping (PKPaymentRequestCouponCodeUpdate) -> Void) {
    guard !couponCode.isEmpty else {
        completion(PKPaymentRequestCouponCodeUpdate(summaryItems: originalItems))
        return
    }
    if isValid(couponCode) {
        completion(PKPaymentRequestCouponCodeUpdate(summaryItems: applyDiscount(to: originalItems)))
    } else {
        let error = PKPaymentRequest.paymentCouponCodeInvalidError(localizedDescription: "Code not recognized")
        completion(PKPaymentRequestCouponCodeUpdate(errors: [error], summaryItems: originalItems,
                                                    shippingMethods: shippingMethods))
    }
}
```

Multi-pass bundle (web server):
```
Content-Type: application/vnd.apple.pkpasses
File extension: .pkpasses  (zip of .pkpass files)
```

## Takeaways

- The redesigned SwiftUI Apple Pay sheet upgrades automatically — no code changes needed — but new icon and summary enhancements require small additions.
- Shipping date ranges and read-only pickup addresses significantly improve in-store and time-sensitive purchase flows with minimal API surface.
- Coupon code support eliminates a major conversion-killer: users no longer need to abandon the Apple Pay flow to apply a discount.
- The new JavaScript `apple-pay-button` web component simplifies web integration and should replace legacy image-based buttons.

---
_Source: WWDC21 Session 10092 page (abstract, chapter summaries, code samples, and resource links)._
