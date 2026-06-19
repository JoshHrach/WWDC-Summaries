# Intentional Design
**WWDC18 · Session 802** · [Watch](https://developer.apple.com/videos/play/wwdc2018/802/)

_Platforms:_ iOS, macOS, watchOS (Design — no new APIs)

## Overview
This design keynote from Apple interaction designer Doug LeMoine argues that great apps are the result of a conscious, focused intent on the people they serve. Using five real apps as case studies — iTranslate Converse, Vanido, Streaks Workout, Carrot Weather, Tinycards, and Gorogoa — the session distills five elements of intentional design: radical simplification, deep understanding, extreme focus, personal connection, and direct communication.

The session contains no new framework APIs; its value is as a design philosophy framework that applies to any platform.

## Key Topics

### What "Simple and Natural" Actually Means

- "Simple" is shorthand for comfort and confidence: when users understand how something works they can move through it without friction or conscious thought.
- Simple is not achieved by removing all features; it is achieved by removing everything that does not serve the user's core need.
- "Natural" means the interaction arises from what users already know — from the world or from the platform.

### Element 1: Radical Simplification

- **iTranslate Converse**: designed to be used by two people together, one of whom has never seen the app. The entire UI is one large button. Context drove the decision — a user in a foreign train station is jetlagged, lost, and stressed; the last thing they need is menus.
- **Vanido** (singing trainer): strips away sheet music and notation entirely; shows only a color/pitch indicator and one interactive element placed high enough that users can hold the phone like a microphone and belt out a note without accidentally dismissing the lesson.
- Risk-reduction: radical simplification feels jarring only when the removed elements were actually needed. Deep understanding of the user context eliminates that risk.

### Element 2: Deep Understanding

- Surface needs mask deeper needs. Dig past what users say they want.
- **Streaks Workout**: users' stated need is exercise guidance; their real need is overcoming inertia (disinclination to act). The app addresses inertia by removing decisions: choose movements once, and the app auto-selects sequence and reps randomly every session. No set order to choose, no rep count to decide.
- Inspiration came from a "prison workout" using a deck of cards — movements assigned to suits, reps on card numbers. The developers prototyped a literal card-flip UI but moved past it because the literal metaphor imposed constraints (four suits limit movement count) and added cognitive work (remember which suit = which movement). They stayed true to the spirit — randomness and surprise — without the literal form.
- Design insight: eliminating decisions is eliminating inertia. Each choice a user must make is a potential exit point.

### Element 3: Extreme Focus

- **The Rollaboard suitcase story**: Robert Plath (a Northwest Airlines pilot) designed the tilted, inline-wheel suitcase with a telescoping handle in 1989. Target audience: ~100,000 travel professionals (0.1% of the travel market). Luggage makers had tried to add wheels for decades without success because they didn't break from the existing form (handle on top, wheels under the bottom). Plath's extreme focus on people who traveled daily freed him to reorient the entire object — turning it on its side.
- **Carrot Weather**: the developer writes forecasts channeling the personalities of specific people close to him (wife, sister, mother). An extremely specific audience ("people who appreciate dark, sarcastic humor") freed him to create something distinctive that resonated far more widely. Personality is adjustable (slider from less edge to more), an "olive branch" for users who expect neutral forecasts.
- Focusing on an edge case can surface the real needs of a much larger audience by eliminating the compromises that come from designing for the average.

### Element 4: Personal Connection

- Apps that feel personal create comfort and loyalty — like a local bar where everyone knows your name.
- Personal does not mean spicy copy; it means the app communicates a genuine human perspective that users can relate to.
- Personality works only when the underlying utility is solid. Carrot Weather's humor lands because the weather data and navigation are excellent.

### Element 5: Direct Communication

- Tab bars, labels, and navigation elements should be as specific as possible.
- **The "Home" tab anti-pattern**: using a house icon and "Home" label in a tab bar avoids the decision of what to actually call the content, but it removes predictability. Users cannot guess what the tab contains. Directness is limiting — and that constraint is a feature, not a bug. It forces clarity about what the app actually does.
- The WWDC app tab bar example: "Schedule," "Videos," "News," "Featured" — each label tells you exactly what you will find.
- **Tinycards**: the entire UX is built on the flashcard metaphor — cards pulse to invite interaction, flip to reveal the answer, can be flicked rapidly. The metaphor is implemented so faithfully that users forget they are interacting with a UI.
- **Gorogoa**: extends the card metaphor beyond the physical — cards can be torn, layered, and stitched together into scenes. Extending a metaphor works only when the extension adds value or delight; it fails when it is decorative (as with the generic "Home" tab).

### Where Bad Design Comes From

- Experience and pattern-recognition help designers work efficiently but also prevent them from noticing when familiar patterns don't fit new problems.
- The brain auto-applies patterns; intentional design requires consciously slowing down and challenging the obvious — asking "why does this need to be this way?"
- Home, hamburger menus, and other "familiar" elements are defaults that feel safe but often mask an unwillingness to make a harder, clearer decision.

## APIs & Frameworks

This session introduces no new APIs. It is a design philosophy session.

Design principles referenced:
- Radical simplification — remove UI until only the essential interaction remains
- Deep understanding — identify what users actually need, not just what they say they want
- Extreme focus — design for a very specific person; the specificity enables better, simpler solutions
- Personal connection — apps with genuine personality create loyalty and delight
- Direct communication — labels and navigation elements should be as specific as possible; metaphors should be implemented faithfully and extended only to add real value

## Code Highlights

No code in this session.

The five elements of intentional design as a checklist:
1. **Radical simplification** — remove every UI element that doesn't serve the user's primary goal in their primary context.
2. **Deep understanding** — identify the real barrier (e.g., inertia, fear, confusion), not just the surface request.
3. **Extreme focus** — design for one specific, well-understood person; resist the pressure to broaden scope prematurely.
4. **Personal connection** — let a genuine human perspective permeate the app's voice and visual character.
5. **Direct communication** — name things specifically; implement metaphors faithfully; extend them only to add real value or delight.

## Takeaways
- The "Home" tab is the session's concrete anti-pattern: it feels safe but is actually an unintentional design — a failure to make a direct, meaningful choice about what the tab contains.
- Focusing on an extreme edge case (a pilot who travels every day; a person who hates working out) frees designers from the compromises of "average user" thinking and often produces solutions that resonate far more broadly.
- Every decision made should be traceable back to a specific person and a specific problem. If you cannot articulate who benefits and how, the decision is likely habitual rather than intentional.
- Extending a metaphor (Gorogoa's cards that stitch together) works only when it adds delight or utility; decorative metaphor-extension (a "Home" icon that means "first tab") adds nothing and obstructs understanding.

---
_Source: WWDC18 Session 802 page (abstract, full transcript, and resource links)._
