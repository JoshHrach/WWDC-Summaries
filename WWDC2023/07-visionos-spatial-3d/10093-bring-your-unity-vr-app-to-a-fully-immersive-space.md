# Bring Your Unity VR App to a Fully Immersive Space
**WWDC23 · Session 10093** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10093/)

_Platforms:_ visionOS 1

## Overview
This session walks Unity developers through the steps required to bring an existing VR app or game to visionOS as a fully immersive experience. The session uses Rec Room (from Against Gravity) as the reference example, covering the build-and-run workflow, graphics adaptations specific to Apple Silicon, and the input model transition from traditional controllers to hand and eye tracking.

A fully immersive visionOS app hides passthrough entirely and replaces the user's surroundings with the app's own environment. Under the hood, Unity leverages Compositor Services for Metal rendering and ARKit for body position, surroundings, and skeletal hand tracking — all surfaced through Unity's engine layer so developers can use familiar Unity workflows.

## Key Topics

### Build and Run Workflow
- Select the visionOS build target in Unity, then enable the XR Plug-in for the platform.
- Native plug-ins must be recompiled for the platform; raw source code and `.mm` files work as-is.
- Building from Unity generates an Xcode project (same as iOS/macOS/tvOS targets); build and run from Xcode to the device or device simulator.
- Minimum Unity version: **Unity 2022 or later**.

### Graphics: Universal Render Pipeline and Foveated Rendering
- **Universal Render Pipeline (URP)** is the recommended choice; it enables **Static Foveated Rendering** throughout the entire pipeline including post-processing, camera stacking, and HDR.
- Foveated Rendering concentrates pixel density at the center of each lens where eyes are focused, reducing peripheral resolution to save GPU bandwidth and improve quality.
- Custom render passes can opt in to Foveated Rendering using new Unity 2022 APIs.
- Because rendering occurs in a nonlinear foveated space, Unity provides shader macros to handle the remapping; these are applied automatically in URP shaders.
- **Single-Pass Instanced Rendering** now supports the Metal graphics API and is enabled by default — one draw call covers both eyes, reducing CPU overhead for culling and shadows.
- Depth buffer correctness: the system compositor uses the depth buffer for reprojection. Every pixel must write a valid depth value. Unity has fixed all built-in shaders (including the skybox's reverse-Z case); custom effects (custom skybox, water, transparency) must be audited.

### Input: Hands and Eyes
Three levels of input abstraction are available:

1. **XR Interaction Toolkit (XRI)** — high-level interaction system: abstracts hand tracking into hover/grab/select/poke events, works with 3D and UI objects, platform-agnostic across VR platforms.
   - Interactable component types: `SimpleInteractable`, `GrabInteractable`, `TeleportArea`, `TeleportAnchor` (custom Interactables also supported).
   - Interactor types: `DirectInteractor` (touch/proximity), `RayInteractor` (far-field, curved or straight), `SocketInteractor` (world-anchored placement zones), `PokeInteractor` (directional poke with motion filtering), `GazeInteractor` (gaze-based with auto-enlarged colliders).
   - `InteractionManager` — mediates between all registered Interactors and Interactables; multiple managers can be activated/deactivated per scene.
   - `XRController` component — maps Input Action References (select, activate, etc.) from hands or tracked devices to Interactor state.
   - Includes a locomotion system for comfortable travel through a fully immersive space.

2. **Unity Input System (system gestures)** — bind to platform built-in gestures (e.g., pinch gesture with position/rotation, gaze focus position/rotation) via Input Action binding paths. Straightforward for simple tap/pinch interactions.

3. **Unity Hands Package** — raw hand joint data via `XRHandSubsystem`; low-level, consistent across platforms.
   - `XRHand.GetJoint(XRHandJointID)` — retrieve a named joint.
   - `XRHandJoint.TryGetPose(out Pose)` — get the joint's world pose.
   - Use for custom gesture recognition (e.g., detecting an extended index finger, mapping to a custom hand mesh).
   - Rec Room uses raw joint data to drive a stylized hand model and show other players' hands.

## APIs & Frameworks

### Unity / visionOS Platform Support **[NEW]**
- Unity 2022 visionOS build target — generates an Xcode project for the platform **[NEW]**
- XR Plug-in for visionOS — enables VR rendering pipeline on the platform **[NEW]**
- Compositor Services — system framework for Metal rendering in fully immersive apps (underlying layer Unity uses) **[NEW]**
- Unity Universal Render Pipeline (URP) — recommended rendering pipeline
- Static Foveated Rendering via URP **[NEW on visionOS]**
- Foveated Rendering APIs in Unity 2022 — opt-in for custom render passes **[NEW]**
- Single-Pass Instanced Rendering with Metal **[NEW support]**
- Shader macros for nonlinear foveated space remapping **[NEW]**

### ARKit (via Unity)
- ARKit skeletal hand tracking — body position and hand skeleton data (surfaced through Unity layer) **[NEW on visionOS]**

### XR Interaction Toolkit (XRI)
- `XRInteractionManager` — mediates Interactor/Interactable state changes
- `XRSimpleInteractable` — basic hover/select/activate events
- `XRGrabInteractable` — grab and throw with velocity inheritance
- `TeleportArea` / `TeleportAnchor` — locomotion teleport targets
- `XRRayInteractor` — far-field ray casting, curved/straight, configurable visualization
- `XRDirectInteractor` — proximity/touch interaction
- `XRSocketInteractor` — world-anchored socket for object placement
- `XRPokeInteractor` — directional poke with motion filtering **[NEW on visionOS]**
- `XRGazeInteractor` — gaze-based interaction with auto-enlarged colliders **[NEW on visionOS]**
- `XRController` — maps Input Action References to Interactor state

### Unity Input System
- Input Action binding paths for system gestures (pinch gesture value, pinch position/rotation, gaze position/rotation)

### Unity Hands Package
- `XRHandSubsystem` — raw hand joint data access
- `XRHand` — hand data container
- `XRHandJoint` — individual joint with pose data
- `XRHandJointID` — enum of all named hand joints (e.g., `.Wrist`, `.IndexTip`, `.IndexIntermediate`)
- `XRHandJoint.TryGetPose(out Pose)` — get joint world pose
- `OnHandUpdate` event — called each frame with updated hand data

## Code Highlights

Detecting an extended index finger from raw hand joint data:
```csharp
static bool IsIndexExtended(XRHand hand)
{
    if (!(hand.GetJoint(XRHandJointID.Wrist).TryGetPose(out var wristPose) &&
          hand.GetJoint(XRHandJointID.IndexTip).TryGetPose(out var tipPose) &&
          hand.GetJoint(XRHandJointID.IndexIntermediate).TryGetPose(out var intermediatePose)))
    {
        return false;
    }
    var wristToTip = tipPose.position - wristPose.position;
    var wristToIntermediate = intermediatePose.position - wristPose.position;
    return wristToTip.sqrMagnitude > wristToIntermediate.sqrMagnitude;
}
```

## Takeaways
- Start with Unity 2022 and adopt the Universal Render Pipeline to get Static Foveated Rendering and correct Metal Single-Pass Instanced Rendering automatically.
- Audit every custom shader and effect for depth buffer correctness — missing depth values produce visible error colors in the compositor.
- Use the XR Interaction Toolkit as the first step for adapting controller interactions to hand/eye input; reach for the Unity Hands Package only when you need custom gesture recognition or a custom hand mesh.
- Apply for Unity's visionOS beta at create.unity.com/spatial to access the platform-specific plug-in and sample code bundles.

---
_Source: WWDC23 Session 10093 page (abstract, chapter summaries, code samples, and resource links)._
