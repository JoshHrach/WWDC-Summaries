# Explore advances in RealityKit
**WWDC26 · Session 279** · [Watch](https://developer.apple.com/videos/play/wwdc2026/279/)

_Platforms:_ visionOS 27, iOS 27, macOS 27

## Overview
This session surveys the major new capabilities added to RealityKit for WWDC26, organized into six technical areas: lighting and shadows, navigation mesh, cloth simulation, performance, 3D Gaussian splats, and immersive audio. Each section combines conceptual explanation with concrete Swift code examples drawn from the "Chaparral Village" and "Alchemist's Lab" demo scenes.

Lighting gains lightmap support for baked indirect lighting and ambient occlusion, new soft shadows for dynamic lights with configurable softness quality, projective textures for spotlights, and the flagship physical space lighting feature (`SpotLightComponent.SurroundingsLight`) that lets virtual lights illuminate the actual real-world environment around the user. Navigation mesh introduces a complete pathfinding API for NPC movement. Cloth simulation brings interactive fabric with kinematic pinning.

Performance tools include a new `LevelOfDetailComponent` with both camera-distance and screen-area algorithms, plus a pattern for responding to device thermal state changes. 3D Gaussian Splatting lands as a first-class rendering primitive via `GaussianSplatResource` and `GaussianSplatComponent`. Immersive audio gains customizable acoustic reverb meshes (`ReverbMeshResource`, `ReverbComponent`) with both preset and fully custom material absorption/scattering coefficients.

## Key Topics

### Lighting and Shadows
- **Lightmaps**: baked indirect lighting and ambient occlusion for static geometry; generated in Reality Composer Pro 3.
- **Soft shadows**: `SpotLightComponent.Shadow.lightSize` controls the angular size of the light source; `quality` selects sample count (.low = hard, .medium, .high).
- **Projective textures**: `SpotLightComponent.ProjectiveTexture(texture:)` projects a `TextureResource` through a spotlight (e.g., gobo patterns, star fields).
- **Physical space lighting**: `SpotLightComponent.SurroundingsLight()` component extends a virtual light's influence into the real-world passthrough environment.

### Navigation Mesh
- `NavigationMeshResource` defines walkable surfaces; generated from scene geometry in Reality Composer Pro 3 or at runtime.
- `NavigationComponent` added to NPC entities.
- `NavigationController(entity:)` — async `computePath(from:to:)` returns an array of path nodes.
- Node categories: `.meshPoint` (standard waypoint) and `.offMeshConnection` (ladders, jumps).

### Cloth Simulation
- `ClothBodyComponent`: attach to a mesh entity to enable real-time cloth dynamics.
- `ClothColliderComponent`: add to entities that interact with cloth.
- `ClothSphereShape(radius:)`: select vertices within a sphere for kinematic pinning.
- `clothBody.motionTypes.set(vertexIndices:value:.kinematic)`: pin cloth to anchor points.

### Performance
- `LevelOfDetailComponent.addByCameraDistance(to:levels:)` **[NEW]**: switch entity detail by distance (meters).
- `LevelOfDetailComponent.addByScreenArea(to:levels:)` **[NEW]**: switch by fraction of screen area.
- Thermal state monitoring: `ProcessInfo.thermalState` with `NotificationCenter` observation of `.thermalStateDidChange` — respond by increasing LOD aggressiveness or lowering shadow quality.

### 3D Gaussian Splats
- `GaussianSplatResource.BufferResource(count:position:scale:rotation:opacity:sphericalHarmonics:)` **[NEW]**: construct from raw buffers.
- `GaussianSplatResource(_:)` wraps the buffer resource.
- `GaussianSplatComponent(splatResource)` **[NEW]**: attach to entity.
- Supports spherical harmonics for view-dependent color (degree configurable).
- See also: [Gaussian splats on visionOS](https://developer.apple.com/documentation/visionOS/gaussian-splats-on-visionos) sample.

### Immersive Audio
- `ReverbMeshResource` **[NEW]**: describes acoustic geometry; preset shapes include `.shoebox(size:)`.
- `ReverbComponent(reverb:)` **[NEW]**: attach to entity; reverb can be `.simulated(mesh:materials:)`.
- Preset materials: `.dryWall`, `.carpet`, and others.
- Custom materials: `Audio.Material(absorption:scattering:)` with `Audio.Absorption` (10-band octave coefficients) and `Audio.Scattering` (3 frequency bands).
- `audio.Material.scalingAbsorption { freq in ... }` modifier for parameterized adjustments.

## APIs & Frameworks

### RealityKit
- `SpotLightComponent.Shadow` — `lightSize: Float`, `quality: SpotLightComponent.Shadow.Quality` (.low/.medium/.high) **[NEW]**
- `SpotLightComponent.ProjectiveTexture(texture: TextureResource)` **[NEW]**
- `SpotLightComponent.SurroundingsLight()` **[NEW]**
- `NavigationMeshResource` **[NEW]**
- `NavigationComponent` **[NEW]**
- `NavigationController(entity:)` — `computePath(from:to:) async` — `PathNode.category` (.meshPoint/.offMeshConnection) **[NEW]**
- `ClothBodyComponent` **[NEW]**
- `ClothColliderComponent` **[NEW]**
- `ClothSphereShape(radius:)` **[NEW]**
- `ClothBody.MotionTypes.set(vertexIndices:value:)` — `.kinematic` motion type **[NEW]**
- `LevelOfDetailComponent.addByCameraDistance(to:levels:)` **[NEW]**
- `LevelOfDetailComponent.addByScreenArea(to:levels:)` **[NEW]**
- `GaussianSplatResource` / `GaussianSplatResource.BufferResource` **[NEW]**
- `GaussianSplatComponent` **[NEW]**
- `ReverbMeshResource` — `.shoebox(size:)` **[NEW]**
- `ReverbComponent` — `.simulated(mesh:materials:)` **[NEW]**
- `Audio.Material` — `.dryWall`, `.carpet`, `.scalingAbsorption` **[NEW]**
- `Audio.Absorption([...])` — 10-band octave absorption coefficients **[NEW]**
- `Audio.Scattering([freq: value])` — per-frequency scattering **[NEW]**
- `ShaderGraphMaterial.setParameter(name:value:)` — existing, used for cloth/water surface integration
- `ModelComponent`, `Entity.components`, `Entity.findEntity(named:)` — existing

### Foundation / ProcessInfo
- `ProcessInfo.thermalState` — `.nominal`, `.fair`, `.serious`, `.critical`
- `Notification.Name.thermalStateDidChange`

## Code Highlights

Soft shadows with configurable quality:
```swift
shadow.lightSize = 0.7 // meters
shadow.quality = .medium
hearthSpotlight.components.set(shadow)
```

Physical space lighting:
```swift
spotLightEntity.components.set(SpotLightComponent.SurroundingsLight())
```

LOD by camera distance:
```swift
LevelOfDetailComponent.addByCameraDistance(to: entity, levels: [
    (entities: lod0, maxDistance: 1.0),
    (entities: lod1, maxDistance: 5.0),
    (entities: lod2, maxDistance: .infinity),
])
```

Custom reverb mesh and materials:
```swift
let mesh: ReverbMeshResource = .shoebox(size: [5, 4, 6])
let reverb: Reverb = .simulated(mesh: mesh, materials: [.dryWall])
entity.components.set(ReverbComponent(reverb: reverb))
```

Gaussian splat creation:
```swift
let resource = try GaussianSplatResource.BufferResource(
    count: splatCount, position: positionBuffer,
    scale: scaleBuffer, rotation: rotationBuffer,
    opacity: opacityBuffer, sphericalHarmonics: (harmonicsBuffer, degree))
splatEntity.components.set(GaussianSplatComponent(GaussianSplatResource(resource)))
```

## Takeaways
- Physical space lighting (`SurroundingsLight`) is the most impactful new rendering feature — it makes virtual lights feel integrated with the real world rather than floating in a separate layer.
- Navigation mesh and cloth simulation bring game-engine-class simulation capabilities into RealityKit without requiring a third-party engine.
- `LevelOfDetailComponent` and thermal state observation provide a structured path to shipping apps that perform well across varying hardware conditions.
- Acoustic reverb meshes allow spatial audio to accurately model the acoustic properties of virtual environments, closing a major parity gap with traditional game engines.

---
_Source: WWDC26 Session 279 page (abstract, chapter summaries, code samples, and resource links)._
