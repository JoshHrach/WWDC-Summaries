# Principles of inclusive app design
**WWDC25 · Session 316** · [Watch](https://developer.apple.com/videos/play/wwdc2025/316/)

_Platforms:_ All Apple platforms

## Overview
This session reframes accessibility from a compliance checklist into a design methodology. The core argument is that disability is not a fixed property of a person but a mismatch between what a person can do and what the environment expects — an "inclusion gap." Every inclusion gap is an opportunity to innovate. Technologies like microphones, glasses, and curb cuts all emerged from closing inclusion gaps and ended up benefiting everyone.

Two Apple accessibility designers walk through four practical principles any developer can apply: support multiple senses, provide customization, adopt Accessibility APIs, and track inclusion debt. The session illustrates each principle with first-party examples (Accessibility Reader) and third-party apps (Crouton recipe app, Carrot Weather, Blackbox puzzle game).

## Key Topics

### The Inclusion Gap
Disability is born from the gap between individual capability and societal expectation. Society builds digital and physical environments assuming a narrow range of human ability. When that assumption fails — for a person with a permanent disability, a temporary impairment, or a situational constraint — the gap creates a disability in that context. Closing the gap is design work, not a medical or legal problem.

### Support Multiple Senses
Apps should never assume a single sensory channel is available. Every information-bearing element should have parallel representations across sight, hearing, touch, and speech:
- Captions for audio content (helps deaf users and anyone in a quiet space)
- Audible playback for visual text content (helps blind users and anyone driving)
- Haptic feedback as a complement to visual state changes

The new **Accessibility Reader** system feature exemplifies this: content can be read visually, listened to, or experienced as highlighted read-along — each mode serves different situations and disabilities.

### Provide Customization
Even multi-sensory design cannot anticipate every individual need. Apps should expose controls that let users adapt the experience: text size and font choice, color and contrast, information density (dense vs. minimal UI), and reading style. Carrot Weather demonstrates density customization spanning from data-rich to single-metric layouts, accessible from any screen.

### Adopt Accessibility APIs
Assistive technologies — VoiceOver, Switch Control, Voice Control, Larger Text — work because apps implement Accessibility APIs. The investment is multiplied: the same API that enables VoiceOver also powers Switch Control and Voice Control. Key APIs:
- `.accessibilityLabel()` — spoken description
- `.accessibilityAction()` — actions surfaced in VoiceOver's rotor
- `accessibilityElement(children:)` — grouping strategy

Blackbox (puzzle game) demonstrates accessible game design: every puzzle has VoiceOver audio labels with just enough information to solve it, combined with rich haptic feedback, making the entire game playable without vision.

### Track Inclusion Debt
Inclusion is iterative, not a launch checklist. "Inclusion debt" is the known gap between current accessibility and full inclusion. Acknowledging it explicitly — like technical debt — enables teams to plan, prioritize, and collaborate with disability community members. Accessibility Reader itself still has room to grow in formatting options; the session names this openly as tracked work.

### "Nothing About Us Without Us"
The session emphasizes involving people with disabilities as active participants in design and testing, not as subjects of design decisions made without them. The disability community phrase "Nothing about us without us" is cited as the guiding principle.

## APIs & Frameworks

- **Accessibility APIs (SwiftUI/UIKit):**
  - `.accessibilityLabel(_:)` — spoken label
  - `.accessibilityAction(_:)` — VoiceOver action menu entries
  - `.accessibilityElement(children:)` — grouping strategy
  - `Larger Text` support — Dynamic Type scaling
- **VoiceOver** — screen reader (adopt via Accessibility API)
- **Switch Control** — motor accessibility (same API as VoiceOver)
- **Voice Control** — voice navigation (same API as VoiceOver)
- **Accessibility Reader** **[NEW system feature]** — multi-modal reading (visual, audio, read-along)
- **Human Interface Guidelines: Accessibility** — design reference
- **Human Interface Guidelines: Inclusion** — inclusion design principles reference
- _(No code-level APIs introduced specifically in this session)_

## Code Highlights

_No new APIs are introduced in this session. The guidance centers on design principles and existing Accessibility API adoption._

```swift
// Core pattern: meaningful grouping and labeling for VoiceOver
HStack {
    Image(systemName: "heart.fill")
    Text("42 likes")
}
.accessibilityElement(children: .combine)
.accessibilityLabel("42 likes")

// Expose a custom action in VoiceOver
Button("Like") { like() }
    .accessibilityAction(named: "Double-tap to like") { like() }
```

```swift
// Larger Text: use Dynamic Type throughout
Text("Recipe title")
    .font(.headline)  // scales with user's preferred text size automatically
```

## Takeaways

- The inclusion gap is a design opportunity, not a compliance obligation — innovations born from accessibility constraints routinely benefit all users.
- Supporting multiple senses (visual, audio, haptic) is the single highest-leverage investment because situational constraints (noise, hands busy, sunlight glare) affect everyone.
- The VoiceOver, Switch Control, and Voice Control APIs share the same underlying implementation — one adoption serves three assistive technologies.
- Name your inclusion debt explicitly in your team's backlog; it converts invisible accessibility work into plannable, priority-ranked tasks.

---
_Source: WWDC25 Session 316 page (abstract, chapter summaries, code samples, and resource links)._
