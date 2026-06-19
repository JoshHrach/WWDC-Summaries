# Design Great visionOS Apps
**WWDC24 · Session 10086** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10086/)

_Platforms:_ visionOS 2

## Overview
This design session distills the lessons Apple's design team learned from the first generation of visionOS apps and codifies them into actionable guidance for visionOS 2. The talk is framed around three foundational principles — connection to physical space, comfort for extended use, and deliberate depth — and walks through how those principles translate into concrete layout, navigation, and interaction decisions.

The session is not API-heavy; it is primarily a design reference for developers and designers making architectural decisions about their visionOS apps. It covers window placement, ornament usage, volume sizing, and the transition between windowed and immersive modes.

## Key Topics
- **Spatial layout** — apps should occupy a predictable region of the user's space; avoid windows that follow the user's head or move unexpectedly; prefer opening new content in new windows rather than replacing existing ones.
- **Ornaments** — toolbars and supplementary controls placed alongside windows using `ornament(visibility:attachmentAnchor:contentAlignment:content:)`; keep ornaments contextual and avoid stacking multiple ornaments.
- **Depth and z-ordering** — use depth sparingly; content that protrudes too far from the window plane (more than ~0.5 m) causes discomfort; prefer `ZStack` with subtle offsets and `PhysicalMetric` for calibrated metric values.
- **Volumes vs. windows** — volumes (bounded 3D scenes) are appropriate for self-contained 3D objects; windows remain appropriate for document-style content; the session provides heuristics for choosing between them.
- **Immersive space transitions** — transitions into and out of full immersion should be intentional and user-initiated; use `ImmersionStyle.progressive` for gradual passthrough dimming rather than abrupt full occlusion.
- **Typography and legibility** — minimum legible text size at arm's length (~60 cm) is roughly 24 pt Dynamic Type; avoid text in volumes without a backing material.

## APIs & Frameworks

**SwiftUI / visionOS**
- `WindowGroup` — primary window container; unchanged
- `ImmersiveSpace` — full immersion scene container; unchanged entry point
- `ImmersionStyle` — `.mixed`, `.progressive`, `.full`; **[NEW]** `.progressive` available as a system-level slider in visionOS 2
- `ornament(visibility:attachmentAnchor:contentAlignment:content:)` — place supplementary controls alongside a window
- `OrnamentAttachmentAnchor` — `.scene(alignment:)` anchors an ornament to a side of the scene bounds
- `PhysicalMetric` — `PhysicalMetricsConverter`; translate logical point values to physical meters for volume sizing
- `Model3D` — display a 3D model asset in a window or volume; unchanged
- `RealityView` — host RealityKit content; unchanged
- `VolumetricWindowStyle` — apply to a `WindowGroup` to create a bounded 3D volume
- `glassBackgroundEffect()` — standard visionOS material for window and control backgrounds; use on text-bearing content in 3D to ensure legibility
- `hoverEffect(_:)` — standard hover highlight; always apply to interactive elements
- `pushWindowAction` / `dismissWindowAction` — open/dismiss named window scenes
- `preferredSurroundingsEffect(_:)` — blend between real world and a custom backdrop in progressive immersion

## Code Highlights
Open a supplementary detail window without replacing the main window:

```swift
@Environment(\.pushWindow) private var pushWindow

Button("Show Details") {
    pushWindow(id: "detail")
}
```

Attach a contextual toolbar ornament to the bottom of a window:

```swift
WindowGroup(id: "main") {
    ContentView()
        .ornament(visibility: .visible, attachmentAnchor: .scene(.bottom)) {
            HStack { /* toolbar buttons */ }
                .glassBackgroundEffect()
        }
}
```

Use `ImmersionStyle.progressive` for user-controlled passthrough blending:

```swift
ImmersiveSpace(id: "garden") {
    GardenView()
}
.immersionStyle(selection: $style, in: .progressive)
```

## Takeaways
- Prefer opening content in new windows over navigating in-place when the content is semantically a different context; visionOS's multi-window model makes this feel natural.
- Keep volumes small enough to sit on a surface or desk metaphor (roughly 0.5 m × 0.5 m × 0.5 m); large volumes crowd the space and make it hard to navigate around them.
- Use `.progressive` immersion style for environments that enhance but do not replace the real world — it gives users direct control over how isolated they feel.
- Always apply `glassBackgroundEffect()` to text content that floats in 3D space without a window; text against raw passthrough video is unreadable in varied lighting.

---
_Source: WWDC24 Session 10086 page (abstract, chapter summaries, code samples, and resource links)._
