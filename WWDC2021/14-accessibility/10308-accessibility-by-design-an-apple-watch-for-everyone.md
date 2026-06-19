# Accessibility by Design: An Apple Watch for Everyone
**WWDC21 · Session 10308** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10308/)

_Platforms:_ watchOS 8

## Overview
This session is a narrative documentary told by Apple engineers and designers who built accessibility features into Apple Watch. Rather than a technical walkthrough, it presents Apple's design philosophy: accessibility is not a separate track but is woven into the product creation process from the start. The session covers how features built for people with disabilities often become universally valued.

The four broad accessibility categories Apple designs for are vision, cognitive, motor, and hearing. The team shares specific examples — from Taptic Time and VoiceOver-enhanced watch faces to the brand-new AssistiveTouch gesture system — illustrating how deeply user research and community engagement drive feature development.

A central theme is that building excellent accessibility features benefits everyone: large-type watch faces designed for low-vision users are available to all; the contrast-heavy black backgrounds that help blind users also improve legibility for everyone; Mickey Mouse's spoken time, developed for VoiceOver users, became beloved by sighted users and children alike.

## Key Topics

### Vision Accessibility on Apple Watch
- High-contrast black backgrounds and legible letterforms benefit low-vision users and all users
- Large-type watch face designed for low-vision was made available to all
- A user with Usher syndrome (tunnel vision) found the small screen advantageous — central-only vision focused well on the compact Watch display

### Taptic Time
- Allows users to feel the time using haptic patterns
- Designed so blind users don't need to stop walking or remove a hand from a mobility aid
- Enabled by "raise to speak" / Wrist Detection workflow: raise wrist, hear the time

### Mickey Mouse Watch Face and VoiceOver
- Original Mickey watch face only read the time aloud in generic text-to-speech for VoiceOver users
- Engineers partnered with Disney to record Mickey Mouse's actual voice reading the time in 30+ languages
- VoiceOver users hear Mickey laugh more frequently; non-VoiceOver users hear less — intentional tuning based on user research
- Example of an accessibility investment producing a beloved mainstream feature

### AssistiveTouch — Gesture Control Without Touching the Screen
- Designed for users with upper-limb differences (missing arm, prosthetic, low muscle tone)
- Uses accelerometer, gyroscope, and optical heart-rate sensor to detect subtle muscle movements
- Machine-learning model distinguishes: pinch, double pinch, clench, double clench
- Provides complete access to Apple Watch UI without needing a second hand
- SF Symbols expanded with a glyph language representing all Apple Watch hardware and software inputs **[NEW]**

### Accessible Design Principles
- Start with the product's core purpose, then ask how each disability category intersects
- Iterate through hardware constraints (Watch CPU "size of a postage stamp")
- Include community members with disabilities throughout testing — e.g., colleague with low muscle tone shaped final gesture set
- Invest in accessibility because it creates equity and independence, and often leads to mainstream innovations

## APIs & Frameworks

This session is a design/engineering story session with no code samples. Key platform features discussed:

- **Taptic Time** — system watchOS accessibility feature; haptic time reading via wrist raise
- **VoiceOver** — system screen reader; enhanced with character voice integration (Mickey Mouse) in watchOS
- **AssistiveTouch** **[NEW]** — new watchOS 8 gesture-based input system using motion/health sensors + machine learning
  - Gestures: pinch, double pinch, clench, double clench
  - Input via: accelerometer, gyroscope, optical heart-rate sensor
- **Large Type watch face** — system watch face designed for low-vision users
- **SF Symbols** — extended with Apple Watch hardware/software input glyphs **[NEW]**
- **Wrist Detection / Raise to Speak** — automatic time announcement on wrist raise

## Code Highlights

No code samples in this session (design and engineering narrative format).

## Takeaways
- Accessibility is most effective when built in from the start of product design, not added afterward — Apple Watch exemplifies this with features like Taptic Time and high-contrast faces that became universal.
- AssistiveTouch (watchOS 8) is a breakthrough motor accessibility feature that uses on-device ML to detect muscle movements via the Watch's existing sensors, enabling full UI control without touching the screen.
- Features created for accessibility frequently become beloved by all users — Mickey Mouse's spoken time (built for VoiceOver) and large-type watch faces are prime examples.
- Engaging real users with disabilities throughout development — not just at testing — is critical to building features that genuinely work.

---
_Source: WWDC21 Session 10308 page (abstract, chapter summaries, and resource links)._
