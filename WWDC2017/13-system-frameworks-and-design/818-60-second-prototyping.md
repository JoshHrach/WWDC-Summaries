# 60-Second Prototyping
**WWDC17 · Session 818** · [Watch](https://developer.apple.com/videos/play/wwdc2017/818/)

_Platforms:_ iOS, macOS (design process; tool-agnostic)

## Overview
This session teaches developers and designers how to build rapid, interactive prototypes using everyday tools — Keynote, Xcode/Swift, HTML/CSS/JS, Sketch, or even a word processor — in as little as 60 seconds. The core argument is that diving into code or high-fidelity design assets too early locks teams into specific solutions before they have had a chance to explore alternatives.

Prototyping serves two main goals: testing ideas (some great ideas turn out not to work, and some weak ideas have unexpected value) and generating new ideas through the act of building and showing. A short feedback loop — build a prototype, show it to people in realistic contexts on real devices, learn from feedback, iterate — helps teams confirm they are building the right thing before committing significant engineering or design effort.

The session uses a real-world Timer app example to walk through a complete prototyping cycle: creating the first version in Keynote using basic shapes and animations, testing with real users performing actual tasks, gathering concrete feedback, and rapidly building an improved second iteration that addresses the issues found.

## Key Topics

- **Why prototype before coding or high-fidelity design** — committing too early forecloses exploration of alternatives; prototypes are interactive in a way static visuals are not.
- **Tools for prototyping** — Keynote (demonstrated live), Xcode/Swift, HTML/CSS/JavaScript, Sketch, Illustrator; anything that displays images and responds to user interaction qualifies.
- **Building a Keynote prototype** — sizing the canvas to match an iPhone screen, using basic shapes, leveraging Keynote's rotation-around-center animation trick to simulate a clock hand, chaining transitions/slides to create interactive flows.
- **Testing in context** — showing prototypes on the actual target device in realistic usage scenarios (e.g., timing a Rubik's Cube solve, using a timer while doing a chemistry experiment); context reveals problems invisible in a vacuum.
- **Gathering and applying feedback** — not defending or dismissing feedback, identifying what works vs. what does not, deciding whether to refine or try something entirely different.
- **Team diversity** — having teammates with varied backgrounds surfaces use cases (e.g., a timer for children brushing teeth) that individuals would never discover alone.
- **Iterating rapidly** — making quick changes (enlarging a tap target by removing a small button and making the whole lower region tappable, adding a finish screen) and cycling through multiple distinct prototypes in parallel.

## APIs & Frameworks

This session is design-process focused and does not cover specific Apple APIs or frameworks. Tools referenced:

- **Keynote** — used as the primary prototyping tool (shapes, animations, slide-to-slide navigation, rotation animations)
- **Xcode / Swift** — mentioned as a developer prototyping option
- **HTML / CSS / JavaScript** — mentioned as a cross-platform prototyping option
- **Sketch / Illustrator** — mentioned as designer-oriented prototyping tools
- **Apple Design Site** — linked as a resource (`developer.apple.com/design/`)

## Code Highlights

No code samples are presented in this session. The demonstration is entirely in Keynote. The key technique shown is:

- Copy-pasting a clock-hand shape, offset its pivot by moving the copy to the bottom of the desired rotation arc, group the two objects, and then apply a Keynote rotation animation — because Keynote rotates around a group's center, this makes the hand appear to rotate correctly around the clock face.
- Adding slide-advance animations triggered by taps to simulate button interactions.

## Takeaways

- Build prototypes before writing production code or polished assets — test the idea first, then build it right.
- Use tools you already know; anything interactive counts as a prototyping tool.
- Always test on the real device in a realistic context; lab-clean demos hide real-world friction.
- Rapid iteration cycles (prototype → show → learn → repeat) surface the right product direction faster and cheaper than any other method.

---
_Source: WWDC17 Session 818 page (abstract, transcript, and resource links)._
