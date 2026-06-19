# Design Tips for Great Games
**WWDC17 · Session 811** · [Watch](https://developer.apple.com/videos/play/wwdc2017/811/)

_Platforms:_ iOS 11, tvOS 11

## Overview
This session delivers ten actionable design tips covering the full arc of the game experience — from first launch through onboarding, in-game UI, tutorials, and accessibility. The session argues that everything a player experiences before, between, and around the actual gameplay is part of the product, and that small design choices (a themed first screen, a responsive button state, a well-placed Skip button) have an outsized impact on retention and enjoyment.

The presenter draws on real App Store titles (Pokémon GO, Don't Starve Pocket Edition, INKS, Splitter Critters, Reigns, Zombie Gunship, Blackbox, KAMI 2) to illustrate both anti-patterns (bouncing arrows, floating hands, repeated tutorial directions) and best practices (edge-to-edge onboarding art, progress bars instead of spinners, learn-through-play tutorials). The final tip emphasizes inclusivity and accessibility as core game design responsibilities, not afterthoughts.

## Key Topics

1. **First launch** — the first screen should be styled to match the game world; use edge-to-edge illustrations that communicate the game's tone before any interaction.
2. **Loading** — disguise load time by entertaining (animated sequences) or educating (gameplay tips); always use a progress bar, not an activity spinner; avoid repeating tips within the same session.
3. **Quick start** — minimize the number of screens between launch and gameplay; the primary call-to-action (Play/Start/Battle) must be the most visually dominant element on the Main Menu.
4. **UI consistency** — layout controls in the same positions across all screens; audit all screens side-by-side to confirm Back/Settings buttons don't jump around; inconsistency forces players to re-map your interface on every screen.
5. **UI clarity** — interactive controls must look different from status/decorative elements; all interactive controls need three explicit visual states: Normal, Pressed, and Disabled; meet the 44×44 pt minimum tap target on iOS; never hide a disabled control — use a visual Disabled state instead.
6. **Tutorials — progressive disclosure** — teach only the three moves in the core game loop; defer economy, upgrades, tournaments, and store to post-tutorial discovery; keep each direction to a single, short sentence.
7. **Tutorials — creative teaching** — avoid bouncing arrows and floating hands (they obscure the UI they're teaching); use animation, motion, and in-world cues instead; teach before the game begins when possible (Splitter Critters approach).
8. **Learn through play** — the most effective tutorial is dropping players into the game with intuitive controls and letting them discover the objective through natural exploration (Mars: Mars example).
9. **Skip controls** — reveal a Skip button on tap (not persistently) during intro sequences; always provide a Skip Tutorial button; ensure players can replay the tutorial later; honor that players may have varying skill levels or limited time.
10. **Accessibility and inclusivity** — support Closed Captions, Dynamic Type, left/right-hand mode, color-blind mode, and sonic interfaces for blind/low-vision players; 285 million people worldwide have blind or low-vision disabilities.

## APIs & Frameworks

This is a game design session with no code content. Relevant Apple system features referenced:

- **Dynamic Type** — `UIFontMetrics`, `UIFont.preferredFont(forTextStyle:)` — supporting scalable text in game UI
- **Closed Captions / Subtitles** — system caption support via AVFoundation / AVKit
- **UIAccessibility** — accessibility labels, traits, and notifications for inclusive UI
- **Color contrast / Color Blind Mode** — designing with sufficient contrast; offering color-blind-friendly palettes
- **Minimum tap targets** — 44×44 pt minimum interactive element size (HIG requirement)
- **UIButton states** — `.normal`, `.highlighted`, `.disabled` — explicit visual states for all interactive controls
- **UIProgressView** — recommended over `UIActivityIndicatorView` for load progress
- **Human Interface Guidelines for Games** — `developer.apple.com/design/`

Games highlighted as positive examples: INKS (Apple Design Award 2016), Splitter Critters, Reigns, Zombie Gunship, Blackbox, KAMI 2, Don't Starve Pocket Edition, Clash Royale, Mars: Mars, Pokémon GO, Icycle: On Thin Ice, Mushroom 11.

## Code Highlights

No code samples presented. The session is a design lecture with visual examples.

Key UIKit implementation notes implied by the tips:

- Use `UIButton.setBackgroundImage(_:for:)` for all three states: `.normal`, `.highlighted`, `.disabled`.
- Size all `UIButton` and custom gesture regions to at least 44×44 points regardless of visual glyph size.
- Use `UIProgressView` during asset loading rather than `UIActivityIndicatorView`.
- Support `UIAccessibility.isReduceMotionEnabled` and `UIAccessibility.isReduceTransparencyEnabled` in game animations.

## Takeaways

- Every screen before gameplay is part of the first impression — style it to match the game world immediately, including legal/accept screens.
- UI consistency (predictable control placement) and clarity (obvious interactive vs. decorative elements, three button states, 44 pt tap targets) are the two most common game UI failure modes.
- Tutorials are most effective when they teach the minimal core loop through play rather than explicit instruction with arrows or floating hands.
- Accessibility is not optional — color-blind mode, caption support, left/right-hand mode, and sonic interfaces each open the game to a substantially larger audience.

---
_Source: WWDC17 Session 811 page (abstract, transcript, and resource links)._
