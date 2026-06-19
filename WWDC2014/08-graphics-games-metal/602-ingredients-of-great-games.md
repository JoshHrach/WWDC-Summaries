# Ingredients of Great Games
**WWDC14 · Session 602** · [Watch](https://developer.apple.com/videos/play/wwdc2014/602/)

_Platforms:_ iOS 8, OS X Yosemite 10.10

## Overview
This session opens the WWDC 2014 game development track with a survey of the key ingredients that distinguish top-tier iOS games from the rest. Rather than focusing on a single technology, it covers game design philosophy, player engagement mechanics, technical best practices (background transfers, graphics optimization), and App Store strategy—all illustrated with Apple Design Award winners such as Cut the Rope, Monument Valley, Threes, Leo's Fortune, and Ridiculous Fishing.

The session is structured as a ten-point list (plus an eleventh: "go to 11") touching everything from first-run experience to localization to leveraging the A7 chip and iOS 8 APIs. The overarching message is that technical excellence and platform integration alone are not sufficient; games must also nail onboarding, core loop design, touch interaction, and continuous content engagement to succeed in a marketplace with 75 billion downloads and 130 million monthly Game Center users.

A closing section from Apple's Game Technologies Evangelist doubles as a call to action: target the state of the art (iOS 8, A7, Metal, SpriteKit, SceneKit, Game Controllers) rather than retrofitting older platform assumptions.

## Key Topics

### 1. Remove Friction
Minimize load time, defer non-critical asset downloads, eliminate forced registration screens, avoid premature configuration prompts, and delay feedback/review requests until the player has experienced the game.

### 2. Be a Good Teacher
Design contextual, in-game onboarding that introduces mechanics one at a time (illustrated by Cut the Rope's coaching tip sequence). Never leave critical interactions undiscovered.

### 3. Tune Your Core Loop
Clearly define the player action → reward → expansion cycle. Great games often layer multiple overlapping core loops (combat + puzzle, etc.) and ensure each iteration is meaningful.

### 4. Design for Touch
Embrace direct manipulation: reach through the glass and interact with game elements directly via taps, swipes, gestures, and path drawing. Provide instantaneous visual, audio, and haptic feedback. Avoid virtual D-pads and virtual button overlays.

### 5. Foster Engagement
Layer progression paths with multiple goals and risk/reward tradeoffs. Provide value for both paying and non-paying players. Incentivize "one more try," "see what's next," and "see how it ends." Plan post-launch content refreshes (seasonal, regional) before shipping.

### 6. Use Background Transfers
Use `NSURLSession` (introduced iOS 7) to download levels, textures, audio, and localization assets out-of-process. Keep the initial bundle under the cellular download limit by bundling only first-session content; download the rest while the player plays.

### 7. Optimize Graphics Performance
Profile with the GPU Performance Graphs in Xcode, the GPU Frame Debugger, and the OpenGL ES Analyzer instrument. Identify and eliminate bottlenecks such as excessive draw calls with low triangle counts (bind/draw loops). Consider SpriteKit and SceneKit to offload rendering optimization. Use Metal for engines that need high draw-call throughput and direct GPU access.

### 8. Create a Great Gameplay Video / App Preview
iOS 8 and OS X Yosemite introduce App Previews on the App Store (up to 30 seconds, H.264/MPEG-4). Capture gameplay directly from an iOS device to a Yosemite Mac (device appears as a camera source). Use it to authentically convey gameplay, not as an advertisement.

### 9. Localize
Localize App Store metadata (name, description, keywords, screenshots) and in-app content. Recommended languages: EFIGS, Japanese, Korean, Traditional Chinese, Simplified Chinese, Brazilian Portuguese, Russian, Turkish, Arabic. Separate text from artwork to avoid duplicating large assets per locale.

### 10. Target the State of the Art
Set Base SDK to current, deployment target to iOS 7 minimum; support iOS 7 and iOS 8. Target A7 capabilities. Integrate Game Center, SpriteKit, SceneKit (new on iOS), Metal (new), Game Controllers, and OpenGL ES 3.0. Use Xcode 6, Swift, and Playgrounds.

## APIs & Frameworks

**Networking / Background Transfers**
- `NSURLSession` — out-of-process background uploads and downloads **[iOS 7+]**
  - `NSURLSessionConfiguration.backgroundSessionConfiguration`
  - `NSURLSessionDownloadTask`
  - `application(_:handleEventsForBackgroundURLSession:completionHandler:)` (app delegate hook)

**Graphics & Games**
- `Metal` **[NEW]** — low-level, highly optimized GPU API for high draw-call games and engines
- `SpriteKit` — 2D game framework; handles OpenGL ES rendering automatically
- `SceneKit` — 3D game framework; now available on iOS 8 **[NEW on iOS]**
- `OpenGL ES 3.0` — most advanced OpenGL ES available at WWDC 2014
- `GLKit` — OpenGL ES helper framework

**Game Input**
- `GameController` framework — supports physical game controllers; opens game to multiple input forms

**Social / Engagement**
- `GameKit` / `Game Center` — leaderboards, achievements, multiplayer matchmaking (130 M monthly active users)

**Developer Tools (referenced)**
- Xcode 6 GPU Performance Graphs
- GPU Frame Debugger (Xcode)
- OpenGL ES Analyzer Instrument
- Time Profiler Instrument
- Metal Debugger
- SpriteKit / SceneKit scene editors in Xcode

**App Store**
- App Previews (gameplay video) **[NEW in iOS 8 / OS X Yosemite]** — H.264 MPEG-4, up to 30 s, uploaded via iTunes Connect
- iTunes Connect Retention Analytics **[NEW, coming fall 2014]** — daily retention by day, region, platform, source
- `UIDevice.identifierForVendor` — use as server-side player identifier to avoid forced registration

**Swift / Xcode**
- Swift (introduced at WWDC 2014)
- Xcode 6 Playgrounds
- Grand Central Dispatch (`DispatchQueue`) — move asset loading off main thread

## Code Highlights

No code samples were presented in this session. It is a design and strategy talk.

Patterns recommended:
- Defer asset loading using GCD: load only enough assets to reach first interactive screen, then load the rest asynchronously.
- Use `NSURLSession` background download tasks for large asset packs (levels, textures, audio); handle `application:handleEventsForBackgroundURLSession:completionHandler:` to move downloaded content to its permanent location.
- Use `UIDevice.current.identifierForVendor` instead of requiring user account registration for server-backed player identification.

## Takeaways

- The first-run experience, core loop design, and touch interaction model are as important to a game's success as technical implementation quality.
- Background `NSURLSession` transfers allow shipping a small initial bundle (under cellular limit) while silently delivering the remaining content during early gameplay.
- Apple's high-level game frameworks (SpriteKit, SceneKit) handle rendering optimization automatically; use them unless you need the explicit GPU control that Metal provides.
- Target iOS 8 and the A7 as your leading-edge capability; maintain iOS 7 as your deployment minimum and handle API differences with runtime checks.

---
_Source: WWDC14 Session 602 page (abstract, chapter summaries, code samples, and resource links)._
