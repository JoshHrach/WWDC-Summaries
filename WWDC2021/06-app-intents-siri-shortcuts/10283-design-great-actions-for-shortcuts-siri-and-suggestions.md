# Design great actions for Shortcuts, Siri, and Suggestions
**WWDC21 · Session 10283** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10283/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8

## Overview
This design session explains how to create Shortcuts actions that extend your app's functionality beyond its own UI—enabling tasks via the Shortcuts app, Siri voice commands, Suggestions widgets, Lock Screen, and Search. The session defines five qualities of great actions: useful, modular, multimodal, clear, and discoverable, and walks through each with concrete examples from real and hypothetical apps.

A key theme is that well-designed actions are composable building blocks. Each action does one thing well and exposes typed outputs that other actions can consume, enabling shortcuts authors to create workflows the app developer never anticipated. The session also covers iOS 15 improvements to shortcut sharing—including instant "Add to Siri" with a suggested phrase—and redesigned action appearance in the Shortcut editor.

## Key Topics

### Useful Actions
- An action should represent a task that users already perform inside the app.
- Perform the task completely without launching the app into the foreground whenever possible.
- If information is missing, ask a follow-up question (resolution) instead of opening the app.
- Avoid immersive tasks (complex visual UIs with custom previews) as actions; favor quick, repeatable tasks.
- Prioritize building actions for the most frequently used features first.

### Modular Actions
- Each action should do exactly one thing; compose multiple atomic actions to achieve complex goals.
- Vague all-in-one actions are inflexible and unclear in Suggestions; modular actions are reusable across contexts.
- Actions should expose rich, typed outputs whose individual properties can be used as variables in subsequent actions.
- Recommended action types for app-specific entities:
  - **Create** — adds a new item, passes it as output with confirmation dialogue
  - **Edit** — modifies an item, passes updated item as output
  - **Delete** — removes an item
  - **Get** — finds items matching criteria, passes results as output with dialogue summarizing what was found
  - **Thing** — lets user pick a specific item; shows custom UI and passes item as output
  - **Open** — opens the app to a specific item in the foreground

### Multimodal Actions
- Actions must work across three contexts: tapping a shortcut in the UI, running via Siri voice, and configuring in the Shortcut editor.
- Prompts must be phrased as questions (not labels like "Deadline:") so Siri can speak them naturally.
- Dynamic Enumerations provide custom picker UIs for app-specific parameter types, shown both during configuration and resolution.
- Snippets (custom result UIs) should preview the outcome before a permanent action, and confirm it after.

### Clear Parameter Summaries (Action Appearance in Editor)
- iOS 15 redesigns actions to focus on the parameter summary; app name is removed from the action card, replaced by an inline app icon.
- Parameter summaries must start with a verb and read like a natural sentence fragment (no terminal punctuation).
- Optional parameters should be hidden in the "Options" disclosure UI (Show More chevron), not in the summary.
- Provide default values for parameters where sensible—this allows the action to run without prompting.
- Any parameter can be set to "Ask Each Time" by users; always provide prompts and/or visual lists for every parameter.
- Action title (in the actions list) should share as many words as possible with the parameter summary, starting with the same verb.

### Discoverable Actions
- iOS 15 and macOS Monterey add new shortcut sharing: links can be placed on websites, in apps, and on social media.
- Updated download UI shows which platforms a shortcut can run on.
- "Add to Siri" now adds the shortcut instantly using your suggested invocation phrase; without a phrase, users must provide their own name, reducing successful adoption.
- Donate actions to the system so Siri can suggest them at the right time in Suggestions widget, Search, and Lock Screen.

## APIs & Frameworks

**SiriKit / Shortcuts (Intents framework)**
- `INIntent` subclasses — define custom intent actions **[existing]**
- `INIntentHandling` protocol — implements the action logic **[existing]**
- `INParameter` — typed parameter definition within an intent **[existing]**
- `INInteraction` — donated to system so Siri learns usage patterns **[existing]**
- Parameter summary (`parameterSummary`) — natural-language description of an action for the editor **[existing, redesigned display in iOS 15]**
- Custom parameter types (Dynamic Enumerations) — app-defined enum types with custom picker UI **[existing]**
- Snippets (`INUIHostedViewControlling`) — custom UI shown when running an action **[existing]**
- Output properties — strongly typed properties on intent responses, consumable as variables by downstream actions **[existing]**
- Suggested invocation phrase (`suggestedInvocationPhrase`) — pre-filled phrase for "Add to Siri" flow **[existing, now required for instant add]**
- Ask Each Time variable — user-set runtime prompting for any parameter **[existing]**

**Shortcuts App (iOS 15 / macOS Monterey)**
- Redesigned action card: inline app icon, parameter summary prominent, app name removed **[NEW visual design]**
- Instant "Add to Siri" with suggested phrase **[NEW behavior in iOS 15]**
- Updated shortcut download UI showing supported platforms **[NEW]**
- Drag-and-drop action composer **[NEW in Shortcuts for macOS]**

## Code Highlights

No code samples were shown in this design session. Implementation details are covered in:
- "Donate intents and expand your app's presence" (WWDC21, Session 10231)
- "Meet Shortcuts for macOS" (WWDC21, Session 10232)
- "Evaluate and optimize voice interaction for your app" (WWDC20, Session 10071)

## Takeaways
- Design actions to be atomic and composable: one action does one thing, exposes typed outputs, and works standalone or chained with others from any app.
- Phrase all prompts as questions and provide default values so actions work naturally whether tapped, spoken to Siri, or configured in the editor.
- Always provide a `suggestedInvocationPhrase`—without it, "Add to Siri" requires the user to invent a name, sharply reducing adoption.
- Prioritize repeatable, high-frequency tasks that complete without foregrounding the app; reserve foreground-opening actions for inherently visual flows.

---
_Source: WWDC21 Session 10283 page (abstract, full transcript, and resource links)._
