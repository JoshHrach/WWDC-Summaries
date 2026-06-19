# Plug-in and play: Add Apple frameworks to your Unity game projects
**WWDC22 · Session 10065** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10065/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
Apple introduced a new set of open-source Unity plug-ins at WWDC22 that allow Unity-based games to directly access six Apple frameworks via C# scripting: Apple.Core, Game Center (GameKit), Game Controller, Accessibility, Core Haptics, and PHASE. Each plug-in maps 1-to-1 with its underlying framework, so familiarity with either the Unity plug-in or the native framework transfers directly between them.

The plug-ins are distributed as Unity packages and are hosted on GitHub at `github.com/apple/unityplugins`. A Python build script (`build.py`) handles building native libraries, copying them to correct locations, updating Unity meta files, and packing plug-ins into tarballs. The workflow requires Xcode, Python 3, npm, and Unity. Apple.Core is a required dependency for all other plug-ins.

## Key Topics

### Apple.Core Plug-in (Foundation)
- Unifies build settings for all plug-ins in a single **Apple Build Settings** pane in Unity Project Settings
- Asset processor ensures native libraries are configured for the correct platform on import
- Post-process build scripts ensure native library references are correct in intermediate Xcode projects
- Defines runtime interop types for passing data between C# and native layers
- Required dependency — must be installed before any other Apple Unity plug-in

### Game Center Plug-in (GameKit)
- Player authentication via `GKLocalPlayer.Authenticate()` — async, call once per game lifetime
- Access player restriction flags: `IsUnderage`, `IsMultiplayerGamingRestricted`, `IsPersonalizedCommunicationRestricted`
- Also supports: achievements, leaderboards, challenges, multiplayer matchmaking
- Requires: add Game Center capability in Xcode, configure features in App Store Connect

### Game Controller Plug-in
- `GCControllerService.Initialize()` — initialize the controller service
- `GCControllerService.GetConnectedControllers()` — enumerate connected controllers
- `GCController.Poll()` — update input state; call each frame in Unity's `Update()` loop
- `GetButton(GCControllerInputName.ButtonSouth)` — poll individual button state
- `ControllerConnected` / `ControllerDisconnected` events for lifecycle management
- Supports MFi controllers and select third-party controllers (Sony, Microsoft)
- Includes support for controller customizations (button remapping) and button glyphs

### Accessibility Plug-in
- VoiceOver integration — programmatically tag content for screen reader playback
- Switch Control support
- Dynamic Type scaling for in-game text and UI
- UI accommodation settings to respect system-wide accessibility preferences
- Deep dive available in companion session "Add accessibility to your Unity games"

### Core Haptics Plug-in
- `CHHapticEngine` — link to the haptic server; create once, call `Start()`
- `CHHapticPattern` — logical grouping of haptic and audio events; create from AHAP asset
- `CHHapticPatternPlayer` — created from engine + pattern; call `Start()`, `Stop()`, `Pause()`, `Resume()`
- `AHAPAsset` — custom Unity asset for AHAP files; `[SerializeField]` to expose in Unity Inspector
- Inspector extension for tuning haptic patterns (add transient/continuous events, adjust parameters)
- Import/Export buttons for AHAP file sharing across team
- Supports real-time parameter adjustment during playback

### PHASE Plug-in (Physical Audio Spatialization Engine)
- `PHASEListener` component — attach to camera; processes audio based on position, orientation, reverb preset
- `PHASEOccluder` component — attach to geometry meshes; dampens audio when between source and listener
- `PHASESource` component — attach to game objects; uses transform to position sounds in world space
- `SoundEvent` asset — describes audio playback events; built in the PHASE Sound Event Composer window
- Sampler nodes + mixer nodes in sound event composer; looping, reverb, reflection support
- Zero-code path: attach components and wire up assets without writing any scripts

## APIs & Frameworks

**Apple Unity Plug-ins (github.com/apple/unityplugins)** **[NEW]**

