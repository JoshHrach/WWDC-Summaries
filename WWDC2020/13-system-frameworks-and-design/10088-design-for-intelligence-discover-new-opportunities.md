# Design for Intelligence: Discover New Opportunities
**WWDC20 · Session 10088** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10088/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7

## Overview
This is the second session in the four-part "Design for Intelligence" series. Presented by JP Lacerda from Apple's Proactive Intelligence team, it defines what intelligence is trying to accomplish — making Apple products feel like they know you — and surveys the entry points through which apps can participate in the intelligent system. It also provides concrete engagement metrics from apps that have adopted intelligence integrations.

This session transitions from the "why" framing of session 10086 to the "what" — specifically, what entry points exist and what measurable business impact intelligence adoption has.

## Key Topics

**Defining the Goal of Intelligence**
Intelligence should make users feel like their Apple products know them: their goals, intentions, habits, preferences, interests, and relationships. Intelligence accelerates users toward goals they already have in mind and helps them discover meaningful content, people, places, and apps at the right moment. The measure of success is less tedium (fewer taps to get somewhere) and fewer distractions (focus on what matters).

**Intelligence Entry Points Covered**
The session surveys multiple intelligence entry points where apps can appear without requiring a home screen tap:
- **Shortcuts** — voice-triggered and tap-triggered app features, plus lock screen and Search suggestions (new in iOS 14: Smart Stack positioning for widgets)
- **Sharing Suggestions** — intelligently surfaces contacts and groups most likely relevant to the content being shared
- **Siri Event Suggestions** — reservation data from apps (or web markup) surfaced in Calendar and on the lock screen with departure-time notifications
- **Maps Proactive Suggestions** — directions to relevant places surfaced with a single tap
- **Do Not Disturb Suggestions** — Siri suggests enabling DND at appropriate moments
- **Lock Screen Suggestions** — e.g., airline check-in notification surfaced at the right moment
- **Siri Suggestions Widget** (new in iOS 14) — the "Siri Suggestions" widget in the Smart Stack
- **Smart Stack** — intelligently rotates to show the most relevant widget at any given time

**Measurable Business Impact**
The session cites specific engagement metrics:
- First-time Sharing Suggestion engagement → users share on average **twice as much** as they did before through that app
- Airline Siri Event Suggestion check-in: **82% of notification check-ins** came from the Siri Event Suggestion action for apps that adopted it
- Third-party apps seen on average **5 times per day** across lock screen, Sharing, Search, and other entry points when properly integrated

**Developer Responsibility**
Developers should identify which intelligence entry points fit their app's key actions, adopt those entry points, and instrument their analytics to measure the impact. The session concludes by pointing to the next video ("Meet People Where They Are", 10200) to understand the user journey perspective.

**Privacy**
All intelligence runs on-device. No behavioral data leaves the device without explicit user consent. The analytics cited come only from users who explicitly opted in to share analytics with Apple.

## APIs & Frameworks

### Shortcuts / SiriKit
- `INIntent` — defines an app's actions to the system
- `INInteraction` — wraps an intent with context; used to donate usage to the system
- `INShortcut` — wraps an intent or URL for lock screen / Search suggestions
- `INRelevantShortcut` — relevance-annotated shortcut for the Siri watch face and Siri Suggestions widget

### WidgetKit (new in iOS 14)
- `Widget` / `WidgetConfiguration` — declares a widget
- `IntentConfiguration` — widget configuration backed by an `INIntent` for user customization
- Smart Stack — system-managed stack of widgets; rotates based on donations and usage patterns

### Siri Event Suggestions
- `INReservation` (and subclasses: `INRestaurantReservation`, `INFlightReservation`, etc.) — structured reservation data
- `INGetReservationDetailsIntent` — intent for surfacing reservation details
- Web markup (new in iOS 14 / macOS Big Sur) — structured data on websites/emails that Siri can parse for event suggestions (Safari and Mail integration)

### Maps Integration
- `MKMapItem` — pass a map item to Maps via `openInMaps(launchOptions:)` or `INReservation` to power proactive direction suggestions

## Code Highlights

No new code samples specific to this session. The session is a design/metrics overview. For implementation, see:
- "Broaden your reach with Siri Event Suggestions" (10197) for `INReservation` usage
- "Add configuration and intelligence to your widgets" (10194) for `IntentConfiguration` and Smart Stack
- "What's new in SiriKit and Shortcuts" (10068) for `INInteraction` donation patterns

## Takeaways
- Participating in intelligence entry points (Shortcuts, Siri Event Suggestions, Smart Stack) has directly measurable engagement impact: first-time Sharing Suggestion users double their sharing; 82% of check-in notifications come from Siri Event Suggestion actions for apps that adopt it.
- Intelligence surfaces your app up to five times per day across the system without any user action — each of these surfaces is an opportunity the system provides for free when you donate intent interactions.
- Smart Stack positioning (new in iOS 14) uses donations from your main app to automatically rotate your widget to the top at the right moment — no additional API required beyond what you already donate for Shortcuts.
- Map departure-time notifications and lock screen suggestions require donating structured reservation data via `INReservation`; the system handles all the prediction and surfacing logic.

---
_Source: WWDC20 Session 10088 page (abstract and full transcript)._
