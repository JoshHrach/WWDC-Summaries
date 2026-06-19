# Meet the Next Generation of CarPlay Architecture
**WWDC24 · Session 10111** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10111/)

_Platforms:_ Next-generation CarPlay vehicle systems (automaker/system developer audience)

## Overview
This session is aimed at automakers and vehicle system developers building for the next generation of CarPlay. It explains how the new CarPlay architecture enables a single cohesive in-vehicle experience that combines the best of both the automaker's native vehicle UI and iPhone, rather than running iPhone apps in a separate mode as in the current generation of CarPlay.

The next generation of CarPlay renders all instrument cluster and infotainment displays from iPhone directly — including the vehicle's speedometer, fuel gauge, temperature gauges, and maps — while the vehicle system remains responsible for data acquisition (vehicle sensors) and audio output hardware.

## Key Topics

**Architecture Overview**
- In the current generation of CarPlay, iPhone renders only some screens; next-gen CarPlay allows iPhone to render all display surfaces including the instrument cluster
- The vehicle system acts as a "rendering client": it provides display metadata and sensor data; iPhone generates all pixel output
- Communication happens over a high-bandwidth connection (USB or wireless); frames are composited on-device before being transmitted to vehicle displays
- The vehicle system is responsible for audio capture and playback hardware; iPhone handles audio processing and routing

**UI Rendering and Compositing**
- iPhone renders CarPlay UI as layers; the vehicle system receives layers and composites them with its own native content (e.g., backup camera, turn signals)
- Vehicle system controls the Z-order and visibility of layers — native vehicle content can appear on top of CarPlay content or vice versa
- Display dimensions, pixel density, and layout are described in a vehicle configuration provided to iPhone at connection time
- Multiple display surfaces supported: center stack, instrument cluster, heads-up display, rear-seat entertainment

**Configuration and Customization**
- Automakers provide a "vehicle configuration" that describes display surfaces, available hardware capabilities, and feature flags
- Custom features can be enabled per vehicle model — e.g., whether the vehicle supports wireless CarPlay, which display surfaces are available, which native controls (steering wheel buttons, rotary knobs) are mapped
- Feature availability is controlled through the MFi Program; automakers apply for specific capability entitlements

**Enabling Custom Features**
- Vehicle-specific features are declared in a configuration plist submitted through the MFi Program
- Automakers can declare custom app icons, brand colors, and model-specific layouts
- iPhone reads the configuration at connection time and adapts the CarPlay experience accordingly

## APIs & Frameworks

This session is targeted at automakers and vehicle system integrators via the MFi Program — not at third-party iOS app developers. There are no public iOS SDK APIs introduced. Vehicle system integration requires:

- **MFi Program enrollment** — required for all next-generation CarPlay vehicle systems
- **Vehicle configuration schema** — describes display surfaces, hardware capabilities, and custom features
- **CarPlay communication protocol** — proprietary protocol between vehicle system and iPhone; not part of the public iOS SDK
- **Display layer compositing** — vehicle system API for receiving and compositing iPhone-rendered layers with native content

**Related Public Resources**
- CarPlay for developers: `developer.apple.com/carplay`
- MFi Program: `mfi.apple.com`
- Forum: App & System Services

## Code Highlights

No public iOS SDK code samples were presented. The session is architecture and integration guidance for automakers working under the MFi Program.

## Takeaways
- The next generation of CarPlay offloads all display rendering to iPhone — vehicle systems receive pixel layers to composite, rather than running their own rendering pipeline
- Automakers declare capabilities and display surfaces in a vehicle configuration submitted through the MFi Program; iPhone adapts automatically
- Multiple simultaneous display surfaces (instrument cluster, center stack, HUD) are supported; Z-order compositing allows native vehicle content to coexist with CarPlay layers
- If you're an automaker interested in the next-gen CarPlay, start with the MFi Program and the CarPlay for developers resources linked in this session

---
_Source: WWDC24 Session 10111 page (abstract, chapter list, and resource links)._
