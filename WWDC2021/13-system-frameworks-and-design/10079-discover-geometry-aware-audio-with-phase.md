# Discover geometry-aware audio with the Physical Audio Spatialization Engine (PHASE)
**WWDC21 · Session 10079** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10079/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
PHASE (Physical Audio Spatialization Engine) is a new Apple framework for geometry-aware, event-driven, spatial audio. Unlike traditional game audio approaches that treat sound sources as hand-placed point sources requiring manual ray-tracing and blending, PHASE accepts mesh geometry for both sound sources and occluders and automatically computes occlusion, distance attenuation, directivity, early reflections, and late reverberation based on the scene geometry.

The session covers the three core pillars of the PHASE API (engine/asset registry, nodes for playback logic, and mixers for spatialization), demonstrates three progressively complex use cases (basic audio file playback, full spatial audio with a volumetric source and occluder, and a hierarchical behavioral sound event for a walking actor), and includes complete Swift code samples for each.

## Key Topics

### Why PHASE
- Traditional game audio: point sources, manual ray tracing to determine mix ratios, hand-tuned in post-production. When visuals change, audio must be re-authored.
- PHASE: audio sources are geometric shapes; the engine handles occlusion, spreading, reflections, and reverb automatically. Audio adapts to visual scene changes without re-authoring.
- Built on Apple's existing spatial audio rendering stack; automatically delivers consistent results across AirPods, headphones, and device speakers.

### Engine, Asset Registry, and Scene Graph
- `PHASEEngine(updateMode:)` — creates the engine in `.automatic` (SwiftUI/game loop driven) or `.manual` mode.
- **Asset registry**: register sound assets (`registerSoundAsset`) from URLs or raw data; register sound event assets (`registerSoundEventAsset`) from node hierarchies.
- **Scene graph**: `PHASEListener`, `PHASESource`, `PHASEOccluder` — all attached to `engine.rootObject` (or children) to activate within the simulation.
- `engine.start()` / `engine.stop()` — controls audio I/O; no audio flows until the engine is started.

### Nodes — Playback Control
Four node types compose a sound event hierarchy:
- **`PHASESamplerNodeDefinition`** (generator/leaf): plays a registered sound asset. Properties: `playbackMode` (`.oneShot` / `.looping`), `calibrationMode` (`.relativeSpl`), `level`, `cullOption` (`.terminate` / `.sleepWakeAtRealtimeOffset`).
- **`PHASERandomNodeDefinition`** (control): selects one child by weighted random choice on each trigger.
- **`PHASESwitchNodeDefinition`** (control): selects a child based on a `PHASEStringMetaParameterDefinition` value (e.g., terrain = "creaky_wood" vs. "soft_gravel").
- **`PHASEBlendNodeDefinition`** (control): blends between children based on a `PHASENumberMetaParameterDefinition` value (e.g., wetness 0–1).
- **`PHASEContainerNodeDefinition`** (control): plays all children simultaneously.

### Mixers — Spatialization
- **`PHASEChannelMixerDefinition`**: non-spatial playback to a channel layout (stereo music, dialogue).
- **`PHASEAmbientMixerDefinition`**: externalized audio with head-tracking but no distance modeling (background beds, ambient environment).
- **`PHASESpatialMixerDefinition`**: full spatialization with geometry-aware effects. Configured via a `PHASESpatialPipeline` that selects:
  - `.directPathTransmission` — direct and occluded path rendering.
  - `.earlyReflections` — specular reflections from walls/floors.
  - `.lateReverb` — diffuse room reverb; choose from presets via `engine.defaultReverbPreset`.
- Distance models: `PHASEGeometricSpreadingDistanceModelParameters` (natural attenuation + `rolloffFactor`) or piecewise curved segments.
- Directivity models: cardioid or cone.

### Volumetric Sources and Occluders **[NEW]**
- `PHASESource(engine:shapes:)` — volumetric source created from a `PHASEShape` (built from an `MDLMesh`). A point source is created by omitting the `shapes` parameter.
- `PHASEOccluder(engine:shapes:)` — occluder from a mesh; each shape element can have a `PHASEMaterial(engine:preset:)` (e.g., `.cardboard`, `.brick`, `.glass`) controlling sound absorption and transmission.
- Sources, listeners, and occluders are positioned using `simd_float4x4` transforms; PHASE computes occlusion and spreading automatically each frame.

## APIs & Frameworks

**PHASE (NEW framework)**
- `PHASEEngine` — root object managing all assets and simulation **[NEW]**
- `PHASEEngine.assetRegistry.registerSoundAsset(url:identifier:assetType:channelLayout:normalizationMode:)` **[NEW]**
- `PHASEEngine.assetRegistry.registerSoundEventAsset(rootNode:identifier:)` **[NEW]**
- `PHASESoundEvent(engine:assetIdentifier:mixerParameters:)` **[NEW]**
- `PHASEListener` **[NEW]**
- `PHASESource` (point and volumetric) **[NEW]**
- `PHASEOccluder` **[NEW]**
- `PHASEShape(engine:mesh:)` **[NEW]**
- `PHASEMaterial(engine:preset:)` — acoustic material presets **[NEW]**
- `PHASESamplerNodeDefinition` **[NEW]**
- `PHASERandomNodeDefinition` **[NEW]**
- `PHASESwitchNodeDefinition` **[NEW]**
- `PHASEBlendNodeDefinition` **[NEW]**
- `PHASEContainerNodeDefinition` **[NEW]**
- `PHASEChannelMixerDefinition` **[NEW]**
- `PHASEAmbientMixerDefinition` **[NEW]**
- `PHASESpatialMixerDefinition` **[NEW]**
- `PHASESpatialPipeline(options:)` **[NEW]**
- `PHASEGeometricSpreadingDistanceModelParameters` **[NEW]**
- `PHASEStringMetaParameterDefinition` / `PHASENumberMetaParameterDefinition` — runtime-controllable parameters **[NEW]**
- `PHASEMixerParameters.addSpatialMixerParameters(identifier:source:listener:)` **[NEW]**

