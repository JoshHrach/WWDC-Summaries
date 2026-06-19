# What's new in Wallet and Apple Pay
**WWDC23 · Session 10114** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10114/)

_Platforms:_ iOS 17, iPadOS 17

## Overview
Wallet and Apple Pay in iOS 17 expand across three domains: payments gain Apple Pay Later merchandising views, deferred preauthorized payments, and a brand-new funds transfer (disbursement) API; Order Tracking improves system integration (Maps, Messages, widgets) and introduces FinanceKit/FinanceKitUI APIs for adding orders to Wallet from apps and websites; and a new Tap to Present ID on iPhone API in the ProximityReader framework enables NFC-based mobile driver's license verification without any additional hardware.

## Key Topics

**Payments**

*Apple Pay Later Merchandising*
- New `PKPayLaterView` (UIKit) and `PayLaterView` (SwiftUI) display styles: `.standard`, `.badge`, `.checkout`, `.price`
- Action options: `.learnMore` and `.calculator`
- `PKPayLaterUtilities.validate(amount:locale:)` to check eligibility
- Web support via Apple Pay JavaScript SDK `<apple-pay-merchandising>` element; JWT authentication required
- Entitlement required for apps; domain registration required for web

*Deferred Preauthorized Payments* (new preauthorized payment type)
- `PKDeferredPaymentSummaryItem` **[NEW]** — amount and charge date
- `PKDeferredPaymentRequest` **[NEW]** — billing agreement, free cancellation date/time/timezone, token notification URL
- Works alongside existing recurring and automatic reload preauthorized payment types
- Apple Pay merchant tokens tied to Apple ID (not device), improving token longevity across device upgrades

*Transfer Funds with Apple Pay (Disbursements)*
- Brand new `PKDisbursementRequest` **[NEW]** — sends funds to a card in Wallet (withdrawals, stored value transfers)
- `PKDisbursementSummaryItem` **[NEW]** — final net amount received by recipient's card
- `PKInstantFundsOutFeeSummaryItem` **[NEW]** — fee for instant transfer; required (zero amount if no fee)
- `PKPaymentAuthorizationController.supportsDisbursements(using:capabilities:)` **[NEW]** — eligibility check
- `instantFundsOut` capability for instant transfer cards
- Disbursement errors: `disbursementContactInvalidError`, `disbursementCardUnsupportedError`
- iOS/iPadOS only; not available on macOS or web

**Order Tracking**
- iOS 16.4+: Messages inline order preview, Order Tracking widget (no adoption needed)
- iOS 17: Maps integration — proactive Siri Suggestions for pickup time/location
- `shippingType` property **[NEW]** — `.shipped` or `.delivered`
- Associated application identifiers for better notification management (including enterprise apps)
- Custom product page identifiers for deep-link to relevant App Store page
- Transactions array in order packages **[NEW]** — each with payment method, amount, receipt file (PDF/JPEG/PNG)
- Refund indication in transactions **[NEW]**
- Attach order packages to emails for inline "Add to Wallet" button
- "Track with Apple Wallet" button in apps and websites

*New FinanceKit / FinanceKitUI frameworks* **[NEW]**
- `FinanceStore` (shared instance) — `containsOrder(withIdentifier:)`, `saveOrder(_:)` async
- `AddOrderToWalletButton` (SwiftUI) **[NEW]**
- Web: `<apple-wallet-button type="track-order">` in Apple Pay JS SDK
- Three result states: added, cancelled, newer order exists

**Identity — Tap to Present ID on iPhone**
- New `MobileDocumentReader` API in `ProximityReader` framework **[NEW]**
- Reads IDs in Wallet and other mobile driver's licenses via NFC+Bluetooth; no additional hardware
- `MobileDocumentReader.isSupported` — class property to check device capability
- `MobileDocumentReader().prepare()` → `MobileDocumentReaderSession`
- Display request: `MobileDriversLicenseDisplayRequest(elements:)` — result shown in system UI; no data returned to app
- Data request: `MobileDriversLicenseDataRequest` — returned to app; additional entitlement required
- `MobileDocumentReaderSession.requestDocument(_:)` async
- Reader tokens (JWT signed with Apple Business Register key pair) for branding display
- `MobileDocumentReader.configuration.readerInstanceIdentifier` — fetch instance ID for server-side token generation
- Document elements: `.ageAtLeast(_:)`, `.givenName`, `.familyName`, `.dateOfBirth`, `.portrait`, `.address`, `.documentExpirationDate`, `.drivingPrivileges`
- `retainedElements` vs `nonRetainedElements` for privacy control in data requests

