# Modernizing Your Audio App
**WWDC19 · Session 508** · [Watch](https://developer.apple.com/videos/play/wwdc2019/508/)

_Platforms:_ macOS Catalina 10.15, iOS 13, iPadOS 13

## Overview
This short session is a targeted deprecation and migration notice from Apple's Core Audio team, covering the full stack of audio APIs on Apple platforms. Three major changes land in macOS Catalina / the next OS release: Carbon-component-based Audio Units are on a removal timeline, `AudioHardwarePlugIn`-based plug-ins are disabled by default, and `AUGraph`, Inter-App Audio, and `OpenAL` are formally deprecated. For each deprecated technology the session names the direct modern replacement.

The session is intentionally brief — a single engineer delivering a set of clear action items — rather than a technical deep dive. Developers maintaining audio apps or Audio Unit plug-ins should treat this as a compliance checklist to audit before their next release.

## Key Topics
- **Carbon Audio Unit removal** — Carbon-component-based AU hosts must switch to the `AudioComponent` API for AU discovery and instantiation; Carbon AU support will be removed in a future macOS release
- **`AudioHardwarePlugIn` disabled by default** — system audio plug-ins using the legacy `AudioHardwarePlugIn` mechanism are now disabled in macOS Catalina by default; can be temporarily re-enabled in Audio MIDI Setup, but full removal is coming; migrate to `AudioServerPlugin`
- **`AUGraph` deprecated** — the high-level Audio Unit graph/routing API is deprecated; migrate to `AVAudioEngine`
- **Inter-App Audio deprecated** — the iOS multi-app audio routing mechanism is deprecated; migrate to Audio Unit Extensions (AUv3)
- **`OpenAL` deprecated** — the cross-platform 3D audio API is deprecated on Apple platforms; migrate to `AVAudioEngine` with `AVAudioEnvironmentNode` for spatialized audio
- **3D Mixer parameter unification** — 3D Mixer Audio Unit parameters have been unified across all platforms; some parameters added, some deprecated; consult `AudioUnitParameters.h` for the updated parameter list

## APIs & Frameworks

### Deprecated (macOS Catalina / iOS 13)
- `AUGraph` — **[DEPRECATED]** replace with `AVAudioEngine`
- Inter-App Audio — **[DEPRECATED]** replace with Audio Unit Extensions (AUv3 / `AUAudioUnit`)
- `OpenAL` — **[DEPRECATED]** replace with `AVAudioEngine` + `AVAudioEnvironmentNode`
- Carbon-component-based Audio Units — **[REMOVAL PENDING]**
- `AudioHardwarePlugIn` — **[DISABLED BY DEFAULT]** replace with `AudioServerPlugin`

### Recommended Replacements
- **`AVAudioEngine`** — high-level Swift/Obj-C audio graph; replacement for both `AUGraph` and `OpenAL`
  - `AVAudioNode` subclasses: `AVAudioPlayerNode`, `AVAudioMixerNode`, `AVAudioEnvironmentNode`
  - `AVAudioEnvironmentNode` — positional 3D audio; replaces `OpenAL` listener/source model
  - `connect(_:to:format:)` — wire nodes in the graph
  - `AVAudioEngine.start()` / `stop()` / `reset()`
- **`AudioComponent` API** — AU discovery and instantiation
  - `AudioComponentFindNext(_:_:)` — enumerate available Audio Units
  - `AudioComponentInstanceNew(_:_:)` — instantiate a component
  - `AVAudioUnit.instantiate(with:options:completionHandler:)` — Swift-friendly async instantiation
- **Audio Unit Extensions (AUv3)** — `AUAudioUnit` subclass in an App Extension; replaces Inter-App Audio
  - `AUAudioUnit` — base class for third-party AU implementations
  - `AUAudioUnitBusArray` — input/output bus management
  - `AUAudioUnit.shouldChangeToFormat(_:for:)` — negotiate stream format
- **`AudioServerPlugin`** — macOS system audio plug-in API; replaces `AudioHardwarePlugIn`
  - Audio MIDI Setup utility — temporary re-enable path for legacy plug-ins during transition

### Audio Unit Parameters
- `AudioUnitParameters.h` — updated header with unified 3D Mixer parameter list across iOS, macOS, tvOS
- Some 3D Mixer parameters added; some deprecated — consult header for per-platform availability

## Code Highlights

```swift
// Replacing AUGraph with AVAudioEngine
import AVFoundation

let engine = AVAudioEngine()
let player = AVAudioPlayerNode()
let mixer  = engine.mainMixerNode

engine.attach(player)
engine.connect(player, to: mixer, format: nil)

try engine.start()
player.scheduleFile(audioFile, at: nil)
player.play()
```

```swift
// Instantiating an Audio Unit via AudioComponent API (replaces Carbon AUGraph host pattern)
var desc = AudioComponentDescription(
    componentType: kAudioUnitType_Effect,
    componentSubType: kAudioUnitSubType_Reverb2,
    componentManufacturer: kAudioUnitManufacturer_Apple,
    componentFlags: 0, componentFlagsMask: 0)

AVAudioUnit.instantiate(with: desc, options: []) { avAudioUnit, error in
    guard let unit = avAudioUnit else { return }
    engine.attach(unit)
    engine.connect(playerNode, to: unit, format: nil)
    engine.connect(unit, to: engine.mainMixerNode, format: nil)
}
```

```swift
// 3D spatialized audio with AVAudioEnvironmentNode (replaces OpenAL)
let environment = AVAudioEnvironmentNode()
engine.attach(environment)
engine.connect(player, to: environment, format: nil)
engine.connect(environment, to: engine.mainMixerNode, format: nil)

// Position the listener
environment.listenerPosition = AVAudio3DPoint(x: 0, y: 0, z: 0)
// Position a source
player.position = AVAudio3DPoint(x: 1, y: 0, z: -2)
```

## Takeaways
- Audit all uses of `AUGraph`, Inter-App Audio, and `OpenAL` immediately — they are formally deprecated in macOS Catalina / iOS 13 and will be removed in a future release; `AVAudioEngine` is the single replacement for all three.
- Any macOS system audio plug-in using `AudioHardwarePlugIn` is now disabled by default in Catalina; ship a migration to `AudioServerPlugin` before the next OS cycle removes the re-enable escape hatch.
- Carbon-component-based AU discovery must be replaced with the `AudioComponent` API; use `AVAudioUnit.instantiate(with:options:completionHandler:)` for the most Swift-friendly path.
- Check `AudioUnitParameters.h` for the updated, now-unified 3D Mixer parameter list if your app reads or writes 3D Mixer parameters directly.

---
_Source: WWDC19 Session 508 page (full transcript, abstract, and resource links)._
