# Say Hello to the Next Generation of CarPlay Design System
**WWDC24 · Session 10112** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10112/)

_Platforms:_ iOS 18 (CarPlay)

## Overview
This design-focused session introduces the deeply customizable design system powering the next generation of CarPlay — the experience that extends CarPlay to all driver-facing displays including the instrument cluster. The session is primarily aimed at automakers and system designers rather than third-party app developers. It explains how each automaker, in partnership with Apple, can create a co-branded CarPlay experience that is visually unique to their brand while maintaining the consistency and safety requirements of an automotive interface.

The system is built to adapt to any vehicle drivetrain, screen configuration (size, shape, count), and feature set, while still expressing the automaker's visual design philosophy. It builds on the technical architecture described in the companion session "Meet the Next Generation of CarPlay Architecture" (Session 10111).

## Key Topics

**Typography System**
SF variable fonts are the typographic foundation. Weight, width (extended/condensed), and corner softness all exist on continuous scales rather than discrete named weights — enabling far more stylistic range than traditional font families. The entire design system is built with the same continuous-scale logic.

**Gauge Customization**
The instrument cluster gauge system is the centerpiece of the presentation. Every visual property is independently adjustable: stroke width, corner softness, arc start/end points (including full-circle option), tick mark count/length/style/position (multiple sets supported), needle vs. arc style, color (flat or gradient), and typography position. These parameters combine to produce gauges ranging from minimal and elegant to bold and technical.

**Gauge Functions**
Regardless of visual style, all gauge styles support a full set of required functions: speedometer always paired with fuel/charge gauge; cruise control (target speed dot + line for adaptive cruise); speed limiter (line in distinct color); road signage and alternate speed units; low/critical fuel/charge states (orange/red thresholds configurable per automaker); tachometer with redline and recommended gear shift arrows; EV power meter with boost and regenerative braking; hybrid gauges showing both combustion and electric simultaneously; secondary gauges (e.g., engine coolant temperature); transmission state; and brand logo in circular gauge center.

**Layout System**
Modular, resizable gauge components assembled freely into instrument cluster layouts. Multiple configurations possible: compact linear gauges, full circular center gauge with minimal linear companions, multi-gauge information-rich arrangements. Content positioning and orientation are configurable. Custom wallpapers span all displays. Layouts flex to any screen size and shape.

**Dynamic Content**
Every layout reserves a space for dynamic content — a rotating set of content panels the driver flips through via steering wheel controls. Permanent content area or stackable behind non-critical gauge elements. Supports notifications (text/symbol, interactive/dismissible, rich multi-state overlays with vehicle imagery). Vehicle content includes: maps, now playing, trip computer, tire pressure, ADAS display, vehicle image (with color/trim customization). Center and passenger displays support broader content.

**Automaker Integration**
Automaker Settings section folds vehicle features into CarPlay settings. Automaker Apps surface brand-specific experiences and interfaces, updateable like a regular app. Native climate app shown on center screen. Terrestrial radio with hard key integration.

## APIs & Frameworks
This session is design-focused; there is no developer SDK API coverage. Relevant program and resources:

- **Next Generation CarPlay design system** — **[NEW]** co-branded instrument cluster and multi-display experience
- **SF Variable Fonts** — weight, width, corner softness on continuous scales (foundation of the typography system)
- **CarPlay for developers** — developer.apple.com/carplay
- **MFi Program** — enrollment required to participate in next generation CarPlay development
- **Companion session**: "Meet the Next Generation of CarPlay Architecture" (Session 10111) covers the technical API layer

## Code Highlights
No code samples are included — this is a design system overview session for automakers.

## Takeaways
- If you are an automaker or system developer, the next generation of CarPlay design system provides deep gauge, layout, color, typography, and wallpaper customization — enroll in the MFi program to participate.
- The gauge design parameters (width, softness, arc, ticks, color) compose multiplicatively, so the creative range is very large — almost any gauge aesthetic is achievable.
- Dynamic content is a mandatory architectural component of every layout — plan how your vehicle's content (ADAS, tire pressure, etc.) will integrate with iOS content (maps, media).
- Pair this session with "Meet the Next Generation of CarPlay Architecture" (10111) for the full picture.

---
_Source: WWDC24 Session 10112 page (abstract, chapter summaries, and resource links)._
