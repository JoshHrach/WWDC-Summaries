# Communication Between Designers and Engineers
**WWDC17 · Session 809** · [Watch](https://developer.apple.com/videos/play/wwdc2017/809/)

_Platforms:_ iOS, macOS (process and tooling; platform-agnostic)

## Overview
This short, practical session targets both designers and engineers and identifies four concrete improvements that reduce friction in cross-functional collaboration. The core observation is that designers and engineers often work toward the same goal but create needless rework when they use different terminology for the same UI elements, lack a single canonical source for approved assets, skip up-front use-case analysis in a rush to differentiate, or hand off static specs instead of dynamic demonstrations.

The session frames good design-engineering collaboration as building a shared vocabulary, shared tools, and shared empathy. It encourages engineers to open Xcode and explore UIKit controls as a way to understand what designers are working toward, and it encourages designers to try Xcode's asset catalog and storyboard features (without writing code) to understand how their assets will actually be implemented.

The four pillars presented — terminology, single source of truth, thoughtful focus, and show-and-tell — are supported by practical team activities, not just principles, making the advice immediately actionable.

## Key Topics

- **Shared terminology** — many common UIKit controls have official names (`UIAlertController`/Alert, `UINavigationBar`, `UIToolbar`, `UISwitch`) that teams frequently replace with informal labels (modal, header, toggle), causing silent translation errors; activities include a 30-minute team postmortem focused on terms, a Pictionary-style flash-card game, or a breakdown report.
- **Single source of truth** — agreeing on one approved location for mockups, strings, and assets before work begins; noting that designers lack the universal version-control equivalents engineers have; tools should integrate with both sides' workflows.
- **Thoughtful focus on use cases and accessibility** — starting with standard SDK patterns (e.g., `UITableView` with a disclosure indicator) before customizing; writing out all intended and edge-case use cases together; baking accessibility requirements in from the start rather than bolting them on later.
- **Show and tell with dynamic prototypes** — replacing static mockup hand-offs with video recordings of animations, referencing exact UIKit animation parameters, or small interactive prototypes built in Principle, Flinto, or Keynote; showing animation curves alongside API reference and a video ensures implementation matches intent.
- **Engineers learning design tools / designers learning Xcode** — engineers gaining empathy by observing design workflows; designers gaining credibility by importing assets into an Xcode asset catalog, exploring storyboards, and using the Object Library to see built-in control states and properties without writing code.
- **Face-to-face review sessions** — short in-context review sessions (designer or engineer teammate while the project is still open); reduces errors before code is checked in.

## APIs & Frameworks

This is a design-process session with no new API introductions. Existing UIKit elements referenced by their correct names:

- `UIAlertController` — the correct term for Alert/ActionSheet patterns
- `UINavigationBar` — correct term for the top navigation area
- `UIToolbar` — correct term for the bottom action bar
- `UISwitch` — correct term for on/off toggle controls
- `UITableView` — recommended starting point for list UIs, with built-in accessibility support
- **Xcode Asset Catalog** — for delivering app icons and image assets at correct resolutions
- **Xcode Storyboard / Interface Builder** — Object Library shows real control names, states, and properties without requiring code
- **Human Interface Guidelines** (`developer.apple.com/design/`) — canonical reference for standard UIKit terminology
- **API Reference Documentation** (`developer.apple.com`) — authoritative source for control and component names
- **Prototyping tools mentioned**: Principle, Flinto, Keynote (for motion/interaction prototypes)

## Code Highlights

No code samples are presented. The session's actionable guidance for Xcode is entirely non-code:

1. Open Xcode, create any app project, drag assets into the Asset Catalog to see icon scaling.
2. Open a Storyboard and import static mockups to create interactive flows without writing Swift.
3. Use the Object Library to browse all built-in UIKit controls and see their official names and configurable properties.

## Takeaways

- Adopt the official UIKit/HIG names for every UI element on your team — eliminating silent translation overhead is one of the cheapest improvements a team can make.
- Establish a single approved location for assets and deliverables before the project starts, not after the first miscommunication.
- Use standard SDK patterns as the starting point for any screen; save customization effort for features with proven user value, and build accessibility in from day one.
- Replace static spec hand-offs with short video recordings or interactive prototypes that show motion, timing curves, and intent — this alone prevents most designer-engineer review-cycle mismatches.

---
_Source: WWDC17 Session 809 page (abstract, transcript, and resource links)._
