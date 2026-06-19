# What's new with in-app purchase
**WWDC22 · Session 10007** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10007/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9

## Overview
Building on the StoreKit 2 foundation introduced at WWDC21, this session covers a wave of enhancements across both on-device StoreKit APIs and the server-side App Store Server API / App Store Server Notifications V2. On the client side, the new App Transaction API provides signed, JWS-verified proof of app purchase — replacing the app-detail portion of the original receipt — and is useful for scenarios like transitioning a paid app to free-with-IAP while preserving entitlements for early buyers.

Three new properties are added to StoreKit models: `priceLocale` on `Product` (for formatting derived price values), `environment` on `Transaction` and `RenewalInfo` (indicating Xcode/sandbox/production origin), and `recentSubscriptionStartDate` on subscription info (tracking continuous loyalty periods). All are backported to last year's OS versions when built with Xcode 14. New SwiftUI APIs arrive for offer code redemption (a new view modifier) and review prompts (an `@Environment` action). The new StoreKit Messages API lets apps defer or control display of App Store-vended sheets such as price increase consent.

On the server, major additions include query parameters for filtering and sorting the Get Transaction History endpoint, a Request a Test Notification endpoint to verify server URL reachability, a companion Get Test Notification Status endpoint to inspect delivery results, and the entirely new Get Notification History endpoint for replaying up to six months of V2 notifications — enabling recovery from server outages.

## Key Topics

### App Transaction API
`AppTransaction.shared` returns a `VerificationResult<AppTransaction>` containing a JWS-signed payload with `originalAppVersion` (the app version at first download). Apps transitioning from paid to freemium use `originalAppVersion` to detect existing customers and grant legacy entitlements without requiring a new in-app purchase.

### New StoreKit Model Properties
- **`priceLocale`** on `Product` — enables correctly formatted arithmetic on `product.price` (e.g., deriving monthly cost from a yearly subscription price).
- **`environment`** on `Transaction` / `RenewalInfo` — distinguishes Xcode, sandbox, and production; use to filter test transactions before sending to analytics servers.
- **`recentSubscriptionStartDate`** on `Product.SubscriptionInfo.RenewalInfo` — date of the most recent continuous subscription period (gaps ≤ 60 days are ignored); helps identify loyal customers for targeted offers.

Sentinel values apply only when using StoreKit Testing in Xcode on older OS simulators: `xx_XX` locale, empty string, and `Date.distantPast` respectively.

### SwiftUI Offer Code & Review APIs
`offerCodeRedemption(isPresented:onCompletion:)` view modifier presents the offer code redemption sheet. `@Environment(\.requestReview)` provides a `RequestReviewAction` callable as a function; the system limits prompts to three per 365-day window.

### StoreKit Messages
`Message.messages` is an async sequence on the `Message` type. Apps that set up a `for await message in Message.messages` listener receive pending App Store messages (e.g., price increase consent sheets) before they are displayed, allowing deferred presentation. `@Environment(\.displayStoreKitMessage)` exposes `DisplayMessageAction` to present a deferred message at the right moment.

### App Store Server API Enhancements
Get Transaction History gains new query parameters: `sort` (`ASCENDING`/`DESCENDING` by modified date), `productType`, `productId`, `subscriptionGroupIdentifier`, `startDate`, `endDate`, `excludeRevoked`, `inAppOwnershipType`, `revoked`, and `familyShared`. Three fields (`productType`, `productId`, `subscriptionGroupIdentifier`) support multiple values.

### App Store Server Notifications V2 — Test & History
- **Request a Test Notification** (`POST /inApps/v1/notifications/test`) sends a `TEST` notification to the registered URL; response includes `testNotificationToken`.
- **Get Test Notification Status** (`GET /inApps/v1/notifications/test/{testNotificationToken}`) returns `signedPayload` and `firstSendAttemptResult` (SUCCESS or error code).
- **Get Notification History** (`POST /inApps/v1/notifications/history`) fetches up to six months of V2 notifications by date range; optional filters by `notificationType`, `notificationSubtype`, or `originalTransactionId`; paginated with `paginationToken`.

### applicationUsername → appAccountToken Migration
When the original StoreKit API's `SKPayment.applicationUsername` is set to a UUID string, the App Store server now persists it as `appAccountToken` in V2 signed transactions, enabling seamless migration to modern StoreKit 2 APIs.

## APIs & Frameworks

