# Optimize CarPlay for Vehicle Systems
**WWDC23 · Session 10150** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10150/)

_Platforms:_ iOS 17, CarPlay (vehicle system integration)

_Audience:_ Automakers and vehicle system developers (not iOS app developers)

## Overview
This session is a technical briefing for vehicle system integrators building CarPlay head units. It covers five areas: visual integration (view areas, safe areas, corner masks, status bar placement, appearance modes, UI focus transfer), connectivity infrastructure (out-of-band pairing, car key pairing, simplified Wi-Fi connection flow, interference management), audio (AirPlay enhanced audio buffering, multi-stream mixing, Enhanced Siri), video encoding (HEVC support alongside H.264), and EV routing. The session also frames all these features as required preparation for the next generation of CarPlay.

No iOS SDK code is involved; all integration is via the CarPlay specification, iAP2 protocol messages, and the CarPlay communication plugin.

## Key Topics

### Visual Integration

**View Area and Safe Area:**
- The view area defines the bounding rectangle where CarPlay draws its UI; give it as much screen space as possible — CarPlay auto-insets when shown alongside built-in widgets
- For non-rectangular displays: view area = smallest enclosing rectangle; safe area = largest inscribed rectangle; CarPlay draws interactive content inside the safe area only
- Enable the safe-area flag to let CarPlay draw its background to display edges (creates immersive look); only available for the main display

**Status Bar Placement:**
- Automatically positioned (vertical on driver side or horizontal along bottom) based on display resolution/aspect ratio
- Can be overridden with a view area flag to align with built-in system controls

**Corner Clipping Masks:**
- CarPlay rounded corners expose black pixels behind them when CarPlay appears in a windowed configuration
- iPhone provides a per-corner blending mask with alpha channel; vehicle applies it to blend CarPlay corners seamlessly with system wallpaper
- Cannot be used simultaneously with "draw outside safe area"

**Dynamic Screen Resizing:**
- Support dynamic view area changes so the display can switch between full-screen CarPlay and layouts that include built-in system widgets without relaunching

**UI Focus Transfer:**
- For knob/touchpad-based vehicles showing CarPlay in a windowed layout
- CarPlay protocol arbitrates focus: only one side shows a focus highlight at a time
- Driver nudges knob → system dismisses focus and sends heading/position → CarPlay presents focus on the most intuitive element; reverse on return

**Appearance Mode:**
- CarPlay supports light and dark themes; synchronize with the vehicle's built-in UI appearance mode
- Appearance can be driven by vehicle state, user settings, or time of day (e.g., auto dark at night)
- Separate appearance control for the map UI — independently configurable per display (main display vs. instrument cluster)

**Multi-Display Content:**
- Main display: CarPlay video stream + iAP2 metadata (route guidance, phone call, now-playing)
- Navigation widget (main display): CarPlay navigation UI stream showing ETA, speed limit, compass
- Instrument cluster: CarPlay navigation UI stream + map zoom support + iAP2 metadata
- Head-up display: iAP2 metadata for turn-by-turn guidance in the driver's line of sight

### Connectivity

**Out-of-Band Pairing:**
- Required for all wireless CarPlay systems
- Driver plugs in iPhone via USB once to pair; subsequent connections are wireless
- iOS 17 adds pairing over a car key connection (digital car key + CarPlay pairing unified into one flow)

**Simplified Connection Flow (New):**
- Uses the existing iAP2 connection to exchange IP address and port information instead of Bonjour
- Simpler, faster, and adds WPA3-only network support
- Compatible with iOS 14 and newer; maintain Bonjour flow for earlier iOS versions

**Instant CarPlay (Car Key):**
- When driver approaches with a digital car key, iAP2 maps the car key pairing to a CarPlay device
- Vehicle pre-warms its CarPlay stack; when car key connection is established, CarPlay starts instantly

**Robust Wireless Connection:**
- Detect interferers on the Wi-Fi channel; use channel switch announcements to steer away
- Use BSS transition management to move iPhone to a less congested access point (if multiple radios available)
- Prefer the 5 GHz band
- Suppress short link-layer disconnects on CarPlay TCP sockets — do not close sockets solely due to a data link layer disconnect

### Audio

**AirPlay Enhanced Audio Buffering (Main Buffered Audio):**
- Audio apps adopting AirPlay enhanced buffering stream audio faster than real-time into a ≤2-minute buffer in the CarPlay communication plugin
- Benefits: improved responsiveness, continued playback through brief wireless disconnections
- Vehicle system adds a new "main buffered audio" stream alongside the existing main audio stream

