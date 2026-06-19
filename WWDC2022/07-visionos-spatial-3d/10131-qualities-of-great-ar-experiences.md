# Qualities of great AR experiences
**WWDC22 · Session 10131** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10131/)

_Platforms:_ iOS 16, iPadOS 16

## Overview
Presented by Alli Dryer from the Apple Design team, this session provides a practical design framework for building augmented reality features and apps on iPhone and iPad. It covers two major themes: how to decide when AR adds genuine value to an experience, and how to navigate the unique design challenges that AR's spatial, movement-based, and environment-dependent nature creates. The session is illustrated with real-world app examples including Monster Park, Color Snap, Warby Parker, IKEA Place, DoodleLens, AR Quick Look, and RoomPlan.

## Key Topics

### When to Use AR
AR adds genuine value when an experience benefits from one or more of the following qualities:

1. **True representation** — presenting real-world size, scale, or physical accuracy that cannot be conveyed as effectively by a flat image or text (e.g., a life-size animated dinosaur in Monster Park).
2. **Involving physical space** — the experience responds to and depends on the user's actual surroundings (e.g., Color Snap previewing paint colors on a user's real walls).
3. **Visualizing in 3D** — helping users understand, evaluate, or try on objects in three dimensions (e.g., Warby Parker virtual glasses try-on, IKEA Place furniture placement with ARKit lighting and shadows).
4. **Streamlining actions** — attaching digital capabilities to physical objects to reduce friction (e.g., iOS Measure app displaying height with minimal UI when centered on a person).

AR can be the primary focus of an entire app or a single lightweight feature. Both approaches are valid.

### Design Tips for AR

**Coach users toward a good environment**
- Guide users through setup requirements before starting AR: safety awareness, surface texture quality (textured surfaces track better than smooth/white/glass), and adequate lighting.
- Keep the coaching sequence short and easy to navigate.
- LiDAR-equipped devices can overcome some challenging surface conditions; consider omitting certain warnings for LiDAR devices.

**Leverage screen space**
- Place text, buttons, and interactive elements in 2D screen space on top of the 3D scene, not anchored to world coordinates, to preserve legibility as the camera moves.
- If text must be attached to a world object, keep it "billboarded" (always parallel to the screen), increase contrast, bump up type sizes, and add a background behind text.

**Design for constant movement**
- Provide real-time visual and audio feedback as the user moves the device.
- Show simple, glanceable instructions and animations in screen space, on an as-needed basis — not all upfront.
- Leverage ARKit's built-in coaching animations, or create custom onboarding animations relevant to the app's specific interaction (e.g., DoodleLens shows an iPhone panning across a doodle).

**Ergonomics**
- Design all UI to be legible and tappable at arm's length.
- Simplify interactions to minimize sustained effort.
- Use oversized buttons with high-contrast icons, positioned for comfortable one-handed thumb reach (place primary controls near the bottom of screen).
- Holding a phone at arm's length is fatiguing; keep sessions under one to two minutes when possible.

**Handle limited field of view**
- Allow users to pinch-scale objects that are larger than the viewable area (with a haptic feedback when scaling past 100%).
- When virtual objects are out of frame, use sounds, haptics, and minimal screen-space indicators or directional arrows.
- Provide a minimap or bird's-eye view showing the user's orientation relative to off-screen objects (e.g., RoomPlan's small 3D model preview in the lower screen).

**Depth cues for spatial understanding**
- Use realistic object size, perspective, and proper texturing to convey distance.
- Implement accurate shadows and lighting (ARKit provides built-in environment-based lighting).
- Use occlusion (overlapping) so virtual objects realistically appear behind real-world objects (e.g., AR Quick Look airplane hidden behind wooden blocks on a desk).

**Limit session duration**
- AR is battery- and thermal-intensive; target experiences of one to two minutes.
- For longer experiences, build in rest chapters or natural pause points (e.g., "For All Mankind: Time Capsule" chapters).

## APIs & Frameworks

**ARKit**
- Environment-based lighting estimation — provides realistic shadows and reflections for virtual content
- LiDAR scene reconstruction — enables AR on challenging surfaces with improved plane detection
- Built-in plane detection coaching animations — guide users to move device for feature-point detection
- `ARCoachingOverlayView` — standard UI for coaching users to find trackable surfaces **[available since ARKit 3]**

**RoomPlan** **[NEW in iOS 16]**
- `RoomCaptureSession` — capture a room's 3D structure; used in session as illustration of a minimap/bird's-eye progress view
- See companion session "Create parametric 3D room scans with RoomPlan" (WWDC22-10127) for full API detail

**AR Quick Look**
- Built-in system viewer for USDZ 3D models in Safari, Mail, Messages, Files
- Supports occlusion, environment lighting, and scale gestures with haptic feedback

**UIKit / SwiftUI**
- Standard gesture recognizers for scale (pinch), rotation, and pan used for direct manipulation of AR objects
- `UIImpactFeedbackGenerator` / `UINotificationFeedbackGenerator` — haptic feedback when scaling, placing objects

## Code Highlights
This session is a design guidance talk with no code samples. Its companion technical sessions are:
- **Discover ARKit 6** (WWDC22-10126) — new ARKit APIs including 4K video capture and motion capture improvements
- **Create parametric 3D room scans with RoomPlan** (WWDC22-10127) — `RoomCaptureSession` full API walkthrough

## Takeaways
- Use AR when you need true-to-scale representation, meaningful use of physical space, 3D visualization, or environment-connected interactions — not merely because it is impressive.
- Place all text and interactive controls in 2D screen space, not locked to world coordinates, to maintain legibility while the camera moves.
- Coach users through environmental requirements (safety, surface texture, lighting) before starting any AR session; keep coaching brief and sequential.
- Depth cues — especially occlusion (virtual objects hidden behind real objects), realistic shadows, and proper scale — are the primary tools for making virtual objects feel physically grounded.
- Keep AR sessions under two minutes for ergonomic and thermal reasons; for longer experiences, structure distinct rest chapters.

---
_Source: WWDC22 Session 10131 page (transcript, design guidance, and related session links)._
