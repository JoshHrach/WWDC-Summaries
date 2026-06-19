# What's New in Apple Pay
**WWDC25 · Session 201** · [Watch](https://developer.apple.com/videos/play/wwdc2025/201/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, watchOS 26

## Overview
This session covers three areas of improvement: a redesigned dynamic Apple Pay button, a unified preauthorized payments experience with rich merchant branding, and new Automatic Order Tracking in Wallet powered by Apple Intelligence. It also introduces background delivery for FinanceKit, enabling apps to process financial data changes even when not running, and announces FinanceKit's expansion to the UK market via the Open Banking Standard.

## Key Topics

### Dynamic Apple Pay Button
The Apple Pay button now displays the user's default payment method with card art, making checkout more engaging. Merchants should provide a `Merchant Category Code` (MCC) and `supportedNetworks`/`merchantCapabilities` on the `PKPaymentRequest` passed to the button's initializer. If the default card is unsupported, the next available card is shown. SwiftUI and UIKit apps get the new button for free — no code changes required. A new view modifier (`payWithApplePayButtonDisableCardArt`) opts back to the original static button.

### Preauthorized Payments with Rich Merchant Information
A new unified Preauthorized Payments view in Wallet (accessible via the More menu on the card details view) shows all ongoing preauthorized payments with per-merchant detail pages. Merchants can now provide:
- An icon (set via Apple Business Connect)
- A custom merchant name, description, and product images per payment
- Upcoming and past payment details with line items

Rich information is vended via a Merchant Token Usage Information bundle (a zip file, max 5 MB) served from a merchant-hosted bundle-vending endpoint. The bundle is end-to-end encrypted using HPKE in auth mode with a Merchant Public/Private key pair and a Merchant Token Public Key from Apple's server. Users receive push notifications about upcoming payments.

### Automatic Order Tracking
Using Apple Intelligence, Wallet now detects order confirmation emails in Mail, automatically parses and imports them into Wallet orders, and links related carrier emails. Merchants should ensure emails include: merchant name in the body, order number in every email, and a tracking number to link carrier emails. Apple Business Connect registration (for logo, merchant name, email addresses) improves the parsed result. Optionally, attach a full order bundle or provide a `webServiceURL` on the payment request for the complete feature set (in-app receipts, returns, deep links).

### FinanceKit Background Delivery
A new background delivery extension type allows apps to process financial data changes (accounts, account balances, transactions) even when the app is not running. The app registers for specific data types and update frequencies (hourly, daily, weekly); the extension is woken when data changes. Two endpoints: `didReceiveData` (processes changed data types, must finish within a time window) and `willTerminate` (gracefully saves in-progress work). Enables use cases like real-time widget updates and on-device spending reports.

FinanceKit is now also available in the UK via the Open Banking Standard (Connected Cards), expanding the addressable market for finance apps.

## APIs & Frameworks

**PassKit (Apple Pay)**
- `PKPaymentButton` dynamic card art display **[NEW]** — automatic; requires `PKPaymentRequest` with `merchantCategoryCode`
- `PKPaymentRequest.merchantCategoryCode` **[NEW]** — industry-standard MCC code property; controls dynamic button display
- `payWithApplePayButtonDisableCardArt` (SwiftUI view modifier) **[NEW]** — reverts to static button appearance
- Merchant Token Usage Information bundle schema **[NEW]** — zip file containing usage info JSON, merchant logo, product images; served from bundle-vending endpoint; end-to-end encrypted
- `Merchant Token Usage Information Package Availability Notification` **[NEW]** — sent to Apple Pay server to register bundle endpoint
- `merchantTokenIdentifier`, `merchantName`, `merchantLogoName`, `upcomingPayments`, `pastPayments` — usage info JSON fields **[NEW]**

**Wallet (Order Tracking)**
- Automatic Order Tracking **[NEW]** — Apple Intelligence detects order emails in Mail and creates Wallet orders automatically
- `webServiceURL` on payment request — existing; enables full order tracking features

**Apple Business Connect**
- Logo and merchant name registration — used in preauthorized payments, order tracking, and other Apple system surfaces

**FinanceKit**
- Background delivery extension **[NEW]** — new extension type; registers with `FinanceStore` for background wakeup
- `didReceiveData([BackgroundDataType])` **[NEW]** — extension entry point; receives changed data types (`.accounts`, `.accountBalances`, `.transactions`)
- `willTerminate()` **[NEW]** — called when time window is nearly exhausted; save in-progress work
- Update frequency registration **[NEW]** — hourly, daily, weekly; app calls `FinanceStore` to enable/disable delivery per data type
- FinanceKit API now available in the UK **[NEW]** — via Open Banking Standard / Connected Cards

## Code Highlights

```swift
// Dynamic Apple Pay button with MCC
let request = PKPaymentRequest()
request.supportedNetworks = [.visa, .masterCard]
request.merchantCapabilities = [.credit, .debit]
request.merchantCategoryCode = PKMerchantCategoryCode(rawValue: "5995") // Pet Shops

// SwiftUI: opt out of card art
ApplePayButton(.buy, action: { ... })
    .payWithApplePayButtonDisableCardArt()
```

```swift
// Background delivery extension
struct SpendingExtension: FinanceKitExtension {
    func didReceiveData(_ dataTypes: [BackgroundDataType]) async {
        let transactions = try await financeStore.transactions(since: startOfWeek)
        let total = transactions.reduce(0) { $0 + $1.transactionAmount.amount }
        store.weeklySpend = total
        WidgetCenter.shared.reloadAllTimelines()
    }
    func willTerminate() {
        store.save()
    }
}
```

```swift
// App registers for background delivery
try await financeStore.enableBackgroundDelivery(for: .transactions, frequency: .hourly)
try await financeStore.enableBackgroundDelivery(for: .accounts, frequency: .daily)
```

## Takeaways
- Pass a `merchantCategoryCode` and `supportedNetworks` on every `PKPaymentRequest` so the dynamic Apple Pay button can display the best card — SwiftUI/UIKit apps get the visual update for free.
- Implement the Merchant Token Usage Information bundle endpoint to provide rich branding and payment details in the Wallet preauthorized payments view.
- Register email addresses and logo in Apple Business Connect to improve Automatic Order Tracking results without any code changes.
- Add a FinanceKit background delivery extension to power widgets and on-device reports without requiring the app to be active.

---
_Source: WWDC25 Session 201 page (abstract, chapter summaries, code samples, and resource links)._
