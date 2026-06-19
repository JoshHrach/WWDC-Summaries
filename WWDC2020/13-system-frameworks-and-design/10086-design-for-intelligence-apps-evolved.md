# Design for Intelligence: Apps, Evolved
**WWDC20 · Session 10086** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10086/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7

## Overview
This is the introductory session in a four-part "Design for Intelligence" series presented at WWDC20. Delivered by Mark Mikin, Siri Experience Evangelist at Apple, it defines what Apple means by "the intelligent system experience" and frames it as a design practice — not merely a feature checkbox. The session argues that app extensions, Siri suggestions, voice, widgets, and App Clips are not separate features but a unified design philosophy that has been evolving since the first app extensions at WWDC 2014.

This session sets up the conceptual framework for the three follow-on sessions (10088 Discover New Opportunities, 10087 Make Friends with "The System", and 10200 Meet People Where They Are). It contains no code samples.

## Key Topics

**Defining the Intelligent System Experience**
The simplest definition: how the operating system works with the apps people use every day to make "the everyday" easier. The session distinguishes this from mere "convenience" — it argues that intelligence is a design practice that should be treated as a core platform convention, like the share button, not an optional add-on.

**Intelligence as a Living Design**
Unlike a static glyph or icon, intelligence adapts to how the system conforms to an individual user's behavior. No two users' Siri Suggestions or conversations with Siri are alike. This "living design" evolves continuously. The practical implication: people now expect their devices and apps to be smart, and they don't distinguish between system-level and app-level intelligence failures — it just feels broken.

**Extensibility as the Enabling Mechanism**
The session reframes the history of app extensibility through the lens of intelligence:
- WWDC14: First app extensions introduced
- WWDC16: SiriKit introduced using the app extensions foundation
- Subsequent years: Shortcuts, new Widgets, App Clips — each building on the same extensibility model
- Handoff: originally for iPad/iPhone/Mac continuity, became foundational for Apple Watch and device-to-device intelligence

The point: extensibility is not just a technical mechanism — it's how an app participates in the intelligent system.

**Privacy as a Foundation**
The session explicitly states that no intelligent system is worth sacrificing privacy for. The intelligent system at Apple is built from the ground up with privacy as a foundational requirement, not an afterthought. Developers should carry this principle into their own intelligence integrations.

**The Three Key Concepts**
Three recurring themes run throughout the series:
1. Extending quick interactions that save people time
2. Measuring successful intelligent interactions and their impact on app value
3. Helping people discover and adopt intelligence features throughout their user journey (including before they download the app)

The session promises that "just a few APIs" can provide significant user value when used correctly.

**Practical Developer Responsibility**
Because apps are a key piece of the intelligent system through extensibility, developers have a meaningful responsibility to integrate well. When intelligence integrations fall short of user expectations, users blame the overall experience — not specifically the app or the OS.

## APIs & Frameworks

This session is a framing/philosophy session. It references the following technologies contextually without API details:

- **App Extensions** — the foundational mechanism for all system intelligence integration
- **SiriKit** — Siri integration via intent extensions (debuted WWDC16)
- **Shortcuts** — user-composable automations built from app intents
- **Widgets** — home screen and Today view extensions (new in iOS 14)
- **App Clips** — lightweight, on-demand app experiences (new in iOS 14)
- **Handoff** — cross-device continuity, now foundational for Apple Watch integration

Technical details are deferred to the companion sessions in this series and throughout the WWDC20 SiriKit/Shortcuts track.

## Code Highlights

No code samples were provided in this session. It is a design philosophy and framing session.

## Takeaways
- System intelligence is a platform convention — like the share button, it is something reinforced by consistent appearance and behavior across apps. Developers who don't participate make the overall platform feel less intelligent and undermine user expectations.
- Extensibility is the mechanism that allows apps to contribute to the intelligent system: app extensions, SiriKit, Shortcuts, Widgets, and App Clips are all part of the same continuous evolution that began in 2014.
- Intelligence is a living design: it adapts to individual users in real time, meaning no two users experience it identically — developers must design for this dynamic behavior rather than static layouts.
- Privacy is a prerequisite, not a tradeoff — the session frames it as a foundational design constraint of Apple's intelligent system.

---
_Source: WWDC20 Session 10086 page (abstract and full transcript)._
