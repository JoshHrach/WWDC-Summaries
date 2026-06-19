# Designing Award Winning Apps and Games
**WWDC19 · Session 802** · [Watch](https://developer.apple.com/videos/play/wwdc2019/802/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session examines the design values behind the 2019 Apple Design Award winners, drawing on behind-the-scenes stories, design artifacts, and developer interviews. Rather than listing selection criteria, the talk surfaces six design qualities shared by all award-winning apps and games: Innovation, Trust, Refinement, Aesthetics, Inclusion, and Attention to Detail. Each quality is illustrated through one or two real-world examples — Asphalt 9, HomeCourt, Pixelmator Photo, The Gardens Between, Butterfly iQ, Thumper, ELOH, Ordia, and Flow by Moleskine.

The session is an inspiration and process guide for developers and designers. It demonstrates that great design involves questioning assumptions, simplifying ruthlessly, iterating over long periods, revealing system intelligence to users, designing for everyone from the start, and sweating every detail — even those invisible to most users.

## Key Topics

- **Innovation — Question assumptions** — Asphalt 9's TouchDrive replaced traditional acceleration/braking/turning with swipe-based path selection and nitro/drift tap-and-hold. The team started from scratch, challenged every fundamental assumption about racing controls, ran rounds of testing, and discovered a new way to play that made the game more accessible and strategic.
- **Innovation — Simplify to what matters** — HomeCourt (basketball ML training app) began requiring a tripod, sunny day, and manual court setup. Iteratively, the team automated hoop detection, three-point-line detection, leg detection, full pose estimation, and finally ground-level phone placement — removing every obstacle until the experience felt like magic.
- **Trust — Reveal system intelligence** — Pixelmator Photo's ML-powered Magic Wand adjusts photos based on 20 million professional photos, but shows every individual parameter change (exposure, sky, skin tone, etc.) animating in the inspector, so users understand cause and effect. Visible attribution builds trust; hidden magic creates doubt.
- **Refinement — Explore widely before converging** — The Gardens Between went through fairy-tale visual styles, dramatically different character designs, and many control schemes before arriving at its final dream-like form. The artifacts showed creative exploration is essential to reaching the best idea.
- **Refinement — Don't copy-paste real-world metaphors** — Butterfly iQ (portable ultrasound app) initially mirrored physical cart controls with spinning dial metaphors. Testing revealed doctors didn't need persistent dials; they drew inspiration from the iOS Camera app and replaced dials with swipe gestures (horizontal for contrast, vertical for depth), freeing up screen real estate for the ultrasound image.
- **Aesthetics — Immersion through simplicity** — Thumper removes all non-essential UI: the pause button fades to a slash, health is shown on the beetle itself, scores only appear at checkpoints. Designing controls specifically for iOS touch ensures players stay present.
- **Aesthetics — Style serves gameplay** — ELOH hired an illustrator, then a sound designer, ensuring visual style enhances interaction clarity and emotional connection. The pacing deliberately includes easy levels after hard ones so the aesthetic craft resurfaces.
- **Aesthetics — Cohesion through consistent design language** — Ordia's organic blobby aesthetic is carried through menus, transitions, characters, and every screen. Using the iOS Share icon (platform-native element) signals the app is at home on the platform.
- **Inclusion — Native components and colorblind support** — Pixelmator Photo chose native iOS UISwitch (which supports Accessibility labels) over Mac-style checkboxes. Ordia built a colorblind mode by simulating multiple types of colorblindness on screenshots, then rebalancing colors; they also tested in pure monochrome.
- **Attention to Detail — Prototype fundamentals first** — Flow by Moleskine built interaction prototypes using plain text buttons to nail gesture behavior before any visual design. Only after interaction was perfected did they add visual refinement, animation, and 1,400+ unique color names.

## APIs & Frameworks

This session is design-focused with no code samples. Relevant platform APIs referenced:

- Machine Learning / Vision framework — HomeCourt (hoop detection, pose estimation), Pixelmator Photo (segment-aware photo adjustment)
- ARKit — (referenced in context of camera-based experiences)
- Accessibility APIs — `UISwitch` with accessibility labels (`accessibilityLabel`), colorblind-accessible color design; `UIAccessibility.isReduceTransparencyEnabled`, `UIAccessibility.isDarkerSystemColorsEnabled`
- UIKit native components — `UISwitch`, Share sheet (`UIActivityViewController`) for platform-native affordances
- Core Haptics / UIFeedbackGenerator — Ordia's haptic-enhanced world interaction
- Human Interface Guidelines — App icons (HomeCourt's H+C court-line composition), Accessibility, Designing for iOS/macOS/watchOS/tvOS

## Code Highlights

No code samples. This is a design and process session.

## Takeaways

- Question every assumption in your design: the "obvious" way to implement a control (racing game steering, ultrasound cart dials) is often not the best way for touch interfaces — iterate, test, and be willing to discard ideas that seemed like obvious wins.
- Reveal ML and system intelligence visibly: when automated adjustments happen, animate and expose each individual parameter change so users can understand, verify, and override. Transparency builds trust; opacity creates anxiety.
- Design for refinement over time: the final award-winning design rarely resembles the initial concept. Create design artifacts, explore multiple directions simultaneously, and allow ideas to develop before converging.
- Inclusion is not a post-launch addition — designing with native components, testing for colorblindness, and sweating accessibility from the beginning allows more people to use and love your app.

---
_Source: WWDC19 Session 802 page (abstract, full transcript, and resource links)._
