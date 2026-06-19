# Advances in CarPlay Systems
**WWDC19 · Session 252** · [Watch](https://developer.apple.com/videos/play/wwdc2019/252/)

_Platforms:_ iOS 13 (CarPlay vehicle system integration)

## Overview
This session is aimed at automotive OEM and tier-1 engineers integrating CarPlay into vehicle head units and instrument clusters. iOS 13 introduces three categories of display improvements — irregularly shaped display support, second screen / instrument cluster support, and dynamic screen resizing — plus a significantly redesigned audio architecture to enable "Hey Siri" hands-free activation from the car's always-on microphone. All features require adoption of the new CarPlay Communication Plug-in R15.

The session also describes a new CarPlay Dashboard (visible to end users) showing navigation, music, and Siri suggestions side-by-side, and explains how the new audio stream architecture allows music to continue playing in the background during Siri interactions, eliminating the silence gap that existed in previous versions.

## Key Topics

### Irregularly Shaped Display Support **[NEW]**
- **View area**: the bounding rectangle that CarPlay renders its H.264 video stream into — may be larger than the physically visible area.
- **Safe area**: a rectangular sub-region of the view area guaranteed to be fully visible and interactable; CarPlay positions all interactive controls (buttons, lists) and critical status info (time, signal, route guidance) within this boundary.
- Enables support for curved displays, displays partially obscured by trim pieces, and instrument cluster displays partially covered by virtual tachometers.
- The vehicle system declares both view area and safe area via the CarPlay Communication Plug-in R15.

### Second Screen / Instrument Cluster Support **[NEW]**
- iOS 13 can output multiple simultaneous H.264 video streams, one per display surface.
- Instrument cluster content types supported: Apple Maps maneuver instruction card, Apple Maps map view — each in its own video stream.
- Each video stream has an independent **night mode** (dark/light appearance), set by the vehicle system per stream.
- The vehicle chooses which content type appears in each stream; CarPlay only renders the requested type.
- iAP2 protocols remain the appropriate path for metadata overlays: Route Guidance (navigation metadata), Now Playing (audio track info), Communications (call state), Call Controls (mute/swap/end).
- View areas and safe areas apply equally to instrument cluster displays.
- Heads-up displays (HUDs) continue to use iAP2 Route Guidance metadata rather than video streams.

### Dynamic Screen Resizing **[NEW]**
- The size of the CarPlay UI can now change at runtime while a CarPlay session is active (previously fixed for the session duration).
- Multiple view areas can be declared per display; the vehicle system switches between them on user request.
- Use cases: configurable horizontal split screens, portrait displays divided into regions, instrument cluster tachometer repositioning.
- CarPlay provides an **Always Available** button the user can tap to enter/exit full-screen CarPlay without needing a dedicated native hardware control.
- Vehicle system declares the transition duration; CarPlay reports the current transition progress so the vehicle can synchronize native UI animations (e.g., moving virtual tachometers).
- Supported on both center console and instrument cluster displays.

### Hey Siri Audio Architecture **[NEW]**
- The vehicle's microphone runs continuously with an always-on echo cancellation and noise reduction (ECNR) pipeline.
- Processed audio is stored in a ring buffer (a few seconds of history) within the vehicle system.
- Two always-running detectors in the vehicle:
  1. **Voice Activity Detector (VAD)** — triggers when the driver begins speaking.
  2. **Keyword Detector** — triggers when "Hey Siri" is detected locally.
- On trigger: the vehicle notifies iPhone via the Communication Plug-in; iPhone opens an **Aux In** audio stream and requests the buffered historical audio.
- iPhone performs a second-pass offline keyword detector for verification before activating Siri — matches the always-on Hey Siri behavior in non-driving scenarios.
- Button-press Siri: same ring buffer is used; Siri activates immediately since the trigger is known and audio data is sent faster than real-time.
- **Music continues during Siri** **[NEW]**: a dedicated **Aux Out** stream for Siri audio output allows the vehicle to mix music + Siri + navigation guidance simultaneously; music volume ducks rather than stopping entirely.
- Vehicle mixer must support three simultaneous audio streams from iPhone: media, Siri output, navigation audio.

### CarPlay Dashboard (User Feature, iOS 13)
- New Dashboard mode shows navigation, music playback controls, and contextual Siri suggestions in a single view.
- No additional OEM integration required; provided by iOS 13.

## APIs & Frameworks

**CarPlay Communication Plug-in R15** **[NEW]** — vehicle-side integration SDK
- View area declaration API **[NEW]** — define view area rectangle and safe area rectangle per display
- Multiple view area declaration **[NEW]** — support dynamic resizing; declare transition duration
- Second screen (instrument cluster) API **[NEW]** — additional H.264 stream(s) for instrument cluster; per-stream content type and night mode selection
- Aux In audio stream **[NEW]** — bidirectional audio for Hey Siri historical buffer transfer
- Aux Out audio stream **[NEW]** — dedicated Siri audio output stream, separate from media
- Always Available CarPlay button **[NEW]** — CarPlay-side control for resize transitions

**iAP2 Protocols (existing, unchanged)**
- Route Guidance — navigation metadata for HUD / native instrument cluster rendering
- Now Playing — current audio track metadata
- Communications — phone call state
- Call Controls — mute, swap, end call actions

**System Features (iOS 13, no OEM API changes needed)**
- CarPlay Dashboard — new UI mode (automatic)
- Hey Siri second-pass keyword detection — runs on iPhone (automatic given vehicle hardware support)

## Code Highlights

No app-side Swift/Objective-C code is exposed in this session. Integration is via the CarPlay Communication Plug-in R15 (vehicle-side C/C++ SDK). Key configuration steps for vehicle system engineers:

1. Declare `view_area` and `safe_area` rectangles in the display configuration for each connected display.
2. For dynamic resizing, declare multiple `view_area` entries with `transition_duration` per display.
3. For instrument cluster, declare a second display with the desired `content_type` (map or instruction card) and independent `night_mode`.
4. Implement always-on microphone pipeline: ECNR → audio ring buffer → keyword detector + VAD.
5. Implement Aux In stream to receive historical audio from iPhone on trigger notification.
6. Implement three-stream audio mixer: media + Siri Aux Out + navigation guidance.

## Takeaways
- Safe areas decouple CarPlay's interactive rendering region from the physical display boundary, enabling correct UI placement on curved, partially occluded, or non-rectangular displays.
- Multiple simultaneous H.264 streams unlock instrument cluster integration — maps and turn instructions can appear in front of the driver without relying solely on iAP2 metadata overlays.
- "Hey Siri" in the car requires always-on vehicle microphone hardware, an ECNR pipeline, a two-detector architecture (VAD + keyword), and a ring buffer — it cannot be retrofitted cheaply, but provides fully hands-free Siri.
- The new Aux Out stream and three-stream mixer model eliminates the music-silence gap during Siri interactions, matching the ambient audio behavior users expect from smart speakers.

---
_Source: WWDC19 Session 252 page (abstract, chapter summaries, code samples, and resource links)._
