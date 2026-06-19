# Customize Spatial Persona Templates in SharePlay
**WWDC24 · Session 10201** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10201/)

_Platforms:_ visionOS 2

## Overview
visionOS 2 introduces custom spatial Persona templates — a new GroupActivities API that gives SharePlay apps full control over where FaceTime spatial Personas are positioned relative to shared content. Previously, apps had to choose from a handful of system-provided templates (side-by-side, conversational, surround). Now, developers can define arbitrary seat layouts with per-seat roles, seat direction, and spatial offsets measured in meters.

The session builds "Guess Together," a team-based word guessing game, as a worked example, implementing a multi-stage spatial template design: a side-by-side default for category selection, a custom role-based layout for team selection (with reserved seats for each team), and a game-stage template with a designated active player seat and directional constraints. The session also covers the new simulated FaceTime call support in Xcode 16's visionOS Simulator and group immersive space opt-in.

## Key Topics
- **`SpatialTemplate` protocol** — define a type with an `elements: [any SpatialTemplateElement]` array of seats; each seat has a position offset from the shared app.
- **`SpatialTemplateRole`** — a `String` raw-value enum protocol; attach roles to seats to reserve them; participants assign/resign roles on the `SystemCoordinator`.
- **Seat direction** — `SpatialTemplateSeatElement` accepts a `direction` parameter; `.lookingAt(position)`, `.alignedWith(appAxis:)`, and `.rotatedBy(.degrees(_:))` control which way the persona faces.
- **Group immersive space** — opt in via `systemCoordinator.configuration.supportsGroupImmersiveSpace = true`; the immersive space origin matches the spatial template origin (both in meters), enabling podium placement at seat positions.
- **Simulated FaceTime in Xcode 16** — Features > FaceTime menu in the visionOS Simulator starts a simulated call with configurable participant counts; critical for iterating on custom templates without a real device.

## APIs & Frameworks

**GroupActivities**
- `SpatialTemplate` protocol — **[NEW]** `var elements: [any SpatialTemplateElement]` requirement
- `SpatialTemplateElement` — **[NEW]** protocol; factory method `.seat(position:)` and `.seat(position:direction:role:)`
- `SpatialTemplateSeatElement` — **[NEW]** concrete seat type; `init(position:direction:role:)`
- `SpatialTemplateElementPosition` — **[NEW]** value type representing a position relative to the app
  - `.app` — the origin at the shared app's center
  - `.app.offsetBy(x:z:)` — offset in meters on the X (left/right) and Z (forward/back) axes
  - `.offsetBy(x:z:)` — chain offset on an existing position
- `SpatialTemplateElementDirection` — **[NEW]** enum-like type controlling seat facing direction
  - `.lookingAt(_ position: SpatialTemplateElementPosition)` — face toward a specific position
  - `.lookingAt(_ seat: SpatialTemplateSeatElement)` — face toward another seat
  - `.alignedWith(appAxis:)` — align with `.x` or `.z` axis of the app
  - `.rotatedBy(_ angle: Angle)` — rotate an existing direction by degrees or radians
- `SpatialTemplateRole` protocol — **[NEW]** conform an enum with `String` raw value to define roles
- `SystemCoordinator` — existing type; new API in visionOS 2:
  - **[NEW]** `assignRole(_:)` — assign the local participant the given `SpatialTemplateRole`
  - **[NEW]** `resignRole()` — release the local participant's current role; returns them to an unroled seat
  - `configuration.spatialTemplatePreference` — existing property
    - **[NEW]** `.custom(_ template: some SpatialTemplate)` — set a custom template (new overload alongside existing `.sideBySide`, `.conversational`, `.surround`)
  - **[NEW]** `configuration.supportsGroupImmersiveSpace` — set to `true` to opt into a shared group immersive space; makes all participants' immersive spaces share an origin

**Xcode 16 / visionOS Simulator**
- Features > FaceTime > [User + N Spatial Participants] — **[NEW]** simulated FaceTime call support in the visionOS Simulator

## Code Highlights
Define a custom template with team-based reserved seats:

```swift
struct TeamSelectionTemplate: SpatialTemplate {
    enum Role: String, SpatialTemplateRole { case blueTeam, redTeam }
    let elements: [any SpatialTemplateElement] = [
        .seat(position: .app.offsetBy(x: -2.5, z: 3.5), role: Role.blueTeam),
        .seat(position: .app.offsetBy(x: 0,    z: 4)),
        .seat(position: .app.offsetBy(x: 2.5,  z: 3.5), role: Role.redTeam),
    ]
}
```

Activate the custom template and assign a role:

```swift
systemCoordinator.configuration.spatialTemplatePreference = .custom(TeamSelectionTemplate())
systemCoordinator.assignRole(TeamSelectionTemplate.Role.blueTeam)
```

Seat with directional constraint:

```swift
let playerSeat = SpatialTemplateSeatElement(
    position: .app.offsetBy(x: -2, z: 3),
    direction: .lookingAt(activeTeamCenterPosition),
    role: Role.player
)
```

## Takeaways
- Design custom templates for each distinct stage of your activity, not a single monolithic layout — Guess Together uses three different templates across its stages.
- Always include enough seats for the maximum number of FaceTime participants (currently six); FaceTime places overflow participants in a fallback system template if seats run out.
- Use roles only for seats that must be reserved (active player, dealer); leave spectator/audience seats roleless so FaceTime can fill them immediately without a role assignment round-trip.
- Enable the Xcode 16 simulated FaceTime call feature early in development — it eliminates the need for real hardware during the spatial template iteration loop.

---
_Source: WWDC24 Session 10201 page (abstract, chapter summaries, code samples, and resource links)._
