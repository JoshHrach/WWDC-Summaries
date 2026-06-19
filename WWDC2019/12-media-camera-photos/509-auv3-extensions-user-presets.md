# AUv3 Extensions User Presets
**WWDC19 · Session 509** · [Watch](https://developer.apple.com/videos/play/wwdc2019/509/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
iOS 13 and macOS Catalina add user preset support to Audio Unit v3 (AUv3) app extensions, complementing the existing factory preset system. A preset is a saved snapshot of an audio unit's parameter state that can be named, stored, recalled, and deleted. Unlike factory presets—which are bundled by the developer—user presets are created and managed by the end user and are accessible across all Audio Unit host applications (GarageBand, Logic Pro X, and any third-party host).

The new API is deliberately simple: audio unit developers opt in by setting a flag, and either accept the default storage behavior (the extension's app container) or override specific methods to implement custom persistence logic. Host applications gain matching methods to enumerate, create, and delete user presets on behalf of the user.

## Key Topics

**Factory Presets (Existing)**
- `factoryPresets` property on `AUAudioUnit` — array of `AUAudioUnitPreset` objects bundled by the developer
- Fixed collection; not editable by users

**User Presets (New in iOS 13 / macOS Catalina)**
- `userPresets` property — returns an array of `AUAudioUnitPreset` objects created by the user
- Cross-host: presets appear in every Audio Unit host app on the device
- Audio unit signals support by setting `supportsUserPresets = true`; hosts check this before calling user preset APIs

**Saving and Deleting Presets**
- `saveUserPreset(_:)` — host calls this to persist current AU state into a named preset
- `deleteUserPreset(_:)` — host calls this to remove an existing preset
- Default implementations store data in the AU extension's app container folder
- Both methods can be overridden for custom storage behavior

**Restoring Preset State**
- `presetState(for:)` — returns the state dictionary stored in a given user preset
- Restore by assigning the returned dictionary to `fullStateForDocument` on the audio unit
- Default implementation reads from the extension container; overridable

**In-Process Loading Check (macOS)**
- `isLoadedInProcess` property — verifies whether the AU loaded in-process (macOS only; host can request it but AU must support it; falls back to out-of-process if not)

## APIs & Frameworks

### AudioToolbox / AVFoundation — AUAudioUnit (NEW additions)
- `AUAudioUnit.userPresets: [AUAudioUnitPreset]` **[NEW]** — array of user-created presets
- `AUAudioUnit.supportsUserPresets: Bool` **[NEW]** — opt-in flag; AU sets to `true` to advertise support
- `AUAudioUnit.saveUserPreset(_ preset: AUAudioUnitPreset) throws` **[NEW]** — save current state; overridable
- `AUAudioUnit.deleteUserPreset(_ preset: AUAudioUnitPreset) throws` **[NEW]** — delete a preset; overridable
- `AUAudioUnit.presetState(for preset: AUAudioUnitPreset) throws -> [String: Any]` **[NEW]** — retrieve stored state dictionary; overridable
- `AUAudioUnit.fullStateForDocument: [String: Any]?` (existing) — used to restore state from a preset dictionary
- `AUAudioUnit.factoryPresets: [AUAudioUnitPreset]?` (existing) — developer-supplied read-only presets
- `AUAudioUnitPreset` (existing) — model object representing a named preset (number + name)
- `AUAudioUnit.isLoadedInProcess: Bool` **[NEW]** — macOS only; indicates in-process load success

### Sample Code
- AUv3 Host sample (updated) — available for both macOS and iOS; demonstrates full user preset workflow

## Code Highlights

Audio unit opting in to user preset support:
```swift
class MyAudioUnit: AUAudioUnit {
    override var supportsUserPresets: Bool { return true }
    // Default implementations of saveUserPreset, deleteUserPreset,
    // presetState(for:) are inherited from AUAudioUnit base class.
}
```

Host enumerating and restoring a user preset:
```swift
// List available user presets
let presets = audioUnit.userPresets

// Restore a specific preset
if let state = try? audioUnit.presetState(for: selectedPreset) {
    audioUnit.fullStateForDocument = state
}
```

Host saving and deleting:
```swift
let newPreset = AUAudioUnitPreset()
newPreset.name = "My Warm Sound"
try? audioUnit.saveUserPreset(newPreset)

try? audioUnit.deleteUserPreset(existingPreset)
```

## Takeaways
- User presets in AUv3 let musicians save and recall their own sounds across all host apps — a significant UX improvement over per-app preset systems.
- Adoption requires just setting `supportsUserPresets = true`; default storage in the extension container works without any extra code.
- Hosts must check `supportsUserPresets` before calling any user preset API to maintain backward compatibility.
- The `isLoadedInProcess` property on macOS lets hosts and extensions verify in-process loading succeeded after requesting it.

---
_Source: WWDC19 Session 509 page (abstract, full transcript, resource links including "Incorporating Audio Effects and Instruments" and "Creating custom audio effects" documentation)._
