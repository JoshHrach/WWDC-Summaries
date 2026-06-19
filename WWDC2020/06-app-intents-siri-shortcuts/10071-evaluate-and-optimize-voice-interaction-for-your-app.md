# Evaluate and Optimize Voice Interaction for Your App
**WWDC20 · Session 10071** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10071/)

_Platforms:_ iOS 14, iPadOS 14 (Siri conversational design guidance)

## Overview
A conceptual and design-focused session from Apple's Siri team that clarifies the full landscape of Siri technologies — system intents, custom intents, Shortcuts, and Suggestions — and explains how to choose the right one for a given use case. The session then dives into what makes voice interaction truly excellent: selecting the right features to voice-enable, supporting both silent and voice modes, and writing natural conversational dialogue.

This session is a design companion to the engineering sessions on SiriKit and Shortcuts (10073, 10060, 10061, etc.). It contains no code but is essential context for making principled decisions about which Siri integration path to take and how to present it to users.

## Key Topics

### The Siri Technology Landscape
Siri encompasses several distinct but related technologies:

**System Intents (SiriKit Domains)** — Apple-defined intents for common everyday tasks (messaging, media, calling, rides, payments, etc.). The conversational flow is provided; developers supply data to complete it. Dialogue is pre-written but developers must verify their data fits naturally into the provided strings and that error cases map to the right dialogue.

**Custom Intents** — Developer-defined intents for app-specific tasks (e.g., ordering coffee, advancing a recipe step). The conversational flow is not provided — developers design it using the intent definition file and Siri dialog fields. Developers are the domain experts and should think through: what steps are needed, what information must users provide, and how would someone naturally talk through the flow.

**Shortcuts** — An umbrella term covering:
- **Suggestions** — Siri can proactively surface shortcuts for repeated (or predicted) actions.
- **Shortcuts App** — pre-made and user-created multi-step workflows; supports sharing.
- **Automations** — device-level or Home automations triggered by time, location, or events.
- **Voice Shortcuts** — powered by custom intents; allow users to invoke in-app actions by voice without opening the app.

### Choosing What to Voice-Enable
Voice is not the right modality for every app feature. It excels when:
- **Simplifying multi-step tasks**: playing a specific album requires finding the app, navigating catalogs, then selecting; voice collapses it to one sentence.
- **Accelerating past navigation**: frequent users of a feature don't need the full app hierarchy every time — voice skips directly to the action.
- **Reaching buried functionality**: power features not prominently displayed in UI become easy to access by voice.
- **Enabling multi-tasking scenarios**: driving, walking with AirPods, cooking — situations where the phone screen isn't practical.

Voice is less well-suited to single-step tasks that are already fast in the app, or tasks that require visual inspection of results before proceeding.

### Silent Mode and Voice Mode (iOS 14)
iOS 14 behavior change: when the iPhone ringer switch is muted, Siri defaults to **silent mode** — Siri does not speak, relying on the screen UI and printed dialogue only.

In **voice mode**, Siri speaks all information users need to complete the task, in addition to showing screen UI.

- **System intents**: both modes are handled automatically.
- **Custom intents**: developers must provide separate dialogue for voice mode (what Siri speaks) and screen mode (what appears in the UI). The intent definition file supports this; Xcode tooling makes it straightforward.

### Attribution and Repeated Use
Siri includes app attribution ("here's what [AppName] found") the first time a shortcut is used. After the user has used the shortcut multiple times and is familiar with it, Siri omits the attribution to keep responses shorter. This means the app's dialogue must be conversationally self-sufficient — it cannot rely on attribution to provide context.

### Conversational Dialogue Best Practices
- **Use questions** when expecting user input — they signal that a response is required.
- **Avoid jargon** — the developer is an expert on the domain; users may not be.
- **Spoken English, not written English** — contractions, natural phrasing; avoid formal written constructions.
- **Keep responses short** — audio is linear and ephemeral; people cannot skim or re-read.

## APIs & Frameworks
This session is design-focused and does not introduce new APIs. It references:

- **SiriKit / Intents framework** — system intent domains (Messaging, Media, Payments, etc.) and custom intent definition files (`.intentdefinition`)
- **Siri Dialog editor** in Xcode — define per-parameter and per-phase voice/screen dialogue for custom intents
- **Silent mode** — automatic in iOS 14 when ringer is muted; requires custom intent dialogue to be complete in text form for the screen-only path
- **Shortcuts app** — `INShortcut`, `INVoiceShortcutCenter`, donation via `INInteraction`
- See companion sessions: 10073 (Empower your intents), 10060 (Design high quality Siri media interactions), 10061 (Expand SiriKit Media Intents to more platforms), 10074 (Decipher and deal with common Siri errors)

## Takeaways
- Siri encompasses multiple technologies — choose system intents when Apple has defined the conversational flow for your domain; choose custom intents when your use case is unique to your app.
- Voice-enable features that involve multiple steps, frequent use, deep navigation, or multitasking scenarios — not features that are already trivially fast in the app.
- iOS 14's silent mode (ringer muted) requires custom intents to have complete screen-facing dialogue that works without speech; system intents handle this automatically.
- Siri progressively shortens its responses as users become familiar with a shortcut, so app dialogue must stand on its own without relying on attribution framing.
- Write dialogue for spoken delivery: use questions to prompt input, avoid jargon, use natural spoken phrasing, and keep responses brief.

---
_Source: WWDC20 Session 10071 page (abstract and transcript)._
