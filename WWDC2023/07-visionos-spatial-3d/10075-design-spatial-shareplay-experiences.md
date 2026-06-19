# Design Spatial SharePlay Experiences
**WWDC23 · Session 10075** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10075/)

_Platforms:_ visionOS 1

## Overview
This session by Carnaven Chiu and Jay Moon from Apple's Design team covers how to design meaningful shared activities for visionOS apps using SharePlay and Spatial Personas. It explains how physical co-presence conventions translate into spatial computing: shared coordinate systems, seating arrangements via Spatial Persona Templates, UI reduction through in-person nonverbal cues, and when to break from shared context into individual Full Space experiences.

Spatial Personas give users a sense of physical presence with remote collaborators — they can make eye contact, use body language, and share a virtual environment. This demands a new design model that goes far beyond adding a share button to an existing app.

## Key Topics

### Shared Context
Shared context means all participants share a common coordinate system and frame of reference — if someone is to your right, you are to their left. The platform manages coordinate synchronization; the app is responsible for keeping content state (animations, interactions, mode changes) in lockstep across all instances. This replaces much of the explicit state-change notification UI needed on iOS SharePlay.

### Activity Types and Window vs. Full Space
- **Windowed activity**: The shared window acts like a communal device (TV, whiteboard, table). Supports multitasking; participants can move it around their space; good for collaboration. Every window in a SharePlay session is clearly labeled "shared" or "not shared" by the system.
- **Full Space activity**: Fully immersive, like a dedicated room. No access to other apps. Best for games and media where one ideal viewing position exists. Supports up to 5 participants total (host + 4 others).

### Joining and Onboarding
- Add in-app entry points to SharePlay (e.g., a Play button that starts a shared activity).
- The platform allows sharing any window with no app adoption; adopt SharePlay for richer volumetric/interactive experiences.
- Minimize setup friction: avoid requiring accounts for shared activities where possible; show a placeholder window to participants who need to resolve permissions issues.
- When prerequisites are met, reveal the activity exactly where the placeholder was.

### Spatial Persona Templates **[NEW]**
Three seating arrangements the app selects to define participant positioning:
1. **Side-by-Side** — participants sit shoulder to shoulder, good for content that requires a front-facing view (video, presentations).
2. **Surround** — content is placed in the center of a circle of people; good for tabletop-style collaboration. Also used when there is no central content (social hangout).
3. **Conversational** — conversation is front and center; app content is ambient/background; not all participants have a direct view of the content.

### UI and Interaction Design in Shared Context
- In-person nonverbal cues (pointing, body language, facial expressions) replace much of the explicit notification UI needed in iOS SharePlay.
- Be permissive with interactions; avoid complex permissions or turn-taking systems unless inherent to the experience.
- Resolve conflicts with consistent, predictable rules rather than locked-out states.
- Personalized controls (e.g., individual volume, subtitles preferences) are appropriate for comfort and accessibility, as long as shared content playback remains synchronized.

### Collaborative Work in Shared Context
- All participants see the same shared viewport simultaneously.
- Allow different editing modes per person (e.g., different annotation colors).
- Allow individuals to open personal windows for focused editing while the shared context remains visible to all.

### Leaving Shared Context for Full Space
Appropriate cases for breaking from shared context:
- Content with a single ideal viewing position (spatial captures, panoramas, 180/360-degree video) — hide Spatial Personas and synchronize playback position.
- Individual safety / needing to return to passthrough (Digital Crown press) — show a window representation of the shared context to the person who stepped out; make it easy to rejoin.
- Design so participants who temporarily leave never feel lost about where the group is.

### Spatial Audio in SharePlay
- When all participants are in the same environment, audio is spatialized from a shared coordinate: a music player window sounds like a speaker positioned in the shared space.
- Sound can emanate from the app window or as part of an immersive soundstage everyone navigates together.

## APIs & Frameworks

### GroupActivities / SharePlay **[NEW/expanded for visionOS]**
- `GroupActivity` protocol — defines a shared activity **[NEW visionOS context]**
- `GroupSession` — manages the active SharePlay session
- `GroupSessionMessenger` — sends state messages between participants
- `SpatialPersonaTemplate` — defines the seating arrangement for a shared activity **[NEW]**
  - `.sideBySide` — shoulder-to-shoulder arrangement **[NEW]**
  - `.surround` — circular/tabletop arrangement **[NEW]**
  - `.conversational` — conversation-focused arrangement **[NEW]**
- Shared window + immersive space pairing — up to one window and one Full Space at a time per shared activity **[NEW]**
- Placeholder window — system-provided or app-provided window for participants who haven't met entry requirements **[NEW]**
- Participant limit — up to 5 Spatial Personas (host + 4 others) **[NEW]**

### Spatial Audio
- Positional audio from shared window origin — sound localizes from the app's world-locked position
- Shared soundstage in Full Space immersive experiences

### Related Frameworks
- `RealityKit` — for volumetric content in shared Full Space experiences
- `GroupActivities` — base SharePlay framework
- Digital Crown passthrough — system-level return to passthrough, app should handle participant temporarily leaving

## Code Highlights
No code samples were shown. This is a design-focused session; see "Build spatial SharePlay experiences" (10087) and "Add SharePlay to your app" (10239) for implementation.

## Takeaways
- Choose the right Spatial Persona Template (Side-by-Side, Surround, or Conversational) based on the nature of the shared content and the interaction model.
- Reduce notification-style UI: in-person cues from Spatial Personas naturally communicate who did what, so explicit state-change banners are often unnecessary.
- Be permissive with shared interactions and use consistent conflict resolution rather than locking participants out.
- Know when to break from shared context — spatial captures and 360° content warrant individual Full Space views, but always keep participants informed of the group's position.

---
_Source: WWDC23 Session 10075 page (abstract, chapter summaries, code samples, and resource links)._
