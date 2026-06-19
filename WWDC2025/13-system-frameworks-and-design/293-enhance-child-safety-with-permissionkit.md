# Enhance Child Safety with PermissionKit
**WWDC25 · Session 293** · [Watch](https://developer.apple.com/videos/play/wwdc2025/293/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26

## Overview
PermissionKit is a new framework that enables apps to integrate with Apple's Communication Limits system — the parental-control mechanism that parents configure in Screen Time to restrict who children can communicate with. With PermissionKit, apps can query which contacts are allowed for a child, request permission for new contacts, display a system-provided UI for asking parents, and handle asynchronous parent decisions — all without ever accessing private contact or parental-control data directly.

The framework is designed for apps in the communication, gaming, and social categories where child-to-adult contacts need to be controlled.

## Key Topics

### Communication Limits
- Communication Limits is a Screen Time feature (iOS 13.3+) where parents specify which handles (phone numbers, email addresses, game center IDs) a child can contact.
- Before PermissionKit, apps had no programmatic way to check or interact with these limits.
- PermissionKit gives apps a privacy-preserving read path and a parent-approval request flow.

### Querying Allowed Contacts
- `CommunicationLimits.knownHandles(in:)` — **[NEW]** takes an array of `CommunicationHandle` objects (phone, email, game center) and returns the subset that the child's Communication Limits allow.
- The app never learns which handles are blocked or why — it only gets the allowed subset.
- Works even when Screen Time is not configured (returns all provided handles in that case, so no special-casing required).

### Requesting Parent Permission
- When a child wants to contact someone outside their allowed set, apps call `PermissionQuestion` to initiate a parent-approval request. **[NEW]**
- `PermissionQuestion` takes a `CommunicationTopic` (describing the context, e.g., "Start a game") and a `PersonInformation` (display name, avatar) for the proposed contact.
- The system presents a `CommunicationLimitsButton` — a standard UI element that triggers the request and handles all parent communication. **[NEW]**
- The parent receives a Screen Time notification and can approve or deny from any of their devices.
- The app receives the decision asynchronously via an async sequence on `CommunicationLimits`.

### DeclaredAgeRange API
- **[NEW]** `DeclaredAgeRange` — apps can declare an age range for the current user (child, teen, adult) in response to user-provided information or app-level age gating.
- Separate from Screen Time; allows apps to self-categorize users for appropriate content and feature access even when Screen Time is not enabled.
- Requires user consent; displayed in Privacy settings.

### Privacy Design
- PermissionKit never exposes Screen Time configuration details to the app.
- The system handles all parent communication; the app only sees a binary allow/deny outcome.
- No entitlement is required for basic Communication Limits queries — only for sending parent requests.

## APIs & Frameworks

### PermissionKit (all NEW)
- `PermissionKit` framework
- `CommunicationLimits` — main namespace; provides `knownHandles(in:)` and async updates sequence.
- `CommunicationLimits.knownHandles(in:)` — returns allowed handles from a candidate set.
- `CommunicationHandle` — represents a phone number, email address, or Game Center identifier.
- `PermissionQuestion` — encapsulates a parent-approval request.
- `CommunicationTopic` — describes the communication context for the parent's review.
- `PersonInformation` — display name and optional avatar for the proposed contact.
- `CommunicationLimitsButton` — SwiftUI view that triggers and monitors parent-approval requests.
- `DeclaredAgeRange` — app-declared age range for the current user.

## Code Highlights

```swift
// Query which candidate handles are permitted
let candidates = contacts.map { CommunicationHandle(phoneNumber: $0.phone) }
let allowed = try await CommunicationLimits.knownHandles(in: candidates)

// Show a permission request button for a blocked contact
CommunicationLimitsButton(
    question: PermissionQuestion(
        topic: CommunicationTopic("Start a game"),
        person: PersonInformation(displayName: "Alex")
    )
) {
    // Called when parent approves
    startGame(with: contact)
}
```

## Takeaways
- Integrate `CommunicationLimits.knownHandles(in:)` at the point where your app presents a contact picker or friend invitation flow — filter the list before presenting it to the child.
- Use `CommunicationLimitsButton` for the parent-request UI; do not build a custom flow as parents expect the standard system experience.
- Handle the async response gracefully — parent approval may come minutes or hours later; use async sequences to react when a decision arrives.
- Consider adopting `DeclaredAgeRange` if your app has its own age-verification or age-gating flow to provide appropriate default settings.

---
_Source: WWDC25 Session 293 page (abstract, chapter summaries, and resource links)._
