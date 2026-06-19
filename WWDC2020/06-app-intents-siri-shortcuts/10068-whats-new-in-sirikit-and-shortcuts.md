# What's New in SiriKit and Shortcuts
**WWDC20 · Session 10068** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10068/)

_Platforms:_ iOS 14, iPadOS 14, watchOS 7

## Overview
iOS 14 brought a redesigned compact Siri UI, significant Shortcuts app improvements, and new opportunities for developers to surface app actions across the system. The compact Siri interface appears as an overlay rather than taking over the full screen, emphasizing concise, essential responses and minimizing disruption. Shortcuts gained folders, automation suggestions in the gallery, new automation trigger types, and a dedicated Shortcuts app on watchOS 7.

The session also introduced `INShortcutAvailabilityOptions` for tagging shortcuts as sleep-friendly for the new Wind Down feature, and noted that disambiguation lists in intents can now include images and subtitles to help users make better choices.

## Key Topics

### Compact Siri UI (iOS 14)
Siri was completely redesigned with a compact overlay that appears on top of the current app rather than replacing it. Intents UI extensions now appear within this compact overlay whether run from Siri or Shortcuts. Key design principles:
- Minimize vertical space in `INUIHostedViewControlling` to feel lightweight
- Use transparent backgrounds to defer to the system material
- Keep responses focused on the single most essential answer

### Disambiguation with Images and Subtitles
Intents that require disambiguation can now present option lists with images and subtitles per item, helping users visually differentiate between similar options (e.g., multiple soup varieties). Developers should use imagery judiciously to avoid overwhelming the compact UI.

### Shortcuts App: Folders
Users can now organize their shortcuts into folders. Smart folders automatically group shortcuts by system behavior (e.g., shortcuts in the share sheet, shortcuts available on Apple Watch).

### Shortcuts on watchOS
A new Shortcuts app on watchOS 7 lets users run shortcuts directly from the watch. Shortcuts can be set as watch face complications. Developers don't need to do anything special — their existing shortcuts work on the watch. Users control availability via an "Apple Watch" toggle in Shortcut details.

### New Automation Triggers
Additional trigger types for Personal Automations:
- Receive email or message
- Close a specific app (complementing the existing Open App trigger)
- Battery level threshold
- Connect to charger

More trigger types now run automatically without requiring user confirmation.

### Automation Suggestions in Gallery
The Shortcuts gallery now includes automation suggestions personalized to how users use their devices, making it easier for users to discover and set up automations involving your app's actions.

### Wind Down Integration
Apps in categories like mindfulness, journaling, or music can opt their shortcuts into the Wind Down sleep routine setup experience using `INShortcutAvailabilityOptions`. Mark shortcut donations as sleep-friendly with the `.sleepMindfulness` (or equivalent) availability option.

### Add to Siri Button Placement
Developers should place "Add to Siri" buttons in contextually relevant locations (not just buried in settings) to maximize shortcut adoption.

## APIs & Frameworks

### SiriKit / Intents Framework
- `INIntent` — base class for custom intents
- `INShortcut` — shortcut wrapping an intent or user activity
- `INShortcutAvailabilityOptions` **[NEW]** — availability flags; includes sleep/Wind Down options for marking shortcuts as suitable for Wind Down routines
- Disambiguation list items — now support images and subtitles **[NEW]**
  - `INObject` or custom intent parameter types with image properties
- `INUIHostedViewControlling` — Intents UI protocol; snippet views now rendered in compact Siri overlay
- `SiriButton` / `INUIAddVoiceShortcutButton` — "Add to Siri" button for in-app shortcut creation

### Shortcuts / Intents UI
- Compact Siri overlay **[NEW]** — Intents UI appears as overlay on current context (not full-screen takeover)
- Transparent background support for Intents UI views **[NEW]** — use system material
- Shortcuts folders **[NEW]** — user-organized shortcut groupings; smart folders for share sheet and Apple Watch
- Shortcuts app for watchOS **[NEW]** — run shortcuts from Apple Watch; set as watch face complications
- Automation triggers (new types) **[NEW]**:
  - Email/message received trigger
  - App closed trigger
  - Battery level trigger
  - Charger connected trigger
- Automation suggestions in Gallery **[NEW]** — system-personalized automation recommendations

### Donation APIs
- `INInteraction.donate(completion:)` — donate intent interactions to power Siri suggestions
- `INShortcut(intent:)` — create shortcut from intent for "Add to Siri"
- `INVoiceShortcutCenter.shared.setShortcutSuggestions(_:)` — suggest shortcuts proactively

## Code Highlights

No specific code snippets were presented in this session. The session is primarily an overview; detailed implementation is covered in companion sessions:
- "Empower your intents" (WWDC20 Session 10073) — deeper intent customization
- "Feature your actions in the Shortcuts app" (WWDC20 Session 10084) — surfacing actions in Shortcuts gallery
- "Integrate your app with Wind Down" (WWDC20 Session 10083) — `INShortcutAvailabilityOptions`
- "Create quick interactions with Shortcuts on watchOS" (WWDC20 Session 10190)

## Takeaways

- The compact Siri UI in iOS 14 requires Intents UI views to be concise and preferably transparent-backgrounded; tall snippet views will feel heavy in the new overlay context.
- Disambiguation option lists now support images and subtitles, making it easier for users to choose between similar items during voice interactions.
- Shortcuts gained folders and a dedicated watchOS app — existing shortcut donations automatically work on the watch, with no extra developer effort.
- Tag shortcuts with `INShortcutAvailabilityOptions` sleep options to appear in the Wind Down setup experience for health/wellness apps.

---
_Source: WWDC20 Session 10068 page (abstract, transcript, and resource links)._
