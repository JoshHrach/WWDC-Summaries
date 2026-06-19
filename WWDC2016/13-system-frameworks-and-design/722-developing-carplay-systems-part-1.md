# Developing CarPlay Systems, Part 1
**WWDC16 · Session 722** · [Watch](https://developer.apple.com/videos/play/wwdc2016/722/)

_Platforms:_ iOS 10, iOS 9 (wireless CarPlay)

## Overview
This session provides an overview of how CarPlay works at the system level and the requirements automotive manufacturers must meet to build a great CarPlay infotainment system. It covers the data flows between iPhone and the head unit (video, audio, control, sensor data), the hardware requirements for displays and user input, the wired and wireless connection protocols, and Apple's design guidelines for integrating CarPlay alongside a car's native UI.

The session also covers the three categories of CarPlay apps (audio apps, automaker-specific apps, and the new messaging apps added in iOS 10), and explains the entitlement and iAP2 protocol requirements for automaker apps. A companion deep-dive session (723) covers the communication protocols and implementation details.

## Key Topics

### How CarPlay Works
- CarPlay begins the moment iPhone is plugged in (wired) or detected (wireless); all data flows are bidirectional.
- **Data streams**: H.264 video from iPhone to car; LPCM (wired) or compressed (wireless: AAC-LC for media, OPUS or AAC-ELD for voice) audio from iPhone to speakers; microphone audio from car to iPhone; iAP2 control protocol for commands, sensor data, now-playing metadata, turn-by-turn metadata.
- The CarPlay session remains active even when the car's native UI is displayed; audio continues even without the video stream.
- Communication plug-in is source code provided by Apple and integrated into the head unit.

### Display Requirements
- Supported resolutions: standard landscape 16:9 ratios (800×480, 960×540, 1280×720) and extra-wide displays; all landscape orientation.
- 24-bit color depth required; 60 Hz refresh rate strongly recommended.
- H.264 High Profile decoder required.
- Aftermarket displays must be at least 6 inches; OEM automakers set their own minimum.
- Per-resolution asset sizes are adjusted so icons appear the same physical size across resolutions.

### Audio
- **Main audio** (bi-directional): music, media, phone calls, Siri voice.
- **Alternate audio** (one-way): notifications; head unit must always mix with main audio.
- Wired: LPCM. Wireless: AAC-LC (media), OPUS or AAC-ELD (voice/Siri).

### User Input
- **High-fidelity touchscreen**: < 140 ms latency; supports swipe-to-scroll gestures that track the finger.
- **Low-fidelity touchscreen**: higher latency (often resistive); tap-only scrolling.
- **Knob**: minimum rotate + select + back events; optional tilt/nudge.
- **Touchpad**: reports x/y coordinates; requires select + back; can support character recognition.
- **Hardware buttons**: optional next, previous, telephony shortcuts; all should map to CarPlay equivalents.
- **Siri button** (required): push-to-talk on steering wheel; press-and-hold 600 ms to activate; activation within 1 second; head unit must send all button up/down events for mid-conversation interaction.

### Connectivity
- **Wired (USB)**: all data over USB; head unit must support USB role swap (host ↔ device); high throughput required; multiple-port vehicles must label CarPlay-capable ports.
- **Wireless (iOS 9+)**: Bluetooth for discovery and initial credential exchange; then all data over Wi-Fi (5 GHz recommended, 25 Mbps required); Wi-Fi Certified AP required.
- GNSS (GPS/GLONASS) data from the vehicle must be sent to iPhone if the vehicle has it; required for wireless CarPlay since the phone may not have clear satellite reception.

### Design Guidelines
- CarPlay must use the entire display — no mixing of CarPlay UI and native UI elements on the same screen.
- CarPlay starts automatically on first connection; never require the user to initiate it manually.
- **Last-user-mode**: on reconnection, restore CarPlay to display if it was the last active screen; resume CarPlay audio if it was the last active audio source.
- **Apple CarPlay button**: must appear in the top-level native menu immediately when CarPlay is active; must hide or disable when iPhone is disconnected.
- On iPhone disconnection: silence audio (do not fall back to another source); gracefully return to the last native screen.
- Physical Siri button behavior mirrors the iPhone Home button.

### CarPlay App Categories
1. **Audio apps**: any developer; use list-based UI and Now Playing screen fixed by iOS; require a CarPlay entitlement.
   - New in iOS 10: tab navigation in list hierarchy, app name on Now Playing screen, additional playback controls, streaming/explicit/live-stream badges, auto-play-on-launch support **[NEW]**.
2. **Automaker apps**: control vehicle functions; only appear in matching vehicles (matched via iAP2 external accessory protocol name); require a CarPlay entitlement.
   - New in iOS 10: Siri integration for radio station changes, climate control, etc. **[NEW]**.
3. **Messaging apps**: new in iOS 10 **[NEW]**; receive and compose messages entirely via Siri interaction; require a CarPlay entitlement.

### Automaker App Protocol Matching
- Head unit declares supported protocol names via iAP2 (e.g., `com.brand`, `com.brand.electric`).
- App `Entitlements.plist` must list the same protocol names.
- Head unit must support start/stop external protocol session; `ExternalAccessoryProtocolCarPlay = true` must be set in iAP2.
- If the vehicle supports both wired and wireless, iAP2 messages must be implemented on all transports.

## APIs & Frameworks

- **CarPlay** (system technology — no public API for head unit; entitlement-gated for apps)
- CarPlay entitlement — required for audio apps, automaker apps, and messaging apps
- **iAP2** — wired accessory communication protocol; used for control, audio metadata, turn-by-turn, sensor data, external accessory protocol negotiation
- `ExternalAccessoryProtocolCarPlay = true` — iAP2 flag required for automaker apps
- `EAAccessoryManager` / `EASession` — external accessory communication (automaker app ↔ head unit)
- **Bonjour / mDNS** — used in wireless CarPlay discovery
- **H.264 High Profile** — video codec (iPhone → head unit)
- **LPCM** — wired audio codec
- **AAC-LC** — wireless media audio codec
- **OPUS / AAC-ELD** — wireless voice/Siri audio codecs
- `NSAppTransportSecurity` / CarPlay-specific Info.plist keys — entitlement configuration
- **Siri / SiriKit** — new in iOS 10 for automaker app voice commands and messaging app integration **[NEW]**
- Tab navigation support in CarPlay audio app list views **[NEW in iOS 10]**
- Streaming/explicit/live-stream playback metadata badges **[NEW in iOS 10]**
- Auto-play-on-launch for audio apps **[NEW in iOS 10]**

## Code Highlights

Entitlements.plist for an automaker CarPlay app:
```xml
<key>com.apple.developer.carplay-audio</key>
<true/>
<key>com.apple.developer.carplay-messaging</key>
<true/>
<key>com.apple.external-accessory.wireless-configuration</key>
<array>
    <string>com.brand</string>
    <string>com.brand.electric</string>
    <string>com.brand.model2016</string>
</array>
```

## Takeaways
- CarPlay must always occupy the full display and start automatically on connection; "last-user-mode" restores the previous CarPlay display and audio state on reconnection.
- Siri is a required, first-class input method; a dedicated steering-wheel button with < 1-second activation latency is mandatory, and it must work whether or not CarPlay is visible on the display.
- iOS 10 adds messaging app support (entirely Siri-driven), Siri integration for automaker vehicle-control apps, and significantly improved audio app UI (tabs, streaming badges, auto-launch).
- For wireless CarPlay, GNSS data from the vehicle must be forwarded to iPhone if available, since the phone may be pocketed with poor satellite visibility.

---
_Source: WWDC16 Session 722 page (abstract, transcript, and resource links)._