**StoreKit 2 — App Transaction** **[NEW]**
- `AppTransaction` **[NEW]** — signed app purchase record
- `AppTransaction.shared` **[NEW]** — async property returning `VerificationResult<AppTransaction>`
- `AppTransaction.originalAppVersion` **[NEW]** — version string at first download
- `AppTransaction.refresh()` **[NEW]** — prompts user to re-verify (use sparingly)

**StoreKit 2 — New Model Properties** **[NEW]**
- `Product.priceLocale: Locale` **[NEW]** — locale for price formatting
- `Transaction.environment: AppStore.Environment` **[NEW]** — `.xcode`, `.sandbox`, `.production`
- `RenewalInfo.environment: AppStore.Environment` **[NEW]** — same as above
- `Product.SubscriptionInfo.RenewalInfo.recentSubscriptionStartDate: Date` **[NEW]**
- `AppStore.Environment` **[NEW]** — enum with `.xcode`, `.sandbox`, `.production`

**StoreKit 2 — SwiftUI Views / Environment** **[NEW]**
- `.offerCodeRedemption(isPresented:onCompletion:)` **[NEW]** — view modifier (iOS 16+)
- `@Environment(\.requestReview) var requestReview: RequestReviewAction` **[NEW]**
- `RequestReviewAction` **[NEW]** — callable as function to request review prompt

**StoreKit 2 — Messages** **[NEW]**
- `Message` **[NEW]** — struct representing an App Store message
- `Message.Reason` **[NEW]** — `.priceIncreaseConsent`, etc.
- `Message.messages: AsyncSequence` **[NEW]** — async sequence of pending messages
- `@Environment(\.displayStoreKitMessage) var displayStoreKitMessage: DisplayMessageAction` **[NEW]**
- `DisplayMessageAction` **[NEW]** — callable with `Message` to present sheet

**App Store Server API** **[NEW/Enhanced]**
- Get Transaction History — new query params: `sort`, `productType`, `productId`, `subscriptionGroupIdentifier`, `startDate`, `endDate`, `excludeRevoked`, `inAppOwnershipType`, `revoked`, `familyShared` **[NEW]**
- `recentSubscriptionStartDate` field in renewal info payload **[NEW]**
- `environment` field in transaction and renewal info payloads **[NEW]**
- Request a Test Notification (`POST /inApps/v1/notifications/test`) **[NEW]**
- Get Test Notification Status (`GET /inApps/v1/notifications/test/{testNotificationToken}`) **[NEW]**
- Get Notification History (`POST /inApps/v1/notifications/history`) **[NEW]**
  - `firstSendAttemptResult` field **[NEW]**
  - `paginationToken` pagination **[NEW]**

## Code Highlights

Using App Transaction to detect legacy paid-app customers:
```swift
let result = try await AppTransaction.shared
switch result {
case .unverified:
    // alert user, show minimal experience
case .verified(let appTransaction):
    let isPaidCustomer = didPurchaseBefore(version: "8.0",
                         originalVersion: appTransaction.originalAppVersion)
    if isPaidCustomer {
        unlockLegacyContent()
    } else {
        checkCurrentEntitlements()
    }
}
```

Subscribing to StoreKit Messages to defer price consent sheet:
```swift
// In DonutEditorView:
.task {
    for await message in Message.messages {
        if !pendingMessages.contains(where: { $0.id == message.id }) {
            pendingMessages.append(message)
        }
    }
}
// In parent view, on dismiss:
@Environment(\.displayStoreKitMessage) var displayStoreKitMessage
for message in pendingMessages {
    await displayStoreKitMessage(message)
}
```

Offer code redemption sheet:
```swift
@State var isShowingOfferCodeSheet = false
Button("Redeem Offer Code") { isShowingOfferCodeSheet = true }
    .offerCodeRedemption(isPresented: $isShowingOfferCodeSheet) { result in
        // handle result
    }
```

## Takeaways
- `AppTransaction.shared` provides JWS-signed proof of app purchase; use `originalAppVersion` to grant legacy entitlements when switching from a paid to freemium model.
- Three new backported properties — `priceLocale`, `environment`, and `recentSubscriptionStartDate` — give richer context to StoreKit transactions and subscription data with no OS update required (Xcode 14 build is enough).
- The StoreKit Messages API lets apps control the timing of App Store-vended sheets (e.g., price consent) so they do not interrupt critical user flows.
- The new Get Notification History endpoint covers up to six months of V2 server notifications, enabling full recovery after server outages without relying solely on the retry system.

---
_Source: WWDC22 Session 10007 page (abstract, chapter summaries, full transcript, and resource links)._
