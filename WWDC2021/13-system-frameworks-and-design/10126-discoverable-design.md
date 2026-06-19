# Discoverable Design
**WWDC21 · Session 10126** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10126/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
This session presents five design principles for building apps that people can understand and navigate intuitively—without relying on heavy onboarding tutorials. Using a fictional recipe app called "Toasty," the speakers demonstrate how to structure navigation, visual cues, gestures, content organization, and personalization controls so that discoverability is built into the interface itself.

The talk covers the psychology behind why reading instruction screens rarely works and argues that the interface itself should teach users in context. Every principle is illustrated with concrete before/after examples showing common anti-patterns (hamburger menus, overly minimal UIs, unlabeled icons) and how to fix them.

A second half of the session, presented by a second designer, extends discoverability to content-heavy apps by examining category organization, ML-powered personalization, and the design of explicit and implicit recommendation feedback controls.

## Key Topics
- **Principle 1 – Prioritize important features:** Rank features by importance and frequency of use. Place primary actions in the tab bar; avoid hamburger menus that hide functionality behind an opaque icon. Balance minimalism against findability.
- **Principle 2 – Provide visual cues:** Use labels alongside icons to remove ambiguity. Address the "blank page problem" with placeholder examples (e.g., suggested search terms). Teach in context with animation and progressive disclosure rather than upfront onboarding screens.
- **Principle 3 – Hint at gestures:** Prefer standard iOS gestures (tap, swipe, long press, pan, pinch, rotate). Treat gestures as shortcuts—always provide a visible primary control. Use animated transitions that are directionally consistent with the gesture that triggers them. Peek at adjacent content to hint swipe navigability.
- **Principle 4 – Organize by behavior:** Categorize content to match users' real-world motivations and context rather than arbitrary taxonomies (e.g., restaurant picks vs. friend recommendations vs. personalized "Tasty Picks"). Use clear section headers and visual groupings to make category divisions scannable at a glance.
- **Principle 5 – Convey a sense of control:** Design explicit feedback controls (thumbs up/down with descriptive labels like "Suggest toast like this") so users know they are influencing a recommendation engine. Disclose implicit feedback signals with explanatory labels ("Because you added avocado toast"). Provide easy-to-find edit/remove controls for personalized content.

## APIs & Frameworks
This is a design/HIG session with no code. The concepts apply across all UIKit and SwiftUI interfaces.

- **Human Interface Guidelines** – Tab bar navigation, hamburger menu anti-pattern, standard gesture set
- **iOS standard gestures:** `UITapGestureRecognizer`, `UISwipeGestureRecognizer`, `UILongPressGestureRecognizer`, `UIPanGestureRecognizer`, `UIPinchGestureRecognizer`, `UIRotationGestureRecognizer`
- **Haptic feedback** – Used contextually to indicate a recommended moment to act (e.g., camera framing confirmation)
- **ML recommendation engine pattern** – Personalized content section powered by on-device or server-side recommendation, surfaced with disclosure labels
- **Scribble / Apple Pencil** – Referenced as an example of a real-world gesture analogy (scratch-out to delete)
- **Tab bar** (`UITabBarController` / SwiftUI `TabView`) – Preferred navigation pattern over hidden menus
- **Navigation controller / back chevron** (`UINavigationController` / SwiftUI `NavigationView`) – Standard back-navigation pattern users recognize
- **Camera / AVCaptureSession** – Referenced in context of the "scan toast" onboarding flow using animated framing hints

## Code Highlights
No code samples are presented. The session is design-focused.

## Takeaways
- Never hide important features behind opaque icons like the hamburger menu; put high-frequency actions in persistent navigation like the tab bar.
- Icons almost always benefit from text labels—even familiar symbols like a camera or heart are ambiguous without them.
- Gestures should be accelerators, not the only path; pair every gesture with a visible tap target and animate transitions so the direction of the animation reveals the gesture.
- When personalizing content with ML, always tell users why they're seeing content ("Because you added avocado toast") and make it easy to edit or remove those inputs with clearly labeled controls.

---
_Source: WWDC21 Session 10126 page (abstract, chapter summaries, code samples, and resource links)._
