# Developing CarPlay Systems, Part 2
**WWDC16 · Session 723** · [Watch](https://developer.apple.com/videos/play/wwdc2016/723/)

_Platforms:_ iOS 10, iOS 9 (wireless CarPlay)

## Overview
This deep-dive companion to Session 722 covers the protocols and resource management logic that head-unit developers must implement to integrate CarPlay correctly with a vehicle's native infotainment system. It explains the system architecture (how sensors, instrument clusters, displays, microphones, and speakers relate to both CarPlay and the native sub-system), the volume management model, the two-resource-managed-resource model (display and main audio), and the iAP2 `changeModes`/`modesChanged` command pair that governs which sub-system owns which hardware at any moment. Finally it covers application state management for turn-by-turn navigation, phone calls, and voice recognition.

## Key Topics

### System Overview
- The head unit has two sub-systems that share hardware: the native UI/audio system and the CarPlay receiver.
- Hardware resources shared between the two: primary display, speakers, microphone.
- Resources NOT managed (always available): secondary displays (instrument cluster, heads-up display), GNSS/sensor data forwarded over iAP2.
- Instrument cluster metadata new in iOS 10: next turn-by-turn direction can be shown on secondary displays **[NEW]**.
- Steering wheel controls (Siri button, next/previous track) can remain linked to CarPlay even when the native UI is on the main screen.

### Audio Architecture
Three shared audio categories:
1. **Telephony / speechRecognition** — bi-directional (microphone + speakers); used for phone calls and Siri.
2. **Media playback** — output only; used for music and other streaming content.
3. **Alerts / announcements** — output only; navigation prompts, ringtones.

Within CarPlay:
- **mainAudio** — encompassing phone call, Siri, and media audio; mutually exclusive by type at any moment.
- **alternateAudio** — navigation announcements and notifications; NOT managed, always available, always mixed on top of mainAudio; cannot be volume-adjusted when mainAudio is active.

Per-application volume contexts (volume knob controls different level depending on active audio):
- Siri voice volume
- Ringtone volume (during incoming call)
- Phone call audio volume (during call)
- Navigation prompt volume (during announcement)
- Media volume (all other times)

### Resource Management Model

Two managed resources:
- `mainScreen` — the primary in-car display.
- `mainAudio` — access to the car's audio system (sub-typed by use: `media`, `alert`, `speechRecognition`, `telephony`, `default`).

Transfer semantics:
- **Take** — permanent ownership; the owner holds the resource indefinitely until explicitly released.
- **Borrow** — temporary ownership; resource is returned when the use case ends.

Borrow constraints (set on `changeModes` to control what can preempt the resource):
- `anytime` — any application may borrow the resource.
- `userInitiated` — only user-triggered events may borrow.
- `never` — no application may borrow (e.g., backup camera showing).

### CarPlay Control Protocol Commands

Two symmetric commands (implemented over the iAP2 CarPlay Communication Plug-in):
- `changeModes` (accessory → controller / iPhone) — declares what the head unit wants to do with a resource: take, borrow, return, and under what borrow constraint.
- `modesChanged` (controller → accessory) — reports the current state of all managed resources; the head unit must not use a resource until a `modesChanged` confirms it is the owner.

Key principle: the iPhone (controller) hosts the resource manager. The head unit (accessory) always asks permission and acts only after receiving a `modesChanged` confirmation.

### Resource Switching Examples

| Scenario | Trigger | Screen result | Audio result |
|---|---|---|---|
| User presses native radio button | Hard key | Native UI | mainAudio → accessory (taken) |
| Native voice recognition starts | Hard key | Accessory borrows screen | Accessory borrows mainAudio |
| Native voice recognition ends | Dialogue complete | Returns to native UI | FM resumes (was owner) |
| Backup camera | Reverse gear | Screen borrowed, borrowConstraint = never | CarPlay audio continues |
| Siri triggered by user | Steering wheel button | Siri borrows screen | Siri borrows mainAudio (speechRecognition) |
| User asks Siri to play music | Voice command | Screen returns to accessory | mainAudio stays with controller for music |

Head unit must: after every `modesChanged`, check ownership; if owner, resume; if not, wait for CarPlay to act. Do not ignore phone-triggered resource switches.

### Application State Management (appStates)

Three shared application states, each preventing duplicate parallel operation:
- `TurnByTurn` — active route guidance; last-in-wins: if iPhone starts navigation, the head unit must stop its own.
- `PhoneCall` — active phone call; a CarPlay call will not ring audibly if a native BT-HFP call is already active.
- `Speech` — active voice recognition; if user triggers Siri while native VR is running, native VR ends and Siri takes over.

appState changes are reported via `modesChanged` and must be respected immediately.

## APIs & Frameworks

- **CarPlay Communication Plug-in** — Apple-provided source code; implements the CarPlay control protocol
- **iAP2** — Apple accessory protocol; transport for all CarPlay control messages (wired USB and wireless Wi-Fi)
- `changeModes` command — accessory requests resource change; fields: resource (`mainScreen`, `mainAudio`), transferType (`take`, `borrow`, `return`), borrowConstraint (`anytime`, `userInitiated`, `never`), audioType (`media`, `alert`, `speechRecognition`, `telephony`, `default`)
- `modesChanged` command — controller reports current resource ownership; accessory must not act until received
- `mainScreen` resource — managed primary display resource
- `mainAudio` resource — managed audio access resource
- `alternateAudio` — unmanaged; always mixed over `mainAudio`; used for navigation announcements
- appState: `TurnByTurn` — tracks active route guidance ownership
- appState: `PhoneCall` — tracks active phone call ownership
- appState: `Speech` — tracks active voice recognition ownership
- Instrument cluster / HUD metadata (iAP2) — turn-by-turn next direction **[NEW in iOS 10]**, now-playing track, active call info
- iAP2 `ExternalAccessoryProtocolCarPlay = true` — required head-unit declaration
- H.264 decoder, LPCM/AAC-LC/OPUS/AAC-ELD audio codecs — same as Part 1
- MFi Program — CarPlay specifications available to licensed manufacturers

## Code Highlights

This session is a system-design and protocol session; no app-level Swift/Objective-C code is shown. The key decision tree for head-unit firmware on receiving `modesChanged`:

```
on modesChanged received:
    if mainScreen.owner == accessory:
        resume native display
    else:
        pause native display, let CarPlay render

    if mainAudio.owner == accessory:
        resume native audio (FM, native music, etc.)
    else:
        mute native audio, let CarPlay play

    update TurnByTurn, PhoneCall, Speech appStates accordingly
```

## Takeaways
- The iPhone acts as the authoritative resource manager; the head unit must never use `mainScreen` or `mainAudio` before receiving a `modesChanged` confirmation that it is the owner.
- Use `borrowConstraint = never` only during scenarios requiring immediate user attention (e.g., backup camera); otherwise CarPlay features like phone calls will be completely blocked.
- Application state management (`TurnByTurn`, `PhoneCall`, `Speech`) prevents conflicting parallel operations — when iPhone wins an appState, the head unit must immediately stop its own competing application.
- `alternateAudio` (navigation announcements, notifications) is always available and always mixed; volume of `alternateAudio` cannot be adjusted independently while `mainAudio` is active.

---
_Source: WWDC16 Session 723 page (abstract, transcript, and resource links)._
