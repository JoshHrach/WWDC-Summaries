# Writing for interfaces
**WWDC22 · Session 10037** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10037/)

_Platforms:_ iOS, iPadOS, macOS, watchOS (design guidance — all platforms)

## Overview
Apple UX Writers Kaely Coon and Jennifer Bush present a practical framework for writing effective interface text across any app or game. The core thesis: writing should be part of the design process from the beginning, not filled in at the end. When words and visuals work together seamlessly, the experience feels intuitive and even invisible. The session centers on a four-concept framework called PACE — Purpose, Anticipation, Context, and Empathy — providing concrete techniques and before/after examples for alerts, onboarding flows, empty states, error messages, and accessibility descriptions.

The session is grounded in Apple's own design history: from the original Mac principle of "what you see is what you get" to the consistent "Hello" greeting across devices, Apple has always treated language as a core design material. Writers and designers share the same goal — helping people do what they want to do — whether that means creating a Memoji, watching a movie, or finding focus.

## Key Topics

### Purpose
Every screen should have one clear purpose. Convey it through information hierarchy: put the most important thing at the top in the largest type, use clear button labels, and know what to leave out. When introducing a feature, state plainly what it does and why it matters. For multi-step flows, define the purpose of the entire flow and each individual screen — this keeps each screen brief and eliminates unnecessary steps. If you cannot articulate a screen's purpose, remove or restructure its content.

### Anticipation
Think of an app as a conversation: it listens, it responds, it asks questions at the right moment. Before writing any screen, ask "what does the user do or think next?" Vary tone to match the situation — celebratory (Activity streak record), calm and clear (Apple Watch hard fall detection), or helpful and direct (Maps commute notification). Develop a consistent voice for your app first (what would it say and not say?), then modulate tone per context. Avoid overusing exclamation points; they lose impact quickly.

### Context
Consider what users are doing when they see your text. Someone exercising outdoors should see a single large button, not a paragraph. A confirmation screen after a workout can include more data. Alerts are interruptions by nature — keep them contextual (triggered at a relevant moment), use clear, specific button labels rather than Yes/No, and make destructive actions visually distinct (red, on the left). Avoid button-label ambiguity by matching action verbs precisely (e.g., "Cancel Subscription" vs. "Keep Subscription" rather than "Cancel" vs. "Confirm"). Empty states are opportunities to educate: tell users what will appear there and how to create content.

### Empathy
Write for everyone. Plain language is most accessible; idioms and humor often do not translate or may exclude readers. Be responsive to localization: text grows 30–50% in many languages, some require taller characters, others read right to left. UI layouts must accommodate all of these. For accessibility, every interactive element, symbol, graph, and image needs descriptive VoiceOver text that conveys both physical details and contextual intent (e.g., "person tilting head to the side with hand beside mouth as if sharing a secret"). Use gender-neutral language by default ("person" rather than "man"/"woman"). Read your writing out loud as a final check — it reveals unnatural phrasing, unnecessary words, and errors.

## APIs & Frameworks

This session contains no code — it is a design and writing guidance session. Key resources referenced:

- [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/) — primary reference for writing, alerts, empty states, accessibility labels
- [Apple Design Resources](https://developer.apple.com/design/resources/) — templates and assets

**Writing Patterns Covered (not APIs but reusable patterns)**
- Alert title + body + specific-verb buttons — alert writing formula
- Destructive action placement — red text, left button position in two-button alerts
- Empty state formula — title stating what's absent + explanation of how content will appear
- Error message formula — title describing the problem + body explaining the fix + action button taking user to resolution
- VoiceOver image description formula — physical description + contextual intent + gender-neutral subject
- Onboarding screen hierarchy — title (purpose) → body (detail) → primary button (action)

## Code Highlights

No code samples. Key writing before/after examples from the session:

**Alert — Destructive action (before):**
> Title: "Confirm Cancellation" | Body: "If you confirm and end this plan now, you'll lose access on June 21, 2022." | Buttons: Cancel / Confirm

**Alert — Destructive action (after):**
> Title: "Cancel Platinum Subscription?" | Body: "You'll continue to have access until June 21, 2022." | Buttons: Cancel Subscription / Keep Subscription

**Error alert (before):**
> Title: "Oops! you can't do that" | Body: "Sorry, bad input. Error 1234567. Please try again." | Buttons: Okay / Cancel

**Error alert (after):**
> Title: "Billing Problem" | Body: "To continue accessing your subscription, add a new payment method." | Buttons: Add Payment Method / Not Now

**Empty state (before):**
> "Nothing strike your fancy? Please come back if you do find something you want to eat."

**Empty state (after, Apple Podcasts pattern):**
> "No Saved Episodes. Save episodes you want to listen to later, and they'll show up here."

## Takeaways
- Write with **Purpose**: every screen should have one clear goal; use information hierarchy to communicate it, and know what to leave out.
- Write with **Anticipation**: treat your app as a conversation; develop a consistent voice, vary tone to match the moment, and always answer the user's next question.
- Write with **Context**: match text density and complexity to what the user is doing; write specific, verb-based button labels in alerts; use empty states to educate, not decorate.
- Write with **Empathy**: use plain, gender-neutral language that localizes and scales; provide rich, intent-aware VoiceOver descriptions for every non-text element; read your writing out loud before shipping.

---
_Source: WWDC22 Session 10037 page (abstract and full transcript)._
