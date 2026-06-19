# Design for Intelligence: Meet People Where They Are
**WWDC20 · Session 10200** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10200/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7

## Overview
This is the fourth session in the "Design for Intelligence" series. Presented by Radhika from Apple's Proactive Intelligence team, it shifts from technology to user perspective, walking through a complete user journey — from discovering an app to becoming a power user — and showing where system intelligence integrations appear at each stage. The vehicle is a fictional gym-goer story that demonstrates six distinct intelligence touchpoints in natural sequence.

This session answers the question: "How does intelligence participation translate into better outcomes for users at different stages of adoption?" It contains no code, but each touchpoint maps to specific API technologies covered in other sessions.

## Key Topics

**User Journey: Pre-Download**
Before any app is installed, Maps can surface a suggestion to navigate to a gym based on an address received in Messages. This requires no developer effort — the system parses location context from Messages automatically. The lesson: even apps that aren't installed benefit from participating in the system's contextual model through other integrations (e.g., your business's address in Messages triggers Maps suggestions).

**User Journey: First Discovery (App Clips)**
A new user at a gym can access the gym's schedule via an NFC-triggered App Clip — no app download required. App Clips provide the most relevant feature of the app in context, and offer a download path to the full app. This is the system's lowest-friction acquisition funnel.

**User Journey: Early Adoption (Siri Suggestions in Search)**
After downloading, the app appears in Siri Suggestions in Spotlight Search before the user even starts typing, because Siri has learned from her usage patterns (she opens the app Sunday mornings). The system learns from frequency, time, and context of app launches — developers contribute by donating intent interactions.

**User Journey: Growing Habit (Shortcut Suggestions)**
After a few weeks of use, the system surfaces shortcut suggestions contextually (e.g., "Yoga Schedule" appears in Search after work). This saves several navigation steps and appears without any user setup — it is powered purely by intent donations from regular in-app navigation. The key: donate an intent when users view specific content sections, not just when they make purchases.

**User Journey: Power User (Widgets and Smart Stack)**
The user adds the gym app widget to a Smart Stack. Because the system has learned from her interaction patterns, the gym widget automatically rotates to the top of the stack when class schedules are relevant — before she even thinks to check. This is driven by the same donation signals used for Shortcuts suggestions. No additional API required beyond intent donations.

**User Journey: Automation (Shortcuts Automations)**
The final stage: the user sets up a Shortcuts automation that triggers when she arrives at the gym — running multiple actions (bus schedule check, workout tracker start) automatically. This is possible only when apps define and expose their key actions as custom intents.

**Why Some Apps Don't Stick**
The session implicitly explains app abandonment: if an app doesn't expose its key functionality as shortcuts, donate interactions, or provide a widget, it cannot participate in any of the above journey stages beyond the initial download. The system cannot learn, cannot suggest, cannot rotate the widget intelligently.

**The Intelligence Spectrum**
The journey illustrates that intelligence participation is not binary — there is a spectrum from passive (Maps picks up a business address from Messages) to active (full intent definition, donation, Shortcuts automation, Smart Stack widget). Each layer adds more value and deeper integration with the user's routine.

## APIs & Frameworks

### App Clips
- App Clip experience (NFC, QR, Safari Smart App Banner, Maps, Messages) — entry points for App Clip invocation
- `NSUserActivity` with `NSUserActivityTypes` — deep-links the App Clip to specific content
- App Store Connect — configure App Clip experiences and advanced experiences

### SiriKit / Shortcuts
- Custom `INIntent` definitions — key actions users perform in the app
- `INInteraction.donate(completion:)` — donate usage after each user action
- Siri Suggestions (lock screen, Search, Siri Suggestions widget) — powered by donations
- Shortcut suggestions in Spotlight — auto-generated after pattern detection from donations
- Shortcuts automations — user-configurable; trigger based on time, location, or NFC

### WidgetKit
- `IntentConfiguration` — user-configurable widget backed by intent parameters
- Smart Stack rotation — automatic, powered by donation history from the main app

### Maps / Proactive Suggestions
- Location context from Messages — system-level; no API required
- Directions suggestion — system-level; powered by `INReservation` or implicit context

### Spotlight / Search
- App suggestions in Spotlight before typing — powered by app launch patterns (system-level)
- Shortcut suggestions in Spotlight — powered by `INInteraction` donations with parameters

## Code Highlights

No code samples were provided in this session. It is a user journey / design perspective session.

The technology touchpoints shown map to these implementation sessions:
- App Clips: "Create App Clips for Other Businesses" (10118) and the App Clips track
- Intent donations: "Design for intelligence: Make friends with 'The System'" (10087)
- Widgets: "Add configuration and intelligence to your widgets" (10194)
- Siri Event Suggestions: "Broaden your reach with Siri Event Suggestions" (10197)

## Takeaways
- Intelligence integrations benefit users at every stage: pre-download (Maps picks up your business address), discovery (App Clip), early adoption (Siri Suggestions in Search), habit formation (shortcut suggestions), power user (Smart Stack rotation), and automation (Shortcuts automations) — each stage requires progressively more developer investment.
- App Clips are the system's lowest-friction acquisition funnel: they deliver the most relevant app feature in context without requiring a download, and include a clear path to the full app install.
- Donate intents not just for transactions (purchases, orders) but for navigational actions (viewing a schedule, opening a section) — these navigational donations are what power contextual shortcut suggestions that appear in Search after work without any user configuration.
- Smart Stack rotation requires no additional API beyond the intent donations you already make for Shortcuts — the same donation signals drive both features.

---
_Source: WWDC20 Session 10200 page (abstract and full transcript)._
