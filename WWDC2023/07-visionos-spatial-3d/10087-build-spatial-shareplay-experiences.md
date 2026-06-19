# Build Spatial SharePlay Experiences
**WWDC23 · Session 10087** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10087/)

_Platforms:_ visionOS 1, iOS 17

## Overview
This session explains how to build SharePlay experiences that feel truly co-present on visionOS. On Apple Vision Pro, FaceTime places Spatial Personas physically in the room rather than in a flat window, creating shared context—a coordinate system where all participants see each other in the same relative positions. The session covers how apps must maintain both spatial consistency (everyone in the right place) and visual consistency (everyone seeing the same content) to make SharePlay feel natural.

The talk is split into two parts. The first covers windowed SharePlay: the new `SystemCoordinator` API, spatial template preferences (Side-by-Side, Conversational, Surround), multi-scene app handling via scene association, and launching SharePlay from the system Share menu. The second part covers immersive SharePlay: enabling group immersive spaces via `supportsGroupImmersiveSpace`, using `GeometryReader3D.immersiveSpaceDisplacement` to place content relative to the local user, and synchronizing immersion style across participants via `groupImmersionStyle`.

## Key Topics

### Shared Context and Consistency
Spatial Personas have a shared coordinate system; the system uses templates to place participants and windows in consistent relative positions. Apps are responsible for visual consistency—keeping content state in sync for all participants using `GroupSessionMessenger`.

### SystemCoordinator
The new `SystemCoordinator` object (retrieved from a `GroupSession`) provides:
- `localParticipantState.isSpatial` — detect if the local participant is running on visionOS.
- `localParticipantStates` — async sequence for observing changes.
- `configuration.spatialTemplatePreference` — request Side-by-Side, Conversational, or Surround template.
- `configuration.supportsGroupImmersiveSpace` — opt into group immersive space for immersive experiences.
- `groupImmersionStyle` — async sequence of the immersion style other participants are using.

### Spatial Templates
- `.sideBySide` — arc of participants facing the shared app (default for vertical windows).
- `.conversational` — half-circle of participants, app in front (music/audio experiences).
- `.surround` — participants circle the app placed at center (default for volumetric windows; only available with volumetric scenes).
- `contentExtent(_:)` modifier — hint for dynamic distance calculation when no window size is available.

### Scene Association (Multi-Scene Apps)
The system evaluates scene activation conditions against the group activity identifier to determine which window scene is "shared." For document-based apps where multiple scenes are interchangeable, `GroupActivityMetadata.sceneAssociationBehavior` can supply a content-specific identifier via `.content("id")`. `GroupSession.sceneSessionIdentifier` reveals the associated scene. Use `.none` behavior only for immersive-only apps or apps where each participant intentionally sees different content.

### Launching SharePlay from the Share Menu
Every visionOS window has a system Share banner. Apps surface relevant group activities by setting `UIActivityItemsConfiguration` (with `NSItemProvider.registerGroupActivity(_:)` and `LPLinkMetadata`) on a view controller in the responder chain. Works alongside AirDrop-initiated SharePlay in iOS 17.

### Group Immersive Space
- Enable with `configuration.supportsGroupImmersiveSpace = true`.
- The system moves the space origin to a template-defined shared location, establishing a shared coordinate system.
- Spatial Personas appear inside the group immersive space.
- Place shared objects at the same offset from the origin for all participants.
- Use `GeometryReader3D.immersiveSpaceDisplacement(in:)` (returns `Pose3D`) to find the local user's position relative to origin; invert it to place per-user UI.
- Monitor `systemCoordinator.groupImmersionStyle` to open/dismiss immersive space in sync with other participants. Digital Crown dismissal does not trigger this stream; a system SharePlay banner lets the user rejoin.

## APIs & Frameworks

**GroupActivities** (visionOS 1 additions unless noted)
- `GroupSession` — existing API
- `GroupSession.systemCoordinator` async **[NEW]** — access the system coordinator
- `GroupSession.sceneSessionIdentifier` **[NEW]** — associated scene session ID
- `SystemCoordinator` **[NEW]** — new class managing spatial session configuration
- `SystemCoordinator.Configuration` **[NEW]** — mutable configuration struct
- `SystemCoordinator.Configuration.spatialTemplatePreference` **[NEW]** — template selection
- `SystemCoordinator.Configuration.supportsGroupImmersiveSpace` **[NEW]** — enable group immersive space
- `SystemCoordinator.localParticipantState` **[NEW]** — current local state
- `SystemCoordinator.localParticipantStates` **[NEW]** — async sequence of state updates
- `SystemCoordinator.LocalParticipantState.isSpatial` **[NEW]** — true on visionOS
- `SystemCoordinator.groupImmersionStyle` **[NEW]** — async sequence of group immersion style
- `SpatialTemplatePreference` **[NEW]** — enum: `.sideBySide`, `.conversational`, `.none`
- `SpatialTemplatePreference.sideBySide.contentExtent(_:)` **[NEW]** — distance hint modifier
- `GroupActivityMetadata.sceneAssociationBehavior` **[NEW]** — `.default`, `.content(_:)`, `.none`
- `GroupActivity` protocol — existing; `sessions()` async sequence
- `GroupSessionMessenger` — existing; state sync between participants

**SwiftUI (visionOS)**
- `ImmersiveSpace` — immersive scene type
- `GeometryReader3D` — 3D geometry reader
- `GeometryProxy3D.immersiveSpaceDisplacement(in:)` **[NEW]** — returns `Pose3D` of space origin offset
- `Pose3D.inverse` — invert pose for local user relative positioning
- `View.handlesExternalEvents(preferring:allowing:)` — scene activation conditions in SwiftUI

**UIKit**
- `UIScene.activationConditions` — `canActivateForTargetContentIdentifierPredicate`, `prefersToActivateForTargetContentIdentifierPredicate`
- `UIActivityItemsConfiguration` — activity items config for Share menu
- `UIActivityItemsConfiguration.metadataProvider` — provide `LPLinkMetadata` for `.linkPresentationMetadata` key
- `NSItemProvider.registerGroupActivity(_:)` **[NEW]** — register group activity on item provider (iOS 17)
- `LPLinkMetadata` — link presentation metadata (title, imageProvider)

## Code Highlights

Observing spatial participant state:
```swift
for await localParticipantState in systemCoordinator.localParticipantStates {
    if localParticipantState.isSpatial {
        // Start syncing scroll position
    } else {
        // Stop syncing scroll position
    }
}
```

Configuring group immersive space with side-by-side template:
```swift
var configuration = SystemCoordinator.Configuration()
configuration.supportsGroupImmersiveSpace = true
configuration.spatialTemplatePreference = .sideBySide.contentExtent(200)
systemCoordinator.configuration = configuration
```

Syncing immersion style across participants:
```swift
for await immersionStyle in systemCoordinator.groupImmersionStyle {
    if let immersionStyle {
        // Open immersive space with matching style
    } else {
        // Dismiss immersive space
    }
}
```

## Takeaways
- `SystemCoordinator` is the central new API for spatial SharePlay; it provides spatial awareness, template preferences, and group immersive space coordination.
- Apps must actively maintain visual consistency (content state sync via `GroupSessionMessenger`) — the system only handles spatial placement.
- Multi-scene apps must adopt scene association to ensure the correct window appears in templates and shows the shared indicator.
- Group immersive spaces require `supportsGroupImmersiveSpace = true` and listening to `groupImmersionStyle` to keep all participants in sync.

---
_Source: WWDC23 Session 10087 page (abstract, chapter summaries, code samples, and resource links)._
