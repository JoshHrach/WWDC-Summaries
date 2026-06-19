# Explore SMS message filters
**WWDC22 · Session 110341** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110341/)

_Platforms:_ iOS 16

## Overview
SMS message filter extensions allow third-party apps to automatically categorize incoming SMS messages from unknown senders into system-provided folders (Transactions, Promotions, Junk) without exposing message content to the extension's developer server. iOS 16 expands the classification API with 12 new sub-categories under Transactions and Promotions, enabling far more granular inbox organization.

The session covers the two-phase lifecycle of SMS filter extensions — a configuration phase (new in iOS 16) where the extension declares its supported sub-categories, and a runtime classification phase where each incoming message is evaluated and placed into the appropriate folder or sub-folder.

## Key Topics

**SMS filter background** — Many markets receive high volumes of business SMS messages (bank alerts, promotions, OTPs). iOS has provided a sandbox extension model since iOS 14, creating Transactions, Promotions, and Junk top-level folders. Previously, classification was limited to these three top-level categories.

**New iOS 16 sub-categories** — 12 new sub-categories for more granular classification: under Transactions: `.transactionalFinance`, `.transactionalOrders`, `.transactionalReminders`, `.transactionalHealth`, `.transactionalPublicServices`, `.transactionalWeather`; under Promotions: `.promotionalCoupons`, `.promotionalOffers`, `.promotionalRewards`, `.promotionalGenericPromotion` and additional promotional sub-types. An extension can declare up to 5 sub-actions.

**Configuration phase (new in iOS 16)** — When a user selects a filter in Settings > Messages > Unknown & Spam, iOS calls the new `handle(_:context:completion:)` method for `ILMessageFilterCapabilitiesQueryRequest`. The extension returns an `ILMessageFilterCapabilitiesQueryResponse` listing up to 5 `transactionalSubActions` and/or `promotionalSubActions`. iOS then creates the corresponding sub-folders in the Messages inbox.

**Runtime classification phase** — For each incoming SMS from an unknown sender, iOS calls `handle(_:context:completion:)` for `ILMessageFilterQueryRequest`. The extension inspects `messageBody` and returns an `ILMessageFilterQueryResponse` with `filterAction` (`.transaction`, `.promotion`, `.junk`) and an optional `filterSubAction`. If an invalid combination is returned (e.g., action = `.transaction` with sub-action = `.promotionalCoupons`), iOS ignores the sub-action and uses only the top-level action.

**Privacy model** — All filtering runs in a sandboxed extension process. Message content never leaves the device unless the extension explicitly makes a network request. Apple's own SMS filter for India is built on these same APIs and has been updated to use the new sub-categories.

## APIs & Frameworks

### IdentityLookup
- `ILMessageFilterExtension` — base class for the extension
- `ILMessageFilterCapabilitiesQueryRequest` **[NEW in iOS 16]** — request object for the configuration phase
- `ILMessageFilterCapabilitiesQueryResponse` **[NEW in iOS 16]** — response for declaring supported sub-categories
- `ILMessageFilterCapabilitiesQueryResponse.transactionalSubActions` **[NEW]** — up to 5 `ILMessageFilterSubAction` values
- `ILMessageFilterCapabilitiesQueryResponse.promotionalSubActions` **[NEW]** — up to 5 `ILMessageFilterSubAction` values
- `ILMessageFilterQueryRequest` — runtime request for classifying an incoming message
- `ILMessageFilterQueryRequest.messageBody` — the SMS text (used for classification logic)
- `ILMessageFilterQueryResponse` — runtime classification response
- `ILMessageFilterQueryResponse.filterAction` — `.transaction`, `.promotion`, `.junk`, `.none`
- `ILMessageFilterQueryResponse.filterSubAction` **[NEW]** — `ILMessageFilterSubAction` value
- `ILMessageFilterAction` — existing enum: `.none`, `.allow`, `.junk`, `.transaction`, `.promotion`
- `ILMessageFilterSubAction` **[NEW]** — new enum with sub-category values:
  - Transactional: `.transactionalFinance`, `.transactionalOrders`, `.transactionalReminders`, `.transactionalHealth`, `.transactionalPublicServices`, `.transactionalWeather`
  - Promotional: `.promotionalCoupons`, `.promotionalOffers`, `.promotionalRewards`, `.promotionalGenericPromotion`
- `ILMessageFilterExtensionContext` — context object passed to handler methods

### Xcode
- Message Filter Extension target template — auto-populates `MessageFilterExtension.swift` with required handler stubs

## Code Highlights

Declaring supported sub-categories (configuration phase):
```swift
func handle(_ capabilitiesRequest: ILMessageFilterCapabilitiesQueryRequest,
            context: ILMessageFilterExtensionContext,
            completion: @escaping (ILMessageFilterCapabilitiesQueryResponse) -> Void) {
    let response = ILMessageFilterCapabilitiesQueryResponse()
    response.transactionalSubActions = [.transactionalFinance,
                                        .transactionalOrders,
                                        .transactionalHealth]
    response.promotionalSubActions   = [.promotionalCoupons,
                                        .promotionalOffers]
    completion(response)
}
```

Classifying an incoming message at runtime:
```swift
func handle(_ queryRequest: ILMessageFilterQueryRequest,
            context: ILMessageFilterExtensionContext,
            completion: @escaping (ILMessageFilterQueryResponse) -> Void) {
    guard let message = queryRequest.messageBody else { return }
    let response = ILMessageFilterQueryResponse()
    switch message {
    case _ where message.contains("debited"):
        response.filterAction    = .transaction
        response.filterSubAction = .transactionalFinance
    case _ where message.contains("coupon"):
        response.filterAction    = .promotion
        response.filterSubAction = .promotionalCoupons
    default: break
    }
    completion(response)
}
```

## Takeaways
- iOS 16 adds a new configuration phase via `ILMessageFilterCapabilitiesQueryRequest` — declare up to 5 sub-categories before any messages arrive; iOS creates the sub-folders immediately upon activation.
- The `filterSubAction` value must be consistent with the top-level `filterAction`; mismatched pairs cause iOS to silently drop the sub-action and use only the top-level category.
- All classification runs on-device in a sandboxed extension; message content can only leave the device if your extension code explicitly makes a network call.
- Apple's own SMS filter for India has been updated to use the new sub-categories, validating the API for high-volume markets.

---
_Source: WWDC22 Session 110341 page (abstract, chapter summaries, code samples, and resource links)._
