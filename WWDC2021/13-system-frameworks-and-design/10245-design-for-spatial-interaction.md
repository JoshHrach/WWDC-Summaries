# Design for spatial interaction
**WWDC21 · Session 10245** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10245/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
This design-focused session from the Apple Design team explores the principles behind creating intuitive spatial interactions—experiences that involve two or more physical devices and leverage the U1 chip's ultra-wideband spatial awareness. Drawing on real features built for AirTag Precision Finding, HomePod mini music handoff, and the iOS share sheet, the presenters distill three core principles: adapt to distance and ability, provide continuous multimodal feedback, and embrace the physical action.

In iOS 15, Apple opens spatial interaction APIs to third-party accessories, making these principles directly applicable to any developer building products that pair with iPhone. The session is design guidance rather than code, but it closely informs how to structure UX for the Nearby Interaction framework.

## Key Topics

### Evolving Interaction Paradigms
- Spatial interactions represent a natural evolution from keyboards → mice → Multi-Touch → physical surroundings.
- The goal is to remove abstraction layers and let users interact more directly with their environment.
- Existing features (like the share sheet) can be enhanced with spatial awareness without requiring entirely new flows; spatial data (facing direction, proximity) intelligently surfaces the most likely target.

### Principle 1: Distance and Ability
- Design must adapt across a wide range of distances—from many meters to millimeters.
- Augment existing UI elements when approaching the appropriate distance (e.g., a "Directions" button becomes "Find Nearby").
- Be generous with angular tolerance when the user is far away; become more precise as they get close—users naturally narrow their aim as distance decreases.
- Switch the dominant feedback modality at close range: visual arrows become less useful; haptics become more precise.
- Accommodate varying human abilities by designing forgiving interactions that gently guide rather than demand precision.

### Principle 2: Continuous Multimodal Feedback
- Feedback must be immediate, continuous, and tightly coupled to physical movement so users perceive a direct cause-and-effect relationship.
- Types of feedback: visual UI changes, hardware lighting (HomePod top), coordinated cross-device feedback, audio, haptics.
- Use discrete spatial zones (e.g., two concentric rings around HomePod mini) to trigger progressive feedback stages: acknowledge → instruct → confirm.
- Visual feedback should animate in a way physically related to the user's motion (e.g., a banner scaling with distance from HomePod).
- Haptic intensity should increase as proximity increases; use crisp, discrete haptics to mark key moments (arrow locking on target).
- Natural pull-away gestures should cancel interactions without requiring additional onscreen buttons—make intent readable from body movement alone.
- Sound and haptics should have a clear, repeatable cause-and-effect mapping; avoid overuse.

### Principle 3: Embracing the Physical Action
- Design UI to be usable peripherally—the user's attention is on the real world, not the screen.
- Use large, bold typography and high-contrast color changes readable in peripheral vision (AirTag distance readout, arrow UI).
- Sound from the target device (AirTag beep, music from HomePod) is the primary confirmation of a successful interaction.
- Defer to the physical action and the concept of "this" vs. "that"—pointing a device at a HomePod is more natural than navigating an AirPlay list.
- Bold UI choices also improve accessibility for users with varying abilities.

### AirTag Precision Finding UX
- Idle state: rotating dots imply readiness while searching for the AirTag signal.
- Connected state: distance readout appears, large arrow points to AirTag.
- Facing confirmation: screen "lights up" boldly + haptic snap when aligned with AirTag direction.
- Arm's reach: arrow zooms into pulsing dot; haptic becomes continuous and shifts with hand position; sound is not used here (haptic is more precise).
- Lost signal: arrow disassembles to communicate the loss.

### HomePod Mini Music Handoff UX
- Two proximity zones around HomePod mini drive progressive feedback.
- Zone 1 entry: onscreen banner appears, iPhone haptic acknowledges; HomePod top light activates.
- Zone 1–2: banner scales with distance, haptics increase; pulling back cancels.
- Zone 2 entry: music transfers; HomePod light confirms; music playing from speaker is the ultimate confirmation.
- Both devices give feedback simultaneously, dividing attention between the two.

## APIs & Frameworks

**Nearby Interaction** framework
- `NISession` — manages a spatial interaction session between two U1-capable devices **[existing, expanded to third-party accessories in iOS 15]**
- `NINearbyObject` — provides distance and direction data from a peer device **[existing]**
- `NINearbyObject.distance` — distance in meters to the peer **[existing]**
- `NINearbyObject.direction` — 3D unit vector pointing toward peer (requires U1) **[existing]**
- Third-party accessory support — `NISession` can now interact with developer-built U1 hardware via the new accessory configuration API **[NEW in iOS 15]**
- `NIConfiguration` / `NINearbyAccessoryConfiguration` — configures sessions with Apple devices or third-party accessories **[NEW for accessories]**

**UIKit / Human Interface**
- Large, bold typography for peripheral-vision-readable distance/direction UI
- `UIFeedbackGenerator` subclasses (`UIImpactFeedbackGenerator`, `UISelectionFeedbackGenerator`) — haptic feedback delivery **[existing]**
- Proximity-driven UI scaling and modality (visual blur backgrounds, banner scaling)

**Human Interface Guidelines**
- [Nearby Interactions HIG](https://developer.apple.com/design/human-interface-guidelines/nearby-interactions) — referenced directly in session resources

## Code Highlights

No code samples were presented in this design session. Implementation details are covered in:
- "Meet Nearby Interaction" (WWDC20, Session 10668)
- "Explore Nearby Interaction with third-party accessories" (WWDC21, Session 10165)

## Takeaways
- Spatial interactions should adapt their dominant feedback modality by distance: visuals at range, audio at mid-range, haptics at close range.
- Always provide continuous feedback tightly linked to physical motion—discrete, button-press confirmation feels wrong for physical-world interactions.
- Design UI to be legible in peripheral vision (large type, bold color) since the user's gaze is directed at the real world, not the screen.
- Natural pull-away gestures should cancel without extra UI; embrace body language as an interaction input.

---
_Source: WWDC21 Session 10245 page (abstract, full transcript, and resource links)._
