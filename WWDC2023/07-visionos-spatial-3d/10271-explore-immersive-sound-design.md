# Explore Immersive Sound Design
**WWDC23 · Session 10271** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10271/)

_Platforms:_ visionOS 1

## Overview
This session is presented by an Apple sound designer on the Design team and explores how Spatial Audio works in visionOS and how developers and designers can use it to create more immersive, responsive app experiences. It covers the fundamentals of how the system renders sound using room geometry, how to design UI interaction sounds, and how to build layered immersive soundscapes for fully immersive environments.

The session demonstrates two main contexts for spatial audio in visionOS apps: UI sounds that confirm interactions (keyboard clicks, photo browsing) and full ambient soundscapes (like Apple's Mount Hood environment) that fill the user's space. Both contexts share common principles — subtlety, spatial placement, and randomization — and can be applied in any type of visionOS app, from windowed apps to fully immersive spaces.

The design philosophy is that spatial audio is not just decorative; it provides familiarity, confirmation of interaction, and a sense of depth that is especially valuable when users are not touching physical objects. The system's understanding of room geometry (size, materials) enables sounds to reverberate naturally, making the experience feel grounded in the user's real environment.

## Key Topics

- **How Spatial Audio works on visionOS** — The system understands the physical space around the user (size, materials) and adds matching reverberation to sounds, so spatial audio sources sound like they are physically located in the room. Volume and direction cues work together; the system mimics real-world sound propagation.
- **Spatial audio sources vs. ambient background audio** — Two main categories: spatial audio sources (point-in-space sound objects like birds, UI elements, keyboard keys) and ambient background audio (surround audio anchored to play all around the user, looping continuously). Both are used together for rich soundscapes.
- **UI interaction sounds** — Adding subtle sounds to hand/eye interactions gives users familiarity and confirms actions without physical touch feedback. Key principle: sounds should be subtle and repeat often, so randomizing pitch and amplitude slightly for each repetition makes them feel natural rather than mechanical (demonstrated with virtual keyboard key presses).
- **Sound design for Photos app** — Selecting sounds that match the visual aesthetic and OS sound language rather than literal/obvious mappings; sounds must align with transition timing; subtlety is critical for frequently heard sounds.
- **Designing immersive soundscapes (Mount Hood example)** — Real location recordings often include unwanted noise; designers curate and reconstruct the "best of reality" rather than using raw field recordings. Process: record with immersive/directional microphones; select the best layers; position spatial sources (foreground, midground, distance); layer over ambient background audio.
- **Placement and distance tools** — Pushing a sound's position into the distance makes it sound farther away; volume adjustments complement positional placement. Together these tools control the perceptual depth of the soundscape.
- **Randomization** — Core technique for all repeating sounds: randomize which recording plays, its position, and timing. Prevents repetitive/static feeling in both UI sounds and environmental soundscapes.
- **Non-immersive space soundscapes** — Even windowed apps can use immersive soundscapes. The visionOS "hello" onboarding example combines a spatial foreground sound (tied to the animation) with a background ambient sound to fill the physical room — applicable to menus and title screens.

## APIs & Frameworks

This is a design and sound engineering session with no new Swift/Objective-C APIs announced. Key platform capability referenced:

- **Spatial Audio** — System-level capability on visionOS; available to developers through existing audio APIs
- **Spatial audio sources** — Point-in-space audio playback; can be placed at any 3D position in the user's environment (foreground, midground, distance)
- **Ambient background audio** — Surround audio files anchored to play all around the user; loop continuously; separate from spatial point sources
- **Room-aware reverberation** — System automatically applies room geometry-matched reverberation to spatial audio sources; no additional developer configuration needed
- **Professional audio libraries** — Recommended source for high-quality sound assets when custom field recording is not feasible

## Code Highlights

No code samples presented in this session. This is a design/sound engineering talk for developers and designers planning spatial audio experiences. Implementation uses existing AVFoundation and RealityKit audio APIs.

Key design decisions to encode in implementation:
- Place UI element sounds at the spatial position of the UI element (e.g., keyboard key presses come from the key's location)
- Use multiple recordings and vary pitch/amplitude slightly to avoid static repetition
- Combine point-source spatial audio with continuous surround ambient audio for full environmental soundscapes
- Use distance/volume to push background sounds into the perceptual background

## Takeaways

- Spatial audio on visionOS is rendered with room-awareness, meaning sounds naturally reverberate and localize to the user's physical space — this is a system capability that applies automatically to spatial audio sources placed in apps.
- Use randomization of pitch, amplitude, timing, and recording selection whenever a sound repeats; this is essential for both UI sounds (keyboard clicks) and environmental sounds (bird calls, ambient loops).
- Combine spatial audio sources (positional, point-in-space) with ambient background audio (surround, looping) to build convincing soundscapes; the layered approach works in both fully immersive spaces and regular windowed apps.
- Prioritize subtlety for UI sounds that users will hear repeatedly; sounds should confirm interactions and provide feedback without drawing undue attention.

---
_Source: WWDC23 Session 10271 page (abstract, transcript, and resource links)._
