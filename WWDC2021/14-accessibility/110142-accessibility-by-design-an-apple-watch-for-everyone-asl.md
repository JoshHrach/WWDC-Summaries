# Accessibility by Design: An Apple Watch for Everyone (ASL)
**WWDC21 · Session 110142** · [Watch](https://developer.apple.com/videos/play/wwdc2021/110142/)

_Platforms:_ watchOS 8

## Overview
This session is the American Sign Language (ASL) version of Session 10308 — "Accessibility by Design: An Apple Watch for Everyone." The content is identical: Apple engineers and designers share the stories behind Apple Watch's accessibility features, illustrating how designing for disabled users produces benefits for all users.

The four accessibility categories Apple designs for are vision, cognitive, motor, and hearing. The session covers specific Apple Watch features including Taptic Time, high-contrast watch faces, the Mickey Mouse watch face with recorded character voice (VoiceOver integration), and the new AssistiveTouch gesture system introduced in watchOS 8.

The ASL presentation format itself is an example of Apple's accessibility commitment — making technical developer content accessible to Deaf and hard-of-hearing developers.

## Key Topics

### Vision Accessibility on Apple Watch
- High-contrast black backgrounds benefit low-vision users and all users
- Large-type watch face designed for low-vision, available to all
- Usher syndrome user case study: small focused display better for tunnel vision than a large screen

### Taptic Time
- Feel the time through haptic patterns on wrist raise — no need to stop or touch screen
- Critical for users who rely on mobility aids (cane, wheelchair)
- Enabled by Wrist Detection / Raise to Speak interaction

### Mickey Mouse Watch Face and VoiceOver
- Partnered with Disney to record actual Mickey Mouse voice in 30+ languages for VoiceOver users
- VoiceOver mode: Mickey laughs more frequently; non-VoiceOver: reduced laughter frequency
- Accessibility feature became beloved by all users including sighted adults and children

### AssistiveTouch — Gesture-Based Control
- New in watchOS 8; designed for users with upper-limb differences
- Four gestures: pinch, double pinch, clench, double clench
- Uses accelerometer, gyroscope, and optical heart-rate sensor + on-device ML model
- Provides complete Apple Watch access without touching screen or needing a second hand
- SF Symbols extended with glyphs for all Apple Watch hardware/software inputs **[NEW]**

### Accessible Design Philosophy
- Accessibility by design, not by retrofit — starts at product conception
- Community engagement and real-user testing throughout development
- Accessible features frequently become mainstream features

## APIs & Frameworks

This session is a design and engineering narrative (ASL version). No code samples. Platform features:

- **AssistiveTouch** **[NEW]** — watchOS 8 gesture-input system using motion/health sensors + ML
  - Gestures: pinch, double pinch, clench, double clench
- **Taptic Time** — watchOS system accessibility feature; haptic time readout
- **VoiceOver** — system screen reader; enhanced with character voice (Mickey Mouse)
- **Large Type watch face** — system watch face for low-vision users
- **SF Symbols** — expanded with Apple Watch input glyphs **[NEW]**

## Code Highlights

No code samples (design/engineering narrative session with ASL interpretation).

## Takeaways
- This is the ASL-interpreted version of Session 10308; all technical content is identical to that session.
- AssistiveTouch (watchOS 8) demonstrates how on-device ML and existing Watch sensors can create entirely new input modalities for motor-impaired users.
- The session itself — a technical WWDC session delivered in ASL — models Apple's commitment to making developer education as accessible as its products.
- Accessible-by-design thinking produces universal benefits: Taptic Time, high-contrast faces, and Mickey's voice all started as accessibility features and became beloved by all users.

---
_Source: WWDC21 Session 110142 page (abstract, ASL video, and resource links). ASL version of Session 10308._