## APIs & Frameworks

**PassKit**
- `PKPayLaterView` **[NEW]** — UIKit merchandising view
- `PKPayLaterView.DisplayStyle` **[NEW]**: `.standard`, `.badge`, `.checkout`, `.price`
- `PKPayLaterView.Action` **[NEW]**: `.learnMore`, `.calculator`
- `PKPayLaterUtilities.validate(amount:locale:)` **[NEW]**
- `PKDeferredPaymentSummaryItem` **[NEW]**
- `PKDeferredPaymentRequest` **[NEW]**
- `PKDeferredPaymentRequest.freeCancellationDate` **[NEW]**
- `PKDeferredPaymentRequest.freeCancellationDateTimeZone` **[NEW]**
- `PKDisbursementRequest` **[NEW]**
- `PKDisbursementSummaryItem` **[NEW]**
- `PKInstantFundsOutFeeSummaryItem` **[NEW]**
- `PKPaymentAuthorizationController.supportsDisbursements(using:capabilities:)` **[NEW]**
- `PKPaymentAuthorizationControllerDelegate.paymentAuthorizationControllerDidFinish(_:)` (existing)
- `PKPaymentAuthorizationControllerDelegate.paymentAuthorizationController(_:didAuthorizePayment:handler:)` (existing, used for disbursements)
- `disbursementContactInvalidError` **[NEW]**
- `disbursementCardUnsupportedError` **[NEW]**

**SwiftUI (PassKit)**
- `PayLaterView` **[NEW]** SwiftUI view
- `.payLaterViewDisplayStyle(_:)` **[NEW]** modifier
- `.payLaterViewAction(_:)` **[NEW]** modifier

**FinanceKit** **[NEW framework]**
- `FinanceStore` — shared instance
- `FinanceStore.containsOrder(withIdentifier:)` async
- `FinanceStore.saveOrder(_:)` async

**FinanceKitUI** **[NEW framework]**
- `AddOrderToWalletButton` **[NEW]** SwiftUI view

**ProximityReader**
- `MobileDocumentReader` **[NEW]**
- `MobileDocumentReader.isSupported` **[NEW]**
- `MobileDocumentReader.configuration.readerInstanceIdentifier` **[NEW]**
- `MobileDocumentReaderSession` **[NEW]**
- `MobileDriversLicenseDisplayRequest` **[NEW]**
- `MobileDriversLicenseDisplayRequest.MobileDriversLicenseElement.ageAtLeast(_:)` **[NEW]**
- `MobileDriversLicenseDataRequest` **[NEW]**
- `MobileDriversLicenseDataRequest.retainedElements` **[NEW]**
- `MobileDriversLicenseDataRequest.nonRetainedElements` **[NEW]**
- `MobileDocumentReaderSession.requestDocument(_:)` async **[NEW]**

## Code Highlights

Tap to Present ID — display request (age verification):
```swift
guard MobileDocumentReader.isSupported else { return }
let reader = MobileDocumentReader()
let readerSession = try await reader.prepare()
let request = MobileDriversLicenseDisplayRequest(elements: [.ageAtLeast(21)])
try await readerSession.requestDocument(request)
```

Tap to Present ID — data request with branding:
```swift
let identifier = try await reader.configuration.readerInstanceIdentifier
let readerToken = try await WebService().fetchToken(for: identifier)
let readerSession = try await reader.prepare(using: .init(readerToken))

var request = MobileDriversLicenseDataRequest()
request.retainedElements = [.givenName, .familyName, .dateOfBirth, .portrait]
request.nonRetainedElements = [.address, .documentExpirationDate]
let response = try await readerSession.requestDocument(request)
self.processResponse(response.documentElements)
```

## Takeaways
- Implement the Apple Pay Later merchandising view with a single `PKPayLaterView` or `PayLaterView` to indicate Buy Now Pay Later support — no other integration required for the payment itself.
- Use `PKDisbursementRequest` to power fund withdrawals, balance transfers, and stored-value payouts with the full security and familiarity of Apple Pay.
- Adopt the `FinanceKit` and `FinanceKitUI` APIs to let users track orders from within your app without navigating away or scanning emails.
- Tap to Present ID on iPhone eliminates dedicated ID-reader hardware for age verification and ID checks — a few lines of ProximityReader code is all that's needed.

---
_Source: WWDC23 Session 10114 page (abstract, chapter summaries, code samples, and resource links)._
