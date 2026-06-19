# 17 Big & Little Things at WWDC23
**WWDC23 · Session 111486** · [Watch](https://developer.apple.com/videos/play/wwdc2023/111486/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10, tvOS 17

## Overview
This short highlight reel from the first day of WWDC23 tours seventeen notable features and announcements across Apple platforms — a mix of flagship headline features and small quality-of-life improvements that collectively define the iOS 17 / macOS Sonoma release cycle.

The video covers both hardware (new MacBook Air 15-inch) and software (iOS 17, watchOS 10, macOS Sonoma), giving developers and enthusiasts a quick visual tour of what's new without the depth of the full Keynote or Platforms State of the Union.

Because this is a consumer-facing highlight reel rather than a technical session, it contains no API-level content. Developer-facing deep dives for each of these features are covered in dedicated WWDC23 sessions.

## Key Topics

- **NameDrop** — AirDrop-based contact sharing by bringing iPhones together.
- **Contact Poster** — Personalized caller ID cards shown during calls and FaceTime.
- **FaceTime voicemail** — Leave a video message when a call goes unanswered.
- **FaceTime handoff to Apple TV** — Move an active FaceTime call from iPhone to the big screen.
- **Remote finder** — iPhone can locate the Apple TV Siri Remote using Find My-style audio cues.
- **Stickers from Photos** — Create emoji-style stickers from people and objects in your photos.
- **Journal app** — Reflect on your day with AI-driven suggestions from photos, music, locations, and workouts.
- **Topographic and offline Maps** — Download detailed topo maps for hiking and areas without connectivity.
- **Snoopy watch face** — New Apple Watch face featuring Snoopy animations.
- **Check In** — Automatically notifies a contact when you arrive home safely.
- **Adaptive Audio (AirPods)** — A new AirPods listening mode that blends Active Noise Cancellation and Transparency to focus on important sounds.
- **Multiple timers and Widgets** — Support for multiple simultaneous timers; widgets arrive on additional devices.
- **MacBook Air 15-inch** — World's thinnest 15-inch laptop announced.
- **Game Mode on Mac** — macOS Sonoma feature that prioritizes game performance.
- **StandBy mode** — When iPhone is charging horizontally, it becomes a bedside clock, music player, or Live Activity display.
- **Live Voicemail** — See a real-time transcription of a voicemail as someone leaves it, and choose to pick up.
- **Improved Autocorrect** — Enhanced on-device machine learning for smarter text correction.
- **Mood and emotion logging** — New Mental Health tracking in Health app on iPhone, iPad, and Apple Watch.
- **Apple Vision Pro** — The "really big" announcement: Apple's first spatial computing headset.

## APIs & Frameworks
This session is a consumer highlight video with no technical API content. For developer APIs related to features shown, see:

- **NameDrop / Sharing** — related to `Contacts` framework updates
- **Contact Poster** — `ContactsUI` **[NEW]**
- **Journal app** — `JournalingSuggestions` framework **[NEW]**
- **Widgets on additional surfaces** — `WidgetKit` (expanded platform support) **[NEW]**
- **StandBy** — `WidgetKit` StandBy mode support **[NEW]**
- **Live Voicemail transcription** — `Speech` framework
- **Mood logging** — `HealthKit` state of mind / emotion APIs **[NEW]**
- **Maps offline/topo** — `MapKit` updates
- **Game Mode** — `GameKit` / Metal priority APIs **[NEW]**
- **Vision Pro** — `visionOS`, `RealityKit`, `SwiftUI` spatial APIs **[NEW]**

## Code Highlights
No code samples — this is a consumer highlight reel, not a technical session.

## Takeaways
- WWDC23 introduced an unusually large breadth of new user-facing features alongside Apple's most significant new platform in decades (visionOS / Vision Pro).
- Many small quality-of-life features (multiple timers, improved autocorrect, offline maps) have underlying API support that developers can leverage.
- StandBy and Contact Poster represent new surfaces developers should design for with WidgetKit and ContactsUI.
- Vision Pro is the dominant developer story of WWDC23 and is covered in dedicated visionOS sessions.

---
_Source: WWDC23 Session 111486 page (abstract, chapter summaries, code samples, and resource links)._
