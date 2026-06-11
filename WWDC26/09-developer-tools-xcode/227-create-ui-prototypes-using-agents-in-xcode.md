# Create UI Prototypes Using Agents in Xcode

**WWDC26 · Session 227** · [Watch](https://developer.apple.com/videos/play/wwdc2026/227/)

_Platforms:_ Xcode 27 · iOS · macOS · iPadOS

## Overview

This session focuses on using Xcode coding agents and SwiftUI Previews as a rapid design prototyping environment. The core thesis is that agents should be treated as collaborators — generators of possibilities — while the developer acts as the creative director who evaluates, remixes, and refines. Three distinct prototyping phases are covered: exploring UI possibilities, making a prototype feel lived-in, and tuning key interaction moments.

The Exploring phase demonstrates how to prompt an agent to produce multiple distinct UI variations simultaneously, then selectively combine the most promising elements from each into a refined iteration. Rather than accepting the first result, developers learn to think in "remix cycles" — generating, critiquing, and recombining — to reach creative solutions that wouldn't emerge from a single prompt.

The final phase introduces a powerful debugging technique: building custom in-app tuning panels that expose animation parameters (duration, spring stiffness, damping) as real-time controls. This lets developers and designers iterate on motion directly in a running preview without code edit/recompile cycles, dramatically accelerating the feel refinement loop.

## Key Topics

### Exploring UI Possibilities
- Craft prompts that request multiple distinct UI variations in a single agent turn.
- Evaluate each variation critically and identify the strongest elements.
- Remix: combine elements from different variations into new iterations.
- Use Xcode Previews as an instant visual feedback loop throughout.

### Making Your App Feel Lived-In
- Use agents to populate prototype views with realistic sample data.
- Cover design edge cases systematically:
  - Empty states
  - Long or truncated text
  - Unbounded/large list sizes
  - Mixed content types
- Ensuring edge cases are covered before design review prevents late-stage surprises.

### Tuning Key Moments
- Identify the dynamic transitions and interactions that define the app's character.
- Build **custom in-app tuning panels** — agent-generated SwiftUI overlays with sliders — to adjust animation parameters (duration, spring stiffness, damping) in real time without recompiling.
- Iterate on motion feel directly in a running Preview or on-device build.

## APIs & Frameworks

**Xcode 27 Coding Agent**
- Multi-variation prompt technique — request N distinct UI approaches in one prompt
- Remix workflow — combining elements across agent-generated variations
- Edge-case data generation (empty states, long strings, large lists)
- **[NEW]** Custom tuning panel generation — agent builds an in-app parameter control overlay

**SwiftUI**
- `Preview` macro / `#Preview` — instant visual feedback while iterating
- `VStack`, `HStack`, `LazyVStack` — layout primitives used in prototypes
- `Slider` — used in tuning panels for real-time parameter adjustment
- Animation modifiers: `.animation(_:value:)`, spring animations
- State management: `@State`, `@Binding` for tuning panel parameters

**Swift Charts**
- Referenced in related session (259) for data-driven UI iteration

## Code Highlights

Custom tuning panel pattern (agent-generated overlay for animation parameter adjustment):

```swift
// Agent generates a parameter panel like this for real-time tuning
struct AnimationTuningPanel: View {
    @Binding var duration: Double
    @Binding var springStiffness: Double
    @Binding var damping: Double

    var body: some View {
        VStack {
            Slider(value: $duration, in: 0.1...1.5) { Text("Duration") }
            Slider(value: $springStiffness, in: 10...300) { Text("Stiffness") }
            Slider(value: $damping, in: 0.1...1.0) { Text("Damping") }
        }
        .padding()
        .background(.ultraThinMaterial)
    }
}
```

Multi-variation prompt approach:
- Prompt: "Generate 3 distinct UI designs for [screen]. Make each visually different — vary layout, typography weight, and information hierarchy. Show all three."
- Then: "Combine the card style from design 1 with the header treatment from design 3."

## Takeaways

- Requesting multiple variations at once and remixing them outperforms iterating on a single design — the creative diversity of variations produces better outcomes.
- Populating prototypes with realistic edge-case data (empty, overflowing, mixed) before design review prevents costly late changes.
- Custom agent-generated tuning panels turn animation refinement from a code-edit cycle into a real-time slider-based workflow, enabling faster and more intuitive feel iteration.
- The developer's judgment — not the agent — is the deciding creative force; agents surface options, humans choose direction.

---
_Source: WWDC26 Session 227 page (abstract, chapter summaries, and resource links)._
