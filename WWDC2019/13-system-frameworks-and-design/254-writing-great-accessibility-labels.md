# Writing Great Accessibility Labels
**WWDC19 · Session 254** · [Watch](https://developer.apple.com/videos/play/wwdc2019/254/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
Accessibility labels are the primary mechanism by which assistive technologies — VoiceOver, Speak Screen, and Voice Control — convey the meaning of UI elements to users who cannot see them. This session, delivered by an Apple Accessibility engineer who is a VoiceOver user herself, demonstrates through live VoiceOver demos the real impact that missing, redundant, or poorly written labels have on usability, and establishes a concise set of best practices that every developer should follow.

The session is context-first: the right label for any element depends entirely on the surrounding UI, the app's purpose, and the number of similar controls on screen. There are no new APIs introduced; the session focuses entirely on how to use the existing `accessibilityLabel` property effectively.

## Key Topics

### What an Accessibility Label Is
- An accessibility label is a localized string that succinctly identifies an accessibility element.
- It is human-readable and gives context and meaning — not a technical description of the element's type or image file name.
- Setting or reading one in code is as simple as assigning or reading a `String`; the difficulty is in writing the right string.

### Context Determines the Label
- The same `+` button demands different labels depending on where it appears:
  - In a notes app nav bar: "Add" is clear.
  - In a shopping app: "Add to Cart" distinguishes it from "Add to Favorites."
  - When dozens of similar buttons exist on the same screen: "Add peanut butter to cart" disambiguates.
- Never write a label in isolation; always consider what the full VoiceOver announcement will sound like at the moment the user focuses on that element.

### Best Practices

**Always add labels.**
- Without a label, VoiceOver reads the image asset name (e.g., "Plus underscore icon underscore outline underscore hash nine nine nine nine dot") or falls back to announcing "button" repeatedly.
- Every interactive and meaningful element must have an explicit label.

**Do not include the element type.**
- VoiceOver automatically appends the element type ("button," "text field," "tab") after the label.
- Writing "Add button" as the label causes VoiceOver to announce "Add button button" — the word "button" appears twice.

**Update labels when UI state changes.**
- A control whose appearance toggles (e.g., an Add button that becomes a Delete button after interaction) must have its `accessibilityLabel` updated to match.
- Failing to update means VoiceOver continues to announce the stale label while the control performs a different action.

**Provide context when multiple similar controls exist.**
- A list of product rows each with an "Add" button must differentiate: "Add peanut butter," "Add bananas," "Add cookies."
- Without context, the user cannot determine which item a given button acts on.

**Avoid redundancy in context-rich screens.**
- In a music player, every control's context (play, next, previous) is already established by the screen; adding "song" to every label is redundant.
- Prefer "Play" over "Play song," "Next" over "Next song."

**Label meaningful animations.**
- Spinners and loading indicators are meaningful UI states; label them so VoiceOver announces "Loading" or "In progress" rather than silence.

**Be succinct — but verbose labels are sometimes correct.**
- "Delete items from the current folder and add to trash" is too verbose; "Delete" is sufficient in context.
- Exception: when verbosity is part of the design intent (e.g., playful sticker packs), rich labels enhance the experience for everyone, sighted or not. The Cookie Monster sticker example — "Me happy face eat small cookie, om nom nom" — is intentional and appropriate.

### Testing Your Labels
- Turn on VoiceOver and swipe through your own app. If a label requires explanation, it needs to be rewritten.
- This is the single most effective way to catch missing or confusing labels before shipping.

## APIs & Frameworks

### UIKit / AppKit / SwiftUI
- `UIAccessibilityElement.accessibilityLabel: String?` — the label VoiceOver reads for any UI element
- `UIView.accessibilityLabel: String?` — available on all `UIView` subclasses
- `UIView.accessibilityValue: String?` — the current value of the element (separate from the label)
- `UIView.accessibilityHint: String?` — optional brief description of what the action does (read after a pause)
- `UIView.accessibilityTraits: UIAccessibilityTraits` — communicates element type/behavior (e.g., `.button`, `.staticText`, `.updatesFrequently`)
- `UIAccessibilityPostNotification(_:_:)` — post accessibility notifications when UI changes (e.g., `UIAccessibility.Notification.layoutChanged` after label updates)
- SwiftUI: `.accessibilityLabel(_:)` modifier — set an explicit accessibility label on any `View`
- SwiftUI: `.accessibilityValue(_:)` modifier — set the value string
- SwiftUI: `.accessibilityHint(_:)` modifier — set the hint string

## Code Highlights

Setting a contextual label on a UIButton in a shopping list:
```swift
// For a row representing a specific product:
func configureCell(for product: Product) {
    addToCartButton.accessibilityLabel = "Add \(product.name) to cart"
}
```

Updating a label when UI state toggles:
```swift
func toggleFavorite() {
    isFavorited.toggle()
    favoriteButton.accessibilityLabel = isFavorited ? "Remove from favorites" : "Add to favorites"
}
```

Labeling a loading spinner:
```swift
activityIndicator.accessibilityLabel = "Loading"
activityIndicator.accessibilityTraits = .updatesFrequently
```

Using SwiftUI accessibility modifiers:
```swift
Button(action: addToCart) {
    Image(systemName: "plus")
}
.accessibilityLabel("Add \(product.name) to cart")
```

## Takeaways
- Missing labels are the single biggest accessibility failure; VoiceOver falls back to raw image file names or meaningless "button" announcements — add an explicit label to every interactive and meaningful element.
- Never include the element type in the label string; VoiceOver appends it automatically, and including it creates double-announcement ("Add button button").
- Context is everything: the correct label depends on the screen, the number of similar controls, and the app's domain. Always evaluate what the full VoiceOver announcement sounds like in situ.
- Update labels immediately when control state or action changes; a stale label describing the wrong action is worse than no label.
- The definitive test: turn on VoiceOver and swipe through your app before shipping.

---
_Source: WWDC19 Session 254 page (abstract, transcript, and resource links)._
