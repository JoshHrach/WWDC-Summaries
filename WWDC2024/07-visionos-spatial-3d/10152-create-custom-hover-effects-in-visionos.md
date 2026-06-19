# Create Custom Hover Effects in visionOS
**WWDC24 · Session 10152** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10152/)

_Platforms:_ visionOS 2

## Overview
visionOS 2 introduces a fully customizable hover effect system that gives developers control over how views respond visually when a user looks at them. Prior to this release, apps were limited to the system-provided highlight; now developers can build their own per-view hover responses — highlighting a glyph, scaling content, revealing additional controls, or running a SwiftUI animation — all without sacrificing the energy-efficient, privacy-preserving eye-tracking design visionOS requires.

The session explains why hover effects run in a separate rendering pass (to protect gaze privacy), how that architectural constraint shapes the API, and then demonstrates building a range of custom effects using the new `hoverEffect` modifier overloads and the `HoverEffectGroup` type.

## Key Topics
- **How hover effects work in visionOS** — hover effects are rendered in a system-compositor pass using only the view's snapshot and a minimal description of the effect, so the app process never sees raw eye-tracking data.
- **`hoverEffect(_:)` with a custom effect** — the existing modifier gains an overload that accepts a custom `HoverEffect` built from a `HoverEffectGroup` closure.
- **`HoverEffectGroup`** — a new builder type whose closure receives a `HoverEffectContent` proxy; use `scaleEffect`, `opacity`, `clipShape`, and transform modifiers on the proxy to describe the effect.
- **`isActive` flag** — the closure is called with both the hovered and non-hovered states; use `isActive` (a `Bool`) to branch between them.
- **Driving SwiftUI state from hover** — `onHover(perform:)` remains available; use it to update `@State` and animate SwiftUI views that live outside the compositor pass.
- **`hoverEffectGroup` modifier** — a container-level modifier that coordinates hover detection across a group of sibling views, ensuring only the first matching view in the group lights up.

## APIs & Frameworks

**SwiftUI (visionOS)**
- `hoverEffect(_:)` — existing modifier; adds system highlight
- **[NEW]** `hoverEffect(_:in:isEnabled:)` — new overload accepting a custom `AnyHoverEffect` or a `HoverEffect` closure
- **[NEW]** `HoverEffect` — protocol-like struct; created via `HoverEffect { proxy, isActive, _ in … }` trailing closure
- **[NEW]** `HoverEffectContent` — proxy type available inside the `HoverEffect` closure; supports:
  - `scaleEffect(_:anchor:)` on `HoverEffectContent`
  - `opacity(_:)` on `HoverEffectContent`
  - `clipShape(_:style:)` on `HoverEffectContent`
  - `brightness(_:)` / `saturation(_:)` / `hueRotation(_:)` on `HoverEffectContent`
  - `transformEffect(_:)` / `rotationEffect(_:anchor:)` on `HoverEffectContent`
- **[NEW]** `hoverEffectGroup(id:in:isEnabled:)` — container-level modifier; coordinates hover across child views; children opt in with `.hoverEffect(in:)` specifying the same namespace
- `onHover(perform:)` — existing modifier; still available for driving SwiftUI animations alongside compositor effects
- `@Namespace` — used to associate `hoverEffectGroup` container with `.hoverEffect(in:)` children

## Code Highlights
Custom scale-and-brighten hover effect on a button:

```swift
Button { … } label: {
    Image(systemName: "star.fill")
        .font(.largeTitle)
}
.hoverEffect { effect, isActive, _ in
    effect
        .scaleEffect(isActive ? 1.2 : 1.0)
        .brightness(isActive ? 0.3 : 0)
}
```

Coordinated group hover across sibling icons:

```swift
@Namespace var hoverNamespace

HStack {
    ForEach(items) { item in
        ItemView(item: item)
            .hoverEffect(in: hoverNamespace)
    }
}
.hoverEffectGroup(in: hoverNamespace)
```

## Takeaways
- The `HoverEffect` closure runs inside the system compositor — do not try to call back into app state from inside it; use `onHover` for that pattern.
- Prefer `hoverEffectGroup` for toolbars and tab bars where only one item should highlight at a time; it deduplicates hit testing automatically.
- `isActive` gives you two snapshots (hovered and idle); visionOS interpolates between them, so you do not need to write the animation yourself.
- Keep custom effects subtle — the system highlight is calibrated for comfortable eye-tracking ergonomics; effects that are too bold can feel fatiguing.

---
_Source: WWDC24 Session 10152 page (abstract, chapter summaries, code samples, and resource links)._
