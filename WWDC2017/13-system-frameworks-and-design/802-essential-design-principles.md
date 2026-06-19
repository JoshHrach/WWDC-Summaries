# Essential Design Principles
**WWDC17 · Session 802** · [Watch](https://developer.apple.com/videos/play/wwdc2017/802/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, tvOS 11, watchOS 4

## Overview
Presented by Mike Stern of Apple's Design Evangelism team, this session makes the case that great design emerges from a deeper understanding of human needs rather than from technique or process alone. Stern introduces eight interconnected design principles — wayfinding, feedback, visibility, consistency, mental models, proximity and grouping, mapping, affordance, progressive disclosure, and symmetry — and illustrates each using a narrative thread of a group trip to Hawaii (airport, car ride, hotel room, restaurant, snorkeling). By anchoring abstract principles in physical, everyday environments, the session makes each principle immediately intuitive and directly transferable to app UI design.

The central thesis is that apps must serve human emotional and practical needs: the need for safety and predictability, for knowledge and understanding, for achievement, and for beauty and joy. Every principle described is a mechanism for satisfying one or more of those needs.

## Key Topics

- **Wayfinding** — every screen in an app is a wayfinding system, just like airport signage. Each screen must answer: where am I? Where can I go? What will I find there? What is nearby? How do I exit? The iOS navigation bar title, tab bar selection, and back button are the primary wayfinding tools. Evaluate every screen against these five questions.
- **Feedback** — feedback must be clear, immediate, and understandable. Four types: status (gear position, fuel level; Mail badges, Calendar acceptance indicators), completion (engine starting, iPhone lock sound, email deletion animation, Apple Pay transaction animation), warning (low fuel), and error (form validation, Things 3 date autocorrection from "June 31" to "July 1"). Feedback lets users know what just happened, what is happening, and what to expect.
- **Visibility** — usability improves when controls and information are clearly visible without requiring extra navigation. Mail badges and Clock app tabs are examples where hiding information in a hamburger menu would measurably reduce usability. Visibility must be balanced against clutter.
- **Consistency** — represent similar features in similar ways. Users bring expectations from every other app on the platform; honor those expectations. The iOS share arrow ("sharrow") is the correct share glyph on iOS regardless of cross-platform conventions. Distinguish external consistency (match platform conventions) from internal consistency (glyphs in a set share a visual style; type uses a limited palette of faces, sizes, and colors).
- **Mental models** — users have an internalized model (system model + interaction model) of how every interface works. When an app matches that model, it feels intuitive; when it breaks the model, it feels unintuitive. The "Mortimer faucet" story illustrates how a logically superior design can fail because it violates users' existing mental model. Changes to long-lived products must clear a very high bar: prove objectively that the change produces clear wins.
- **Proximity and grouping** — controls near the objects they affect are perceived as related. Group related controls together; separate unrelated ones. Keynote's object tools are positioned above the canvas they affect; Sketch clusters related path/transform operations into distinct groups. Grouping is a fundamental structural principle.
- **Mapping** — design controls to resemble the objects they affect. A shade toggle with up/down arrows maps to the shade's motion. Light switches for ceiling lights should be arranged to mirror the ceiling layout. In interfaces: horizontal sliders for horizontal properties, rotary dials for rotation. Labels appear on controls when mapping is unclear — treat that as a design signal.
- **Affordance** — the perceived action possibilities of an object. Controls must make possible interactions obvious: sliders afford dragging, dials afford rotating, buttons afford tapping. Increasing abstraction is acceptable as long as the affordance cue remains present (the rounded rect still reads as a button; the filled circle on a line still reads as a slider thumb). Animation can also signal affordance (Weather app content that nudges up on tap to suggest scrollability).
- **Progressive disclosure** — gradually move users from simple to complex; hide less-used options behind a second level. The 80/20 rule: expose the 20% of functions that 80% of users need, and reveal the other 80% on demand (macOS Print dialog as canonical example). Risk: progressive disclosure can bury important options; validate discoverability empirically.
- **Symmetry** — bilateral, radial/rotational, and translational symmetry are ubiquitous in nature and perceived as stable, balanced, and aesthetically pleasing. Attractive app interfaces combine reflectional symmetry (centered key elements, counter-balanced flanking elements) and translational symmetry (evenly spaced repeating elements such as city rows in Clock or location cards in Weather).

## APIs & Frameworks

This is a design principles session with no new API introductions. Platform controls and components referenced throughout:

- `UINavigationBar` / `UINavigationItem` — wayfinding (title = "where am I")
- `UITabBar` / `UITabBarItem` — wayfinding (tabs = "where can I go")
- `UIBarButtonItem` (Back) — wayfinding (exit path)
- `UIBadge` (app icon badge, Mail badge) — visibility/status feedback
- `UIAlertController` — error feedback
- `UNNotificationSound` — sound as completion/status feedback
- `UISlider`, `UIRotationGestureRecognizer` — mapping/affordance examples
- `UIApplicationShortcutItem` — Home screen quick actions (proximity to OS context)
- Human Interface Guidelines (`developer.apple.com/design/`) — consistency reference

## Code Highlights

No code samples presented. The session is a design lecture with visual and narrative examples.

## Takeaways

- Walk through every screen in your app and ask the five wayfinding questions; if a screen cannot answer them, redesign it before adding any new features.
- Treat feedback as a conversation: after observing a first-time user, note every point where you verbally explain what the app is doing — then encode that explanation into the UI itself.
- Honor platform conventions for icons and navigation patterns unless you have empirical evidence that breaking them yields measurable improvement; the cost of inconsistency is always borne by the user.
- Before shipping a significant UX change to an existing app, test against the existing mental model; a logically superior design that violates learned expectations will be perceived as worse.

---
_Source: WWDC17 Session 802 page (abstract, transcript, and resource links)._
