# Design Hover Interactions for visionOS
**WWDC25 · Session 303** · [Watch](https://developer.apple.com/videos/play/wwdc2025/303/)

_Platforms:_ visionOS 26

## Overview
This design-focused session explores how to create compelling, privacy-preserving hover interactions for visionOS apps. Because the system never exposes gaze direction to apps directly, hover effects operate by defining two view states — standard and hovered — and letting the system animate between them when a person looks at or away from an element.

The session covers three major interaction categories: custom hover effects (animations tied to gaze), Look to Scroll (scrolling scroll views by gazing at their edges), and persistent controls (keeping auto-hiding UI visible while a person looks at it).

Developers and designers will come away with a solid framework for deciding when and how to apply each pattern, along with practical pitfalls to avoid.

## Key Topics

### Fundamentals Recap
- Interactive elements should be rounded (circles, pills, rounded rectangles) and spaced at least 60 points apart (≈ 2.5° angular size / 4.4 cm at 1 m).
- Standard components apply highlight effects automatically; custom components must add them explicitly — including on selectable 3D objects.

### Custom Hover Effects
Three animation archetypes:
- **Instant** — starts immediately on look; good for small contextual additions (timestamp on video scrubber, arrow icon on a button).
- **Delayed** — waits before completing; good for tooltips and secondary labels that shouldn't fire on casual glance.
- **Ramp** — starts slowly then pops open; good for expandable cards or environment icons that hint at interaction without fully opening.

Best-practice rules:
- Provide **anchoring elements** — keep some portion of the view stable so reading is not interrupted.
- Effects must start from a **visible element** (no hidden triggers).
- Apply effects **sparingly** — toolbar buttons and table cells usually should just use the standard highlight.
- Keep effects **small and subtle**; avoid applying them to large imagery.
- Avoid effects that wash out colors — show highlight, then fade so true colors show through.
- **Test on device** — the simulator cannot faithfully reproduce gaze-reactive effects.

Custom effects are privacy-preserving by design: they run out of process, so apps cannot infer where users are looking.

### Look to Scroll
- Enabled by opting in specific scroll views; not on by default.
- Best for reading/browsing content (articles, media libraries) — not settings lists or UI-heavy views.
- Scroll views should ideally fill the window edge-to-edge for clear trigger zones.
- Avoid for scroll views that drive parallax or custom animation tied to scroll position.

### Persistent Controls
- Standard video player controls **[NEW in visionOS 26]** now stay visible while the user is looking at them.
- Custom video controls and other auto-hiding UI (FaceTime, Mindfulness) can adopt the same persistence behavior via API.

## APIs & Frameworks

### visionOS / SwiftUI & RealityKit
- **Custom hover effects** — created in SwiftUI or RealityKit (refer to 2024 sessions for implementation).
- **Look to Scroll** — opt-in API on scroll views; documentation at developer.apple.com. **[NEW]**
- **Persistence behavior for controls** — API to keep auto-hiding controls visible while gazed at. **[NEW]**

### CompositorServices
- `rendering_hover_effects_in_metal_immersive_apps` — documentation for rendering hover effects in Metal immersive apps.

## Code Highlights
No code samples were shown in this session (it is a design-focused talk). Implementation details are covered in the referenced 2024 engineering sessions on custom hover effects in SwiftUI and RealityKit.

## Takeaways
- Default to the standard highlight effect for toolbar buttons and table cells; only build custom hover effects where they add clear value.
- Use the ramp animation pattern (slow ease-in → quick spring) for expandable content to give intent-signaling feedback without premature opening.
- Opt into Look to Scroll for primary reading/browsing scroll views that fill the window, not for settings or control-dense views.
- Adopt persistent controls for any auto-hiding UI — not just video players — so it doesn't vanish while a user is looking at it.

---
_Source: WWDC25 Session 303 page (abstract, chapter summaries, and full transcript)._
