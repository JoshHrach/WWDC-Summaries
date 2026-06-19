# Designing Great Shortcuts
**WWDC19 · Session 806** · [Watch](https://developer.apple.com/videos/play/wwdc2019/806/)

_Platforms:_ iOS 13, iPadOS 13, watchOS 6, macOS Catalina 10.15 (via HomePod/AirPods)

## Overview
This design session covers how to choose, expose, and design Siri Shortcuts effectively. The session walks through the full shortcut design process using the SoupChef sample app as a case study: deciding which app features make good shortcuts, how and where to surface the Add to Siri button in your UI, and how to design a complete voice-interaction flow using the new iOS 13 parameterized shortcuts feature. The session addresses shortcut dialog writing, disambiguation prompts, confirmation prompts, and voice-only (eyes-free) scenarios. It complements the engineering sessions "Building Great Shortcuts" (805) and "Introducing Parameters for Shortcuts" (213).

## Key Topics

- **Choosing what to shortcut** — Not every feature is a good shortcut candidate. Good shortcut features are: valuable and worth repeating, doable without a visual display, and applicable in many contexts (not time-limited). Bad candidates: visually-complex browsing (menus requiring scrolling/tapping), narrowly applicable actions (live order status), rarely-repeated housekeeping (viewing past orders). Ordering food is a strong candidate; browsing a menu is not.
- **Discoverability: Add to Siri button** — Place the Add to Siri button only where there is a clear signal the user might want to repeat an action (e.g., immediately after completing an order — not on every menu item). Don't scatter the button across entire list views. In iOS 13, you can customize the button's corner radius and it automatically respects light/dark mode. **[NEW]**
- **Shortcut phrase suggestions** — iOS 13 allows apps to pre-fill a suggested default phrase in the Add to Siri sheet. Keep suggested phrases short (three words or fewer), ideally a verb + object or proper noun, to minimize recall errors (e.g., "Order Soup" not "Check the bus schedule"). **[NEW]**
- **Parameterized shortcuts in iOS 13** — Users can tap fields in the shortcut to choose how much information to pre-bake vs. leave for Siri to ask at runtime. Design confirmations with the assumption that some fields may be blank and Siri will collect them conversationally. **[NEW]**
- **Conversational design workflow** — Start by listing all information required to complete an action; write scripts covering all conversation paths (including failure/disambiguation branches); converge to a flow diagram covering all states and transitions.
- **Prompt patterns:**
  - **Free-form prompt** — Open-ended question; phrases it so the user knows what format to respond in.
  - **Option list / disambiguation prompt** — Present when input could mean multiple things; use up-front when options are constrained (start with "Which..."). For voice-only, only read out differentiating details, not full option names; provide synonyms for each option so natural phrasing matches.
  - **Parameter confirmation prompt** — Use sparingly, only for high-consequence values; can be used proactively to present a best-guess for the user to accept or override.
  - **Action confirmation prompt** — Required by the system for ordering-category shortcuts; customize the UI to show relevant details; provide separate voice-only dialog since the UI can't be tapped on HomePod/AirPods.
  - **Response** — State outcome concisely ("OK. Ordered."); display rich details in UI; for voice-only, expand dialog to convey the same information verbally.
- **Shortcut categories in Xcode** — Select the category that best matches your shortcut's action (e.g., `order`). The system auto-generates appropriate confirmation question language based on category; don't duplicate a question in your custom dialog.
- **Robustness requirements** — Provide clear error messages for invalid inputs so Siri can re-prompt automatically. Filter out invalid options before presenting lists. Handle context changes (e.g., user's default delivery location becoming far away while traveling — detect and re-prompt for location).
- **Tapping the UI opens the app** — Any custom confirmation or response UI is one big tap target that opens the app; don't draw UI elements that look individually tappable.
- **Dialog writing rules** — Don't be overly polite or inject unnecessary personality (users hear this on every invocation). Don't include your app name (already attributed visually). Don't include the user's name (Siri may already say it on HomePod). Don't use "I" or "we" (Siri isn't performing the action, your app is). Use "here are a few options" instead of "we have." Keep dialog concise and neutral; test by listening to Siri read it multiple times in a row.

## APIs & Frameworks

### SiriKit / Shortcuts **[NEW in iOS 13]**
- `INShortcut` — wraps an intent or NSUserActivity as a shortcut
- `INUIAddVoiceShortcutButton` — pre-built Add to Siri button; customizable corner radius in iOS 13 **[NEW]**
  - `cornerRadius` property — customize to match app design **[NEW]**
  - Automatically adapts to light/dark mode **[NEW]**
- `INVoiceShortcutCenter` — query existing shortcuts, present edit sheet
- `INUIAddVoiceShortcutViewController` / `INUIEditVoiceShortcutViewController` — standard Add to Siri / Edit sheets
- Parameterized intents — define intent parameters that can be left unspecified by the user and collected by Siri at runtime **[NEW]**
- `INIntentResponse` — provide confirmation dialog and response dialog
- Intent category — set in Xcode intent definition file (`.intentdefinition`); e.g., `order`, `search`, `information`; affects system-generated confirmation question phrasing
- Pronunciation hints — `INObject(identifier:displayString:pronunciationHint:)` — provide simplified spoken representation for list items in voice-only scenarios **[NEW]**
- Synonyms for options — defined per parameter option to handle natural language variations in user responses

### System References
- `INDonateShortcutRequest` / `INVoiceShortcut` — shortcut donation and management
- Lock screen and Spotlight suggestion surfaces — driven by NSUserActivity or intent donation

## Code Highlights

Add to Siri button with custom corner radius (iOS 13):

```swift
import IntentsUI

let shortcut = INShortcut(intent: orderSoupIntent)
let button = INUIAddVoiceShortcutButton(style: .whiteOutline)
button.shortcut = shortcut
button.cornerRadius = 12   // [NEW] custom corner radius
button.delegate = self
view.addSubview(button)
```

Suggested default phrase (iOS 13 Add to Siri sheet):

```swift
// Set a suggested invocation phrase on the intent before presenting
orderSoupIntent.suggestedInvocationPhrase = "Order Soup"
let shortcut = INShortcut(intent: orderSoupIntent)
```

Disambiguation option with synonym and pronunciation hint:

```swift
// When building the option list for the "soup" parameter
let beefNoodleSoup = INObject(
    identifier: "beef-noodle",
    displayString: "Beef Noodle Soup",
    pronunciationHint: "Beef")   // what Siri reads aloud to differentiate
beefNoodleSoup.alternativeSpeakableMatches = [
    INSpeakableString(identifier: "beef-noodle", spokenPhrase: "beef",
                      pronunciationHint: nil)
]
```

## Takeaways

- Only shortcut your app's most valuable, repeatable actions — the test is whether a user would genuinely invoke it via voice on a regular basis, not whether it can technically be expressed as an intent.
- Place the Add to Siri button contextually at the moment of highest intent signal (right after a completed order), not scattered across a list of hypothetically repeatable items.
- Keep suggested shortcut phrases to three words or fewer; longer phrases are hard to recall and invite substitution errors that prevent matching.
- Treat shortcut dialog as UI copy — it will be heard repeatedly; every unnecessary word is friction that accumulates over many invocations. Test by listening to Siri speak it five times in a row.

---
_Source: WWDC19 Session 806 page (abstract, full transcript, and resource links)._
