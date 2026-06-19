# Discover RealityKit APIs for iOS, macOS, and visionOS
**WWDC24 · Session 10103** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10103/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2

## Overview
This session introduces a broad wave of new RealityKit APIs, demonstrated by building a Spaceship game from scratch. It covers hover effects, physics force effects and joints, dynamic lights and shadows, portal crossing, cross-platform support, and more. Many of these features were directly driven by developer feedback since the launch of Apple Vision Pro.

A key headline is **cross-platform parity**: the same RealityKit code — hover effects, force effects, joints, lights, portals — now compiles and runs on iOS, iPadOS, macOS, and visionOS with minimal conditional code, making it practical to share a spatial experience across all Apple platforms.

## Key Topics

### Hover Effects and Input
`HoverEffectComponent` gains two new styles alongside the existing spotlight: `.highlight` (uniform tint overlay with configurable color and strength) and `.shader` (integrates a Reality Composer Pro ShaderGraph material, enabling the new `HoverState` node). A `SpatialTrackingSession` allows hand tracking directly in RealityKit without ARKit setup, enabling custom gesture-based input.

### Force Effects and Physics Joints
`ForceEffectProtocol` lets apps define custom force fields (e.g., inverse-square gravity). Built-in effects include constant radial, vortex, drag, and turbulence. The new `PhysicsCustomJoint` lets developers constrain angular and linear motion per axis independently, going beyond the four built-in joint types (fixed, spherical, revolute, prismatic, distance).

### Dynamic Lights and Shadows
`SpotLightComponent`, `DirectionalLightComponent`, and `PointLightComponent` now cast dynamic shadows on visionOS. `DynamicLightShadowComponent(castsShadow: false)` opts individual entities out of shadow casting.

### Portal Enhancements
`PortalComponent` now supports `crossingMode: .plane(.positiveZ)` and `clippingMode: .plane(.positiveZ)` for smooth object crossing. `PortalCrossingComponent` on an entity enables it to straddle the portal boundary. `EnvironmentLightingConfigurationComponent` smooths lighting transitions as an object approaches the portal.

### Cross-Platform Capabilities
`RealityView` is available on iOS/iPadOS/macOS with new `camera` mode options. Setting `content.camera = .worldTracking` enables AR on iOS without code changes to the RealityKit scene logic.

## APIs & Frameworks

**RealityKit**
- `HoverEffectComponent` — expanded styles **[NEW]**
  - `.highlight(HighlightHoverEffectStyle)` **[NEW]**
  - `.shader(.default)` **[NEW]**
  - `HoverEffectComponent.HighlightHoverEffectStyle(color:strength:)` **[NEW]**
- `HoverState` node in ShaderGraph (Reality Composer Pro) **[NEW]**
- `SpatialTrackingSession` **[NEW]**
- `ForceEffectProtocol` **[NEW]**
  - `parameterTypes: PhysicsBodyParameterTypes`
  - `forceMode: ForceMode`
  - `func update(parameters: inout ForceEffectParameters)`
- `ForceEffect(effect:spatialFalloff:mask:)` **[NEW]**
- `ForceEffectComponent(effects:)` **[NEW]**
- `SpatialForceFalloff(bounds:)` **[NEW]**
- `PhysicsMotionComponent(linearVelocity:)` (existing, highlighted)
- `Entity.pins.set(named:position:)` **[NEW]**
- `PhysicsCustomJoint(pin0:pin1:)` **[NEW]**
  - `.angularMotionAroundX/Y/Z: .range(...)` or `.fixed`
  - `.linearMotionAlongX/Y/Z: .fixed`
  - `joint.addToSimulation()` **[NEW]**
- Built-in joints: `PhysicsFixedJoint`, `PhysicsSphericalJoint`, `PhysicsRevoluteJoint`, `PhysicsPrismaticJoint`, `PhysicsDistanceJoint`
- `SpotLightComponent(color:intensity:attenuationRadius:)` **[NEW on visionOS]**
- `SpotLightComponent.Shadow()` **[NEW]**
- `DirectionalLightComponent`, `PointLightComponent`
- `DynamicLightShadowComponent(castsShadow:)` **[NEW]**
- `PortalComponent(target:clippingMode:crossingMode:)` **[NEW params]**
- `PortalCrossingComponent()` **[NEW]**
- `EnvironmentLightingConfigurationComponent` **[NEW]**
  - `.environmentLightingWeight: Float`
- `LowLevelMesh`, `LowLevelTexture` **[NEW]**
- `BillboardComponent` **[NEW]**
- `PixelCast` **[NEW]**
- Subdivision surface rendering **[NEW]**
- `RealityView(camera:)` with `.worldTracking` mode **[NEW on iOS]**
- `Entity.addForce(_:relativeTo:)` (existing)
- `Entity.position(relativeTo:)` (existing)

## Code Highlights

```swift
// Highlight hover effect
let highlightStyle = HoverEffectComponent.HighlightHoverEffectStyle(color: .lightYellow, strength: 0.8)
spaceship.components.set(HoverEffectComponent(.highlight(highlightStyle)))

// Custom gravity force effect
struct Gravity: ForceEffectProtocol {
    var parameterTypes: PhysicsBodyParameterTypes { [.position, .distance] }
    var forceMode: ForceMode = .force
    func update(parameters: inout ForceEffectParameters) {
        for i in 0..<parameters.physicsBodyCount {
            parameters.setForce(computeForce(parameters.distances![i], parameters.positions![i]), index: i)
        }
    }
}

// Portal crossing
portal.components.set(PortalComponent(target: portalWorld,
                                      clippingMode: .plane(.positiveZ),
                                      crossingMode: .plane(.positiveZ)))
spaceship.components.set(PortalCrossingComponent())
```

## Takeaways
- Use the new `.highlight` or `.shader` hover effect styles to give 3D entities richer visual feedback without affecting privacy.
- Adopt `ForceEffectProtocol` for custom physics fields (gravity wells, custom attractors) instead of per-frame manual force application.
- Enable dynamic lights and shadows to ground virtual content in the scene — but profile carefully as they are expensive.
- Set `content.camera = .worldTracking` in `RealityView` to bring an existing visionOS experience to iOS AR with minimal changes.

---
_Source: WWDC24 Session 10103 page (abstract, chapter summaries, code samples, and resource links)._
