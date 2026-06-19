# Principles of spatial design
**WWDC23 · Session 10072** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10072/)

_Platforms:_ visionOS 1

## Overview
This foundational session from Apple's Design team lays out the core principles for designing great apps on visionOS. Rather than prescribing specific APIs, it establishes five guiding principles — Familiar, Human-centered, Dimensional, Immersive, and Authentic — that should inform every design decision for spatial computing. The session is an essential starting point before diving into more technical visionOS sessions.

The session acknowledges that visionOS introduces a completely new canvas: depth, scale, real-world space, eyes and hands as inputs, and Spatial Audio. Yet it argues that the best spatial apps still feel intuitive because they build on familiar patterns (windows, tabs, navigation) while leveraging these new dimensions where they genuinely enhance the experience. The goal is to identify a single "key moment" in your app that can only be experienced spatially, and use that as the foundation for your spatial design.

## Key Topics

### Familiar: Windows, Points, and Known Patterns
visionOS apps live in windows rendered with a new glass material that provides contrast with the real world and adapts to lighting. Windows can be moved, resized, and repositioned by users via the window bar. The points coordinate system — already familiar from iOS/macOS — ensures interfaces scale correctly at any distance. Apps should use familiar patterns (sidebars, tab bars, toolbars, search fields) and prefer a single window; multiple windows add management burden.

### Human-Centered: Field of View and Ergonomics
People see in a wide landscape field of view, so interfaces should use landscape layouts and place the most important content at center. Content should be placed at a comfortable arm's-length-plus distance, relative to the direction the person is facing (accounting for varying heights and positions). Never anchor content to someone's view — it feels stuck and disorienting; anchor to space instead. Experiences should be usable without requiring the person to move. The system's Digital Crown recenter gesture handles repositioning; apps don't need their own reset mechanism.

### Dimensional: Depth, Scale, Light, and Shadow
Depth is a new design variable that can create hierarchy and focus. Nearby objects invite close inspection; far objects encourage distant interaction. Depth must be reinforced with visual cues: light-emitting objects cast colored light onto surroundings; most objects cast shadows to appear grounded. Text should remain flat (not 3D) when used as an interface element. Prefer subtle depth; modal dialogs use a gentle pushback of the background window. Scale dramatically changes the emotional quality of an object — from personal and lightweight (small) to impressive and larger-than-life (large).

### Immersive: The Immersion Spectrum
Apps exist on a spectrum from Shared Space (windowed, alongside other apps) through partially immersive to Full Space (other apps hidden). Start in the Shared Space and let users opt into deeper immersion. Dimming is a simple way to create focus without leaving the space. Full Space experiences can take users to an entirely new place or exist within their own room. Transitions between immersion levels should be smooth and predictable. Immersive experiences should guide focus (not overwhelm), blend thoughtfully with reality using soft edges, use subtle animation to create liveliness, and leverage Spatial Audio to anchor sound to objects or create soundscapes.

### Authentic: Key Moments and Platform-Native Experiences
Great spatial apps are not quick in-and-out experiences; they are engaging, worthwhile, and distinct enough that people welcome them into their space. Identify one key moment — something only possible spatially — and build around it. Examples: Photos showing a memory at lifelike scale, or a panorama transporting the user back to a place. Experiences should be rich, use high-fidelity visuals and audio, and create memorable feelings (reliving a moment, seeing from another's perspective).

## APIs & Frameworks
This is a design principles session; it does not introduce specific code APIs. Referenced platform concepts include:

- **Windows** — the primary UI container in visionOS; glass material, movable, resizable
- **Window Bar** — system-provided control for repositioning windows
- **Shared Space** — the default multi-app environment on visionOS
- **Full Space** — exclusive single-app environment (other apps hidden)
- **Points system** — coordinate unit for interface layout, scales with viewing distance
- **Tab bars / Toolbars** — layered above window for persistent access
- **Sidebar** — familiar navigation pattern adapted for spatial UI
- **Glass material** — visionOS window background that contrasts with and adapts to surroundings
- **Spatial Audio** — anchoring audio to objects in 3D space or creating surrounding soundscapes
- **Digital Crown** — hardware button; press-and-hold recenters app content
- **Dimming** — system capability to darken passthrough and focus attention on content
- **Passthrough** — live video of real-world surroundings rendered by visionOS
- **Environments** — immersive full-space scenes that extend beyond physical room dimensions
- **Shadow and lighting** — physical simulation cues that ground virtual objects in real space

## Code Highlights
No code samples — this is a design principles session. See companion sessions for implementation:
- "Design for Spatial User Interfaces" (10076) — window layout and UI specifics
- "Design for Spatial Input" (10073) — eyes, hands, and input design
- "Design Spatial SharePlay Experiences" (10075) — shared immersive experiences
- "Explore Immersive Sound Design" (10271) — Spatial Audio design
- "Get Started with Building Apps for Spatial Computing" (10260) — developer starting point

## Takeaways

- Start every spatial design by identifying a single "key moment" unique to the spatial medium — depth, scale, immersion, or real-world blending — rather than trying to make everything spatial at once.
- Default to a windowed Shared Space experience; let users opt into deeper immersion, and always provide a clear, labeled way in and out of Full Space.
- Place content ergonomically: landscape layout, center-weighted, at arm's-length-plus distance, relative to the user's facing direction — never anchored to their view.
- Reinforce depth with light and shadow cues, prefer subtle depth over dramatic effects, keep text flat, and use scale deliberately to change the emotional register of content.

---
_Source: WWDC23 Session 10072 page (abstract, chapter summaries, and resource links)._