Apple.Core (C#)
- `AppleSettings` — build configuration access

GameKit plug-in (C#) **[NEW]**
- `GKLocalPlayer.Authenticate()` — **[NEW]** `async Task<GKLocalPlayer>`
- `GKLocalPlayer.IsUnderage: bool`
- `GKLocalPlayer.IsMultiplayerGamingRestricted: bool`
- `GKLocalPlayer.IsPersonalizedCommunicationRestricted: bool`

Game Controller plug-in (C#) **[NEW]**
- `GCControllerService.Initialize()` — **[NEW]**
- `GCControllerService.GetConnectedControllers()` — returns `IEnumerable<GCController>`
- `GCControllerService.ControllerConnected` event
- `GCControllerService.ControllerDisconnected` event
- `GCController.Poll()` — update input state
- `GCController.GetButton(GCControllerInputName)` — **[NEW]** button state query
- `GCControllerInputName` enum (ButtonSouth, ButtonNorth, etc.)

Core Haptics plug-in (C#) **[NEW]**
- `CHHapticEngine` — **[NEW]** `new CHHapticEngine()`, `.Start()`
- `CHHapticPattern` — returned by `AHAPAsset.GetPattern()`
- `CHHapticPatternPlayer` — created via `CHHapticEngine.MakePlayer(pattern)`, `.Start()`, `.Stop()`
- `AHAPAsset` — custom Unity `ScriptableObject` for AHAP files; `[SerializeField]` attribute

PHASE plug-in (C#/Unity Components) **[NEW]**
- `PHASEListener` — MonoBehaviour component
- `PHASEOccluder` — MonoBehaviour component
- `PHASESource` — MonoBehaviour component
- `SoundEvent` — custom ScriptableObject asset; designed in Sound Event Composer
- Sampler node, Mixer node — composer graph nodes

**Native Apple Frameworks (underlying)**
- `GameKit` — Game Center services
- `GameController` — controller input and customization
- `Accessibility` — VoiceOver, Dynamic Type, Switch Control
- `CoreHaptics` — haptic engine, AHAP files
- `PHASE` — Physical Audio Spatialization Engine

## Code Highlights

Game Center authentication and player restrictions (C#):
```csharp
using Apple.GameKit;

public class GameManager : MonoBehaviour {
    private GKLocalPlayer _localPlayer;

    private async Task Start() {
        try {
            _localPlayer = await GKLocalPlayer.Authenticate();
            if (_localPlayer.IsUnderage) { /* hide explicit content */ }
            if (_localPlayer.IsMultiplayerGamingRestricted) { /* disable multiplayer */ }
            if (_localPlayer.IsPersonalizedCommunicationRestricted) { /* disable comms UI */ }
        } catch (Exception e) { /* handle error */ }
    }
}
```

Controller polling in Unity Update loop (C#):
```csharp
foreach (GCController controller in _myConnectedControllers) {
    controller.Poll();
    if (controller.GetButton(GCControllerInputName.ButtonSouth)) {
        // handle button press
    }
}
```

Core Haptics component (C#):
```csharp
using Apple.CoreHaptics;

public class Haptics : MonoBehaviour {
    private CHHapticEngine _hapticEngine;
    private CHHapticPatternPlayer _hapticPlayer;
    [SerializeField] private AHAPAsset _hapticAsset;

    private void PrepareHaptics() {
        _hapticEngine = new CHHapticEngine();
        _hapticEngine.Start();
        _hapticPlayer = _hapticEngine.MakePlayer(_hapticAsset.GetPattern());
    }

    private void Play() { _hapticPlayer.Start(); }
}
```

## Takeaways
- Six open-source Apple Unity plug-ins are now available on GitHub, each mapping directly to a native Apple framework; no bridging expertise required for teams familiar with the underlying APIs.
- Apple.Core is a required foundation plug-in that centralizes build settings, asset processing, and post-build steps; always install it first.
- PHASE and Core Haptics plug-ins include custom Unity Editor extensions (PHASE Sound Event Composer, AHAP inspector editor) that enable component-driven workflows without writing code.
- The plug-ins are designed so that learning the Unity C# API is equivalent to learning the underlying native framework, enabling knowledge transfer between Unity and native Apple development.

---
_Source: WWDC22 Session 10065 page (abstract, chapter summaries, code samples, and resource links)._
