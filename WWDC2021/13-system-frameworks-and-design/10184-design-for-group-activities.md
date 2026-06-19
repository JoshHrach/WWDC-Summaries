# Design for Group Activities
**WWDC21 · Session 10184** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10184/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session introduces the design philosophy behind SharePlay and the Group Activities framework, helping developers identify which parts of their app are best suited for shared FaceTime experiences. The speaker frames SharePlay as access to a world-class, low-latency, end-to-end encrypted networking infrastructure—the same one powering FaceTime—now open to third-party apps.

The session emphasizes a three-step process: discover activities in your app worth sharing, enhance them for a group context, and then automate the experience using the Group Activities API so that joining is seamless. Concrete guidance is given on screen sharing restrictions, coordinated media playback, the Messenger broadcast protocol for custom UI, and how to minimize friction for users on live FaceTime calls.

A key design principle is that any user interaction during SharePlay is multitasking—friends are waiting—so every step before reaching the shared content must be questioned and, where possible, eliminated or deferred.

## Key Topics

### Discovering Shareable Activities
- Look for things people love to do together: watching, listening, gaming, learning, co-browsing.
- SharePlay can inspire entirely new app categories designed around group participation rather than single users.

### Screen Sharing
- Screen sharing is available by default while on a FaceTime call; all on-screen content is shared except secure entry fields.
- UIKit API can restrict additional views from being shared during a screen sharing session.
- App audio is shared automatically; DRM-protected content (music, movies) is not shared over screen sharing—that requires coordinated media playback.

### Coordinated Media Playback
- Starts playback on all participants' devices simultaneously and keeps them in sync (rate changes, seeks).
- Media is not streamed device-to-device; each device plays its own local or network copy.
- Smart volume ducking of FaceTime audio is provided automatically.
- iOS 15 supports seamless upgrades from screen sharing to full media playback when a Group Activity begins.

### Messenger Protocol (Custom UI)
- Broadcasts arbitrary data to all participants in near real time.
- End-to-end encrypted, private, same security model as FaceTime.
- Used to drive custom synchronized UI (e.g., shared drawing canvases, collaborative tools).

### Making the Experience Contextual
- When your app is launched during a FaceTime call, the system shows a SharePlay affordance within your app's context.
- Video-content apps should surface a SharePlay button on content detail screens.
- Activity previews (title, subtitle, thumbnail) communicate to all participants what is being shared—treat them like rich links.

### Automation and Reducing Friction
- Apps should auto-launch from the background when a Group Activity begins; this requires Picture in Picture support.
- If foreground interaction is required (e.g., subscription), call the Group Activities API for foreground presentation; a banner appears for participants to tap.
- If the app is not installed, participants are sent to the App Store automatically.
- Minimize account sign-up, purchase flows, and any interaction that makes friends wait.

## APIs & Frameworks

**Group Activities** framework **[NEW]**
- `GroupActivity` protocol — defines a shareable activity with metadata **[NEW]**
- `GroupSession` — represents the active shared session among FaceTime participants **[NEW]**
- `GroupActivityMetadata` — title, subtitle, and thumbnail for the activity preview **[NEW]**
- `Messenger` protocol — broadcasts `Codable` messages to all session participants **[NEW]**
- Coordinated media playback integration — synchronizes `AVPlayer` or custom player across devices **[NEW]**
- Foreground presentation API — requests the app be brought to the foreground for a required interaction **[NEW]**
- Picture in Picture support — enables automatic background launch when a Group Activity starts **[existing, required for auto-launch]**

**UIKit**
- Screen sharing visibility restriction API — hides sensitive views during screen sharing sessions **[NEW]**

## Code Highlights

No code samples were shown in this design-focused session. Implementation details are covered in:
- "Meet Group Activities" (Session 10183)
- "Build custom experiences with Group Activities" (Session 10187)
- "Coordinate media playback in Safari with Group Activities" (Session 10189)

## Takeaways
- Identify activities in your app people love doing together, then design a group-first version of those flows.
- Use coordinated media playback (not screen sharing) for DRM-protected content; use the Messenger protocol for custom synchronized UI.
- Activity preview metadata (title, subtitle, thumbnail) is the first thing participants see—make it descriptive and contextual.
- Treat every interaction step before reaching shared content as a barrier; automate or eliminate it since participants are on a live FaceTime call.

---
_Source: WWDC21 Session 10184 page (abstract, full transcript, and resource links)._
