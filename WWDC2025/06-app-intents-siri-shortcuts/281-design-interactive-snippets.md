# Design Interactive Snippets
**WWDC25 · Session 281** · [Watch](https://developer.apple.com/videos/play/wwdc2025/281/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, watchOS 26

## Overview
Snippets are compact views driven by App Intents that appear in Spotlight, Siri, and the Shortcuts app, floating above other content so users never have to leave their current context. This session covers how to design snippets that are glanceable, interactive, and appropriately scoped — including the new capability for snippets to contain buttons and display live updated data.

The session walks through appearance principles (type size, layout, contrast), interaction patterns (stateful buttons, animated data updates), and the two core snippet types — results and confirmations — with guidance on when to use each.

## Key Topics

### Appearance
- Snippets use **larger-than-default type sizes** to draw attention to key information at a glance.
- Consistent margins are critical; use `ContainerRelativeShape` to ensure margins adapt correctly across platforms and screen sizes.
- Keep content under 340 points tall to avoid unexpected scroll behavior — link to a full app view for richer content.
- Vibrant backgrounds tied to app identity are encouraged but must maintain strong contrast, especially when viewed at a distance.

### Interaction
- **[NEW]** Snippets now support interactive elements: buttons that trigger actions and data that updates in-place with a scale-and-blur animation.
- Updating data within the snippet (rather than dismissing it) builds user trust in the App Intent for repeated use.
- Snippets can contain multiple buttons and multiple updatable data regions simultaneously.
- Even non-interactive snippets can animate in fresh data automatically.

### Snippet Types
- **Result snippets** — show an outcome that requires no further action; the only system button is "Done." Best for order status, confirmation receipts, etc.
- **Confirmation snippets** — require a user action (e.g., "Order") before a result is shown; action-verb buttons are customizable from a set of pre-written options or a developer-supplied string. Always display a result snippet after a confirmation is completed to close the loop.

### Dialog
- Siri dialog (spoken text) is essential for voice-first interactions (AirPods, eyes-free).
- Design the snippet to be fully understandable without dialog — treat dialog as supplementary, not load-bearing.

## APIs & Frameworks

### App Intents
- `AppIntent` — base protocol for all intents; snippets are the visual output of intent execution. **[NEW interactive capabilities]**
- `ContainerRelativeShape` — **[NEW]** use in snippet layouts to get responsive, platform-adaptive margins.
- `Displaying static and interactive snippets` — updated documentation covering the new interactive snippet APIs.

### SwiftUI (snippet views)
- Interactive snippet views are built with SwiftUI.
- `Button` inside snippet views — **[NEW]** buttons trigger App Intent actions and animate data updates in-place.

## Code Highlights
No explicit code samples were shown in this design-focused session. The referenced documentation page "Displaying static and interactive snippets" contains implementation details.

## Takeaways
- Keep snippets under 340 points tall and link out to the app for deeper content rather than scrolling inside the snippet.
- Add buttons and live-updating data to turn routine intents into genuinely interactive moments — this is the key differentiator of interactive snippets.
- Choose confirmation type when the intent requires user consent before executing; use result type when the outcome is already known.
- Design snippets to communicate their purpose without relying on Siri dialog, since many users run intents in non-voice contexts.

---
_Source: WWDC25 Session 281 page (abstract, chapter summaries, and full transcript)._
