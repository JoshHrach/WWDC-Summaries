# Design Considerations for Vision and Motion
**WWDC23 · Session 10078** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10078/)

_Platforms:_ visionOS 1

## Overview
This session, presented by Apple Vision Science team members Manda and Herman, explains the physiological and perceptual constraints that shape comfortable immersive experiences on visionOS. It bridges human biology — how the visual and vestibular systems work — with concrete design decisions developers must make when building apps and games for Apple Vision Pro.

The session covers two major comfort domains: vision comfort (how the eyes perceive and converge on 3D content) and motion comfort (how moving virtual content can trigger discomfort when visual and vestibular signals conflict). Both domains are grounded in neuroscience and translated into actionable guidelines.

Understanding these constraints is critical for visionOS developers because immersive apps engage the full field of view. Poor choices around depth cues, contrast, or motion can cause double vision, eye fatigue, dizziness, or nausea, directly harming user experience and app ratings.

## Key Topics

### Visual Depth Cues
The brain uses depth cues — familiar size, relative size, blur, shadow, occlusion, texture density, motion parallax — to drive eye convergence. Sparse scenes missing these cues cause the eyes to struggle to converge correctly, leading to double vision or fatigue. Conflicting cues (e.g., relative size contradicts occlusion) are equally problematic. Repeating patterns can cause each eye to lock onto different elements, creating rivalry and discomfort.

### Content Parameters: Size, Contrast, and Brightness
Content intended for extended reading should be placed beyond arm's length. Users should be able to adjust content depth. Reserve close-range placement for momentary interactions or direct touch. Use higher contrast for readable text and lower contrast / blur to redirect attention. Transitions from dark to bright scenes should be gradual to allow brightness adaptation.

### Eye Effort
The most comfortable gaze directions are downward, left, and right. Upward and diagonal eye rotations are the most fatiguing. Place content requiring sustained attention slightly below the line of sight. Design natural break points to let users rest their eyes.

### Motion of Virtual Objects
Large objects occupying much of the field of view that move can be misinterpreted by the brain as self-motion, causing dizziness. Mitigation: make moving large objects semi-transparent so passthrough content remains visible.

### Head-Locked Content
Avoid content anchored to the user's head. If unavoidable, use small windows near the center of the line of sight and far from the viewer. Prefer world-locked views or lazy-follow animations (content drifts slowly to a destination).

### Camera Motion in Windows
For windowed video content, keep the horizon aligned with the real horizon. Keep the focus of expansion (the vanishing point toward which all pixels appear to flow) slow, predictable, and within the field of view. Avoid fast rotations or pure rotational camera motion — use fade-out transitions instead. Avoid large objects at close range; prefer small objects at distance with low-contrast textures.

### Oscillating Motion
Avoid sustained oscillations, especially near 0.2 Hz (one cycle every five seconds). If oscillation is unavoidable, keep amplitude low and content semi-transparent. Always provide an oscillation-free alternative via the Reduce Motion accessibility setting.

## APIs & Frameworks

- **visionOS** — the target platform for all guidelines **[NEW]**
- **Reduce Motion** accessibility setting — should be respected with an oscillation-free animation alternative **[NEW]**
- Stereo rendering / manual disparity rendering — must provide correct per-eye disparity to avoid severe visual conflict (relevant when manually authoring stereo video content)
- Passthrough content compositing — semi-transparency of moving objects to reveal real-world passthrough

_Note: This is a design-principles session; it does not introduce specific Swift APIs but directly informs how developers configure RealityKit scenes, SwiftUI window placement, and animation parameters on visionOS._

## Code Highlights
No code samples were shown. The session is a design and human-factors talk intended to inform architectural and visual decisions rather than specific API usage.

## Takeaways
- Always include multiple redundant depth cues (size, blur, shadow, occlusion) to help the brain converge eyes correctly in 3D scenes.
- Keep large moving objects semi-transparent so passthrough remains visible and the brain doesn't interpret object motion as self-motion.
- Avoid head-locked UI; use world-locked or lazy-follow placement instead.
- Respect the Reduce Motion accessibility setting with an oscillation-free animation path.

---
_Source: WWDC23 Session 10078 page (abstract, chapter summaries, code samples, and resource links)._
