# Iterative UI Design
**WWDC16 · Session 805** · [Watch](https://developer.apple.com/videos/play/wwdc2016/805/)

_Platforms:_ iOS 10, macOS Sierra 10.12

## Overview
Two designers from Apple's iWork team (who make Numbers, Pages, and Keynote) share their process for moving from a vague product idea to a well-defined, well-designed iOS app. The session is organized around three fundamental design questions: what are we making, where do we start, and what is the right design? It is notably practical rather than theoretical, with live Keynote demos showing exactly how to build high-fidelity UI mockups and interactive prototypes without any specialized design software.

Keynote is positioned throughout as a first-class UI design tool — the session notes that Keynote itself was designed in Keynote — capable of producing pixel-perfect mockups, real device dimensions, and interactive prototypes that can be loaded directly onto an iPhone.

## Key Topics

### Defining Your App
- Start by listing every feature you can imagine, then set it aside temporarily.
- Identify your target audience by articulating 3–5 essential characteristics (e.g., "wants a quick, fresh, healthy meal every day"). You are not the user; designing for yourself introduces bias.
- Convert audience characteristics into **customer goals** (e.g., "enjoy a fresh meal," "eat quickly," "try something new").
- Define **app goals** — the qualities of experience you want users to have (e.g., "highlight daily entrees," "provide a convenient delivery service"). App goals differ from business goals (profitability, ROI), which do not provide design direction.
- Map your feature list against customer goals and app goals to keep features that serve both and defer those that don't.

### Where to Start: Drawing UI in Keynote
- Start with what you know — one or two screens you're certain the app needs (e.g., a menu screen for a food delivery app).
- Use Keynote as a UI design tool:
  - Set canvas size to match device resolution (375 × 667 points for iPhone 6).
  - Use screenshots of existing apps as color/size/typography references; use the eyedropper color picker to match exact colors.
  - Use shapes, lines, text boxes, and image masks to assemble realistic screen layouts.
  - Duplicate and use "distribute spacing" to position repeated elements quickly.
  - Use `Command-Control-Shift-4` to screenshot specific screen regions for UI collaging.
  - Zoom in with Accessibility Zoom to ensure pixel-perfect alignment.
- Use real, believable content (realistic food photos, actual menu copy) to make mockups more evaluable.
- A screen can be drawn in ~3 minutes with practice.

### Iterating: Generating Design Alternatives
- Never evaluate only one idea; generate multiple alternatives before judging.
- Techniques for exploring variations of a single screen:
  - Change row/card height and information density.
  - Adjust white space (margins, padding) and typography (weight, size, color).
  - Switch between list, grid, and card layouts.
  - Do a "180" — if the design is text-heavy, try an image-heavy version, and vice versa.
  - Push ideas to extremes (full-bleed hero photos, no photos at all).
  - Combine elements from different alternatives.
  - Consider alternative navigation paradigms (vertical scroll vs. horizontal swipe).
- The goal at this stage is quantity of ideas, not quality — do not judge during generation.

### Critiquing: Finding the Right Design
- Set aside ego and time invested; evaluate as a neutral third party.
- Questions to apply to each alternative:
  - Does it serve the audience's goals?
  - Does it meet the app goals?
  - Does it provide the right information and enough detail to act?
  - Is it readable and accessible (consider color blindness, vision impairment)?
  - Is it intuitive (familiar controls and terminology)?
  - Does it feel right — personality and aesthetic?
- Run elimination rounds, cutting obviously weaker designs first, then comparing survivors in detail.
- Involve teammates or, as in the live demo, the audience — disagreement forces articulation of reasoning.

### Designing Workflows
- A workflow is the sequence of steps to complete a task (e.g., order a meal).
- Apply the same start-iterate-critique loop to workflows as to individual screens.
- Common pitfalls in initial workflow designs: missing key information (delivery time/address), abrupt commitment (ordering without confirmation), too many steps for a simple task.
- Refine by questioning each step: can the user accomplish this earlier or with fewer taps? Can optional steps be deferred behind progressive disclosure?

## APIs & Frameworks

This is a design process session — no iOS APIs are discussed. Tools and techniques referenced:

- **Keynote** — primary prototyping tool; supports shapes, masks, text, images, animations, interactive builds, and device-resolution canvases
- iPhone device canvas: 375 × 667 points (iPhone 6 / 6s)
- macOS Accessibility Zoom — pixel-perfect alignment during mockup work
- `Command-Control-Shift-4` keyboard shortcut — screenshot selection to clipboard
- Keynote eyedropper color picker — match exact colors from reference screenshots
- Keynote "Distribute Objects" — evenly space repeated elements
- Keynote interactive prototypes — tap-to-advance, magic move, and other transitions for on-device testing

## Code Highlights

No code. The session focuses entirely on design methodology and Keynote prototyping technique.

Workflow design example (ordering food, final iteration — 3 visible steps):
1. Menu screen with inline "Buy" button per item.
2. Slide-up tray showing order summary + Place Order button (optional: expand tray to full detail screen).
3. Confirmation state shown in same tray.

## Takeaways
- Define the app before drawing anything: customer goals and app goals act as a filter for features and design decisions throughout the entire project; without them, design choices are arbitrary.
- Generate many alternatives (aim for 10+) before evaluating any — you cannot know if a design is right if you only tried one idea.
- Keynote is a capable, fast UI prototyping tool that requires no specialized training: shapes, masks, duplicate-and-distribute, and the eyedropper picker are enough to produce high-fidelity, pixel-perfect mockups in minutes.
- Critique designs against concrete audience and app goals, not personal preference; involve others to force clear articulation of reasoning.

---
_Source: WWDC16 Session 805 page (abstract, transcript, and resource links)._