## Code Highlights

Create an engine, register a sound asset, and play it back with a channel mixer:
```swift
let engine = PHASEEngine(updateMode: .automatic)

let audioFileUrl = Bundle.main.url(forResource: "DrumLoop_24_48_Mono", withExtension: "wav")!
try engine.assetRegistry.registerSoundAsset(url: audioFileUrl, identifier: "drums",
                                            assetType: .resident, channelLayout: nil,
                                            normalizationMode: .dynamic)

let channelLayout = AVAudioChannelLayout(layoutTag: kAudioChannelLayoutTag_Mono)!
let channelMixer = PHASEChannelMixerDefinition(channelLayout: channelLayout)
let sampler = PHASESamplerNodeDefinition(soundAssetIdentifier: "drums",
                                         mixerDefinition: channelMixer)
sampler.playbackMode = .looping
sampler.setCalibrationMode(.relativeSpl, level: 0)

try engine.assetRegistry.registerSoundEventAsset(rootNode: sampler, identifier: "drumEvent")
try engine.start()
let soundEvent = try PHASESoundEvent(engine: engine, assetIdentifier: "drumEvent")
try soundEvent.start()
```

Set up a volumetric source with a spatial mixer and a cardboard-box occluder:
```swift
// Spatial pipeline with late reverb
let pipeline = PHASESpatialPipeline(options: [.directPathTransmission, .lateReverb])!
pipeline.entries[PHASESpatialCategory.lateReverb]!.sendLevel = 0.1
engine.defaultReverbPreset = .mediumRoom
let spatialMixer = PHASESpatialMixerDefinition(spatialPipeline: pipeline)

// Volumetric source (HomePod mini-sized sphere)
let mesh = MDLMesh.newIcosahedron(withRadius: 0.0142, inwardNormals: false, allocator: nil)
let source = PHASESource(engine: engine, shapes: [PHASEShape(engine: engine, mesh: mesh)])
source.transform = /* 2m in front of listener */
try engine.rootObject.addChild(source)

// Cardboard box occluder
let boxMesh = MDLMesh.newBox(withDimensions: simd_make_float3(0.61, 0.30, 0.10),
                              segments: simd_uint3(repeating: 6),
                              geometryType: .triangles, inwardNormals: false, allocator: nil)
let boxShape = PHASEShape(engine: engine, mesh: boxMesh)
boxShape.elements[0].material = PHASEMaterial(engine: engine, preset: .cardboard)
let occluder = PHASEOccluder(engine: engine, shapes: [boxShape])
occluder.transform = /* 1m in front of listener */
try engine.rootObject.addChild(occluder)
```

Building a behavioral sound event (random terrain + wetness blend + noisy jacket):
```swift
// Switch between wood and gravel footsteps
let terrain = PHASEStringMetaParameterDefinition(value: "creaky_wood")
let terrainSwitch = PHASESwitchNodeDefinition(switchMetaParameterDefinition: terrain)
terrainSwitch.addSubtree(footstep_wood_random, switchValue: "creaky_wood")
terrainSwitch.addSubtree(footstep_gravel_random, switchValue: "soft_gravel")

// Blend between dry footsteps and splashes based on wetness
let wetness = PHASENumberMetaParameterDefinition(value: 0.5, minimum: 0, maximum: 1)
let wetnessBlend = PHASEBlendNodeDefinition(blendMetaParameterDefinition: wetness)
wetnessBlend.addRangeForInputValues(belowValue: 1, fullGainAtValue: 0,
                                    fadeCurveType: .linear, subtree: terrainSwitch)
wetnessBlend.addRangeForInputValues(aboveValue: 0, fullGainAtValue: 1,
                                    fadeCurveType: .linear, subTree: splashRandom)

// Container: blend + noisy jacket playing simultaneously
let actorContainer = PHASEContainerNodeDefinition()
actorContainer.addSubtree(wetnessBlend)
actorContainer.addSubtree(noisyClothingRandom)
```

## Takeaways
- PHASE replaces manual point-source management and hand-tuned audio post-production with geometry-driven simulation: pass meshes for sources and occluders, and the engine handles occlusion, distance, and reverb automatically.
- The node hierarchy (random, switch, blend, container) provides a sound-design-friendly, event-driven system for interactive audio that can be wired to game parameters at runtime.
- Volumetric sources remove the need to place multiple point sources along a surface and manually blend between them as the listener moves.
- All spatial audio rendering leverages Apple's existing head-tracking and binaural stack, providing consistent quality across AirPods, headphones, and speakers with no extra configuration.

---
_Source: WWDC21 Session 10079 page (abstract, full transcript, code samples, and resource links)._
