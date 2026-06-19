# Design for Spatial Input
**WWDC23 · Session 10073** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10073/)

_Platforms:_ visionOS 1

## Overview
This session covers how to design interactions for Apple Vision Pro's two primary spatial input modalities: eyes and hands. Presenters Israel Pastrana Vicente and Eugene Krivoruchko from the Apple Design team explain the principles behind eye-based targeting, hand gesture design, and direct touch — and how these combine to create comfortable, intuitive, and precise spatial experiences.

The guidance directly informs how developers should lay out UI elements, size controls, apply hover effects, and handle gesture recognition on visionOS. It also underscores that the responsibility for ergonomics and accessibility is higher in spatial computing than on traditional flat displays.

## Key Topics

### Input Modalities on visionOS
Available inputs: eyes + hand pinch gesture (primary spatial input), voice, virtual keyboard with fingertip typing, trackpad and keyboard (hardware), and game controllers. The session focuses on eyes and hands as the new and defining inputs for the platform.

### Eyes: Targeting and Comfort
- Eyes are the primary targeting mechanism — interactive elements highlight as you look at them.
- Keep all main content inside the field of view to minimize neck and body movement.
- Place primary content in the center of the field of view; use edges only for secondary, less frequently accessed content.
- Manage depth carefully: keep interactive content at a consistent Z depth; modal presentations push the background back while keeping the modal at the original depth.
- Subtle depth differences can communicate UI hierarchy (e.g., tab bar vs. segmented control).

### Eye Target Design
- Use round shapes (circles, pills, rounded rectangles) to guide eyes to the center of elements; avoid sharp edges.
- Minimum target area: **60 points** (combining element size and spacing).
- Use generous spacing between controls to improve accuracy.
- Standard system components are pre-sized for eye targeting; custom controls must match the 60-point minimum.
- Always use **dynamic scale** (not fixed scale) for custom UI so target areas remain consistent at all distances.
- Keep UI oriented to face the viewer; tilted/angled interfaces are difficult to use.

### Hover Effects and Privacy
- All interactive elements must show a hover effect when the user's gaze falls on them.
- Hover effects run out of the app's process — the app only learns which element was targeted after a gesture completes; raw gaze data is never exposed to apps.
- System elements use gaze dwell to reveal extra information: tooltips on buttons, label expansion on tab bars, Speak to Search on the system search field.
- Dwell Control (accessibility feature) allows selection with eyes alone, with no hand gesture required.

### Hands: Gesture Design
- Pinch = tap; pinch + drag = scroll; two-handed pinch + spread = zoom; two-handed rotate = rotate.
- Gesture feedback should continue the motion of the hand to feel physically connected.
- Custom gestures: must be easy to learn, not conflict with system gestures, reproducible without fatigue, have low false-positive rates, and be accessible-technology-compatible.
- Always provide a UI affordance fallback for custom gestures.
- Eye + hand combination: gaze position at gesture start determines interaction origin (e.g., zoom origin, Markup brush jump location).

### Direct Touch
- Reaching out to interact directly with virtual content is supported but causes arm fatigue — reserve for experiences where physical closeness matters (3D object manipulation, close inspection, muscle memory from real-world tasks).
- Compensate for lack of tactile feedback with hover proximity cues (brightness gradient approaching surface), responsive state changes, and spatial audio on contact.

## APIs & Frameworks

- **visionOS** input model — eyes + hands as primary spatial input **[NEW]**
- **Hover effect** — system-provided highlight that appears when gaze falls on an interactive element; runs out-of-process for privacy **[NEW]**
- **Dynamic scale** — system window scaling that preserves target area sizes at any distance **[NEW]**
- **World-locked windows** — windows that stay in place in the user's space rather than following the head **[NEW]**
- **Dwell Control** — accessibility feature enabling selection via sustained gaze **[NEW]**
- **Speak to Search** — voice input activated by gazing at the system search field microphone **[NEW]**
- **60-point minimum target area** — the platform HIG requirement for interactive controls, combining size and spacing
- `UIHoverGestureRecognizer` — hover gesture support (carried forward; relevant to eye targeting on visionOS)
- **Pinch gesture** — primary select action; equivalent of tap on touch screens **[NEW]**
- **Two-handed zoom / rotate gestures** — system standard multi-hand gestures **[NEW]**
- **Direct interaction** / fingertip touch — direct contact with virtual content within arm's reach **[NEW]**

## Code Highlights
No code samples were presented. This is a design-principles session; implementation details are covered in platform engineering sessions.

## Takeaways
- Apply hover effects to every interactive custom element — this is the primary signal to users that their gaze is driving the interaction.
- Enforce the 60-point minimum target area through a combination of element size and spacing, and always use dynamic (not fixed) scale for custom UI.
- Use eye direction as a signal of intent within gestures (e.g., zoom origin, pointer jump) to create magically precise interactions unique to visionOS.
- Limit direct touch to experiences where physical proximity is core; compensate for absent tactile feedback with proximity highlight gradients and spatial audio.

---
_Source: WWDC23 Session 10073 page (abstract, chapter summaries, code samples, and resource links)._