**Audio Stream Mixing:**
- Vehicles must support simultaneous mixing of: main audio, main buffered audio, alternate audio, and auxiliary audio
- Matches the audio mixing behavior users expect from iPhone

**Enhanced Siri:**
- Leverages vehicle microphone to detect "Hey Siri" (or "Siri") voice activation
- Audio apps/radio are mixed into background during Siri audio (no interruption)
- Push-to-talk button instantly activates Siri without a listening delay

### Video Encoding

**HEVC Support (New):**
- CarPlay UI streams now support HEVC (H.265) encoding in addition to H.264
- HEVC is more efficient and enables support for higher-resolution displays
- Continue to support H.264 for backward compatibility with older iPhones

### EV Routing

- Apple Maps provides EV-aware route planning with charging stop suggestions
- iOS 17 additions: user-preferred charging network selection, real-time charging station availability on the map
- Vehicle systems must provide: EV characteristics (battery capacity, charge rate, connector type) and real-time state of charge via iAP2 messages
- Automaker apps should support SiriKit intents for EV routing both inside and outside the vehicle

### Next Generation of CarPlay

All connectivity, audio, and video encoding features covered in this session are **required** for next-generation CarPlay integration. The next generation provides a fully integrated iPhone experience utilizing the vehicle's full instrument cluster and additional displays, building directly on the infrastructure described here.

## APIs & Frameworks

- **CarPlay specification** – vehicle-side integration protocol (not iOS SDK)
- **iAP2 protocol** – Apple Accessory Protocol 2; used for metadata (route guidance, now-playing, phone call), video stream negotiation, state of charge messages, car key pairing, and simplified connection flow IP exchange
- **CarPlay communication plugin** – software component on the vehicle side; handles audio buffering, encoding/decoding
- View area / safe area flags – CarPlay specification: defines display bounds, interactive zones, draw-outside-safe-area flag, status bar placement override
- Corner clipping masks **[NEW guidance]** – per-corner alpha masks provided by iPhone; vehicle applies to CarPlay window corners
- Dynamic screen resizing – view area changes without session restart
- UI focus transfer **[NEW API]** – CarPlay protocol focus arbitration messages; heading + position data for initial focus placement
- Appearance mode / map appearance mode – iAP2 CarPlay appearance control; per-display configuration
- Navigation UI stream – CarPlay navigation video stream for instrument cluster and map widget
- Out-of-band pairing over USB – required for all wireless CarPlay systems
- Car key pairing for CarPlay **[NEW in iOS 17]** – unified car key + CarPlay pairing flow
- Simplified connection flow **[NEW]** – iAP2-based IP/port exchange replaces Bonjour; WPA3 support; iOS 14+
- Instant CarPlay via car key **[NEW]** – pre-warm CarPlay stack on car key proximity event
- BSS transition management – steer iPhone to alternate access point during interference
- Channel switch announcements – direct iPhone away from interfered Wi-Fi channel
- AirPlay enhanced audio buffering / main buffered audio **[NEW]** – up to 2-minute audio buffer; faster-than-realtime streaming
- HEVC encoding support **[NEW]** – H.265 for all CarPlay UI streams; more efficient; enables higher-resolution displays
- EV routing iAP2 messages – state of charge, battery characteristics; SiriKit intents for EV routing in automaker apps
- `SiriKit` intents – used in automaker iOS apps for EV routing outside the vehicle

## Code Highlights

This session targets vehicle system integrators, not iOS app developers. No iOS SDK code is presented. Integration is through:
- The CarPlay specification document (MFi program)
- iAP2 protocol message definitions
- The CarPlay communication plugin SDK

For iOS app integration, refer to "Get more mileage out of your app with CarPlay" (WWDC22).

## Takeaways
- Every feature in this session — simplified connection flow, HEVC encoding, enhanced audio buffering, car key pairing — is required for next-generation CarPlay; implementing them now future-proofs vehicle systems
- Corner clipping masks eliminate the visual artifact (black triangles behind rounded corners) that appears when CarPlay is displayed in a windowed configuration alongside built-in vehicle UI
- Enhanced audio buffering (main buffered audio) is a drop-in improvement for audio apps already using AirPlay enhanced buffering — the vehicle system simply needs to add support for the new stream
- EV routing requires both static vehicle metadata (battery, connector type) and a real-time iAP2 state-of-charge interface; without both, Apple Maps cannot tailor charging stop recommendations

---
_Source: WWDC23 Session 10150 page (abstract, chapter summaries, and transcript)._
