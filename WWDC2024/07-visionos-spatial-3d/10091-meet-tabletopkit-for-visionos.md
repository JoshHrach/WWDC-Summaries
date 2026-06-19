# Meet TabletopKit for visionOS
**WWDC24 · Session 10091** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10091/)

_Platforms:_ visionOS 2

## Overview
TabletopKit is a new framework for visionOS that makes it straightforward to build spatial board games and tabletop experiences. It provides high-level abstractions for play surfaces, seats, equipment (pieces and tiles), and interaction handling — plus built-in multiplayer support via SharePlay and spatial Personas. The session walks through building a complete dice-rolling conveyor-belt board game from scratch.

## Key Topics

**Setting Up the Play Surface**
- `Tabletop` — the central play surface; initialized with a shape (`.rectangular(entity:)`) and a RealityKit `Entity` loaded from a Reality Composer Pro bundle
- `TableVisualState.Pose2D` — 2D position/rotation for placing objects on the table surface
- Seats are positioned as an array of `Pose2D` values specifying position (x, z) and rotation (facing the center); TabletopKit maps these to player positions in 3D

**Defining Equipment**
- `Equipment` protocol — conforming types represent game pieces (tiles, dice, pawns); each has an `id`, an `Entity`, and an `initialState` of type `BaseEquipmentState`
- `EntityEquipment` protocol — variant for equipment backed by RealityKit entities
- `BaseEquipmentState` — holds initial position (`Point2D` / `Pose2D`), category, size, and whether the piece is interactable
- Equipment is designed to be value-type and declarative — describe what pieces exist and where; TabletopKit manages their 3D presence

**Implementing Rules (Interactions)**
- `TabletopInteraction` protocol — implement to receive callbacks when players grab, move, or release equipment
- `update(context:value:)` — called on every interaction update; `TabletopInteractionValue.phase` lets you react to `.started`, `.changed`, `.ended`
- `value.proposedDestination` — where the player intends to drop the piece; validate and either accept or redirect
- Interactions are authoritative — the game logic decides what moves are valid
- The `RealityView` uses the `.tabletopGame(_:)` modifier to bind the TabletopKit game to the scene

**Integrating RealityKit Effects**
- TabletopKit sits on top of RealityKit — all visual effects use standard RealityKit APIs
- `AudioLibraryComponent` — attach to an entity to play spatialized audio effects (e.g., die roll sound)
- Retrieve the component from the entity and play a named audio file during interaction callbacks

**Multiplayer with SharePlay**
- SharePlay + spatial Personas integration requires only a few extra lines of code
- `GroupActivities` framework: define a `GroupActivity`, provide a button to call `Activity().activate()`, then listen for `Activity.sessions` to join
- TabletopKit handles synchronizing equipment state across participants automatically once the SharePlay session starts
- Spatial Personas appear seated at the table's seat positions, making remote players feel physically present

## APIs & Frameworks

**TabletopKit** **[NEW]**
- `Tabletop` **[NEW]** — represents the play surface
- `Tabletop.rectangular(entity:)` **[NEW]** — create a rectangular table from a RealityKit entity
- `TableVisualState.Pose2D` **[NEW]** — 2D position and rotation on the table surface
- `TableVisualState.Point2D` **[NEW]** — 2D position on the table surface
- `PlayerSeat` **[NEW]** — describes a seat position at the table
- `Equipment` **[NEW]** — protocol for game pieces
- `EntityEquipment` **[NEW]** — protocol for entity-backed equipment
- `BaseEquipmentState` **[NEW]** — initial state for equipment (position, size, interactability)
- `EquipmentIdentifier` **[NEW]** — unique identifier for each piece
- `TabletopInteraction` **[NEW]** — protocol; implement `update(context:value:)` to handle player interactions
- `TabletopInteractionContext` **[NEW]** — context passed to interaction callbacks
- `TabletopInteractionValue` **[NEW]** — describes the current interaction (phase, proposed destination, equipment involved)
- `TabletopInteractionValue.phase` **[NEW]** — `.started`, `.changed`, `.ended`
- `TabletopInteractionValue.proposedDestination` **[NEW]** — intended drop location
- `.tabletopGame(_:)` **[NEW]** — SwiftUI/RealityView modifier to bind a TabletopKit game to the scene

**RealityKit**
- `Entity` — loaded from Reality Composer Pro bundles; used as visual representations of equipment
- `RealityView` — hosts the tabletop scene
- `AudioLibraryComponent` — spatialized audio; attach to entities for sound effects

**GroupActivities / SharePlay**
- `GroupActivity` — protocol to define a shareable activity for multiplayer
- `GroupActivity.activate()` — start a SharePlay session
- `Activity.sessions` — async sequence of joined SharePlay sessions
- Spatial Personas — FaceTime spatial avatars automatically placed at seat positions

## Code Highlights

Create a rectangular table:
```swift
let entity = try! Entity.load(named: "table", in: table_Top_KitBundle)
let table = Tabletop.rectangular(entity: entity)
```

Define a player pawn conforming to `EntityEquipment`:
```swift
struct PlayerPawn: EntityEquipment {
    let id: EquipmentIdentifier
    let entity: Entity
    var initialState: BaseEquipmentState

    init(seat: PlayerSeat, pose: TableVisualState.Pose2D, entity: Entity) {
        self.id = .init()
        self.entity = entity
        self.initialState = .init(pose: pose, ...)
    }
}
```

Respond to a drag interaction ending:
```swift
func update(context: TabletopKit.TabletopInteractionContext,
            value: TabletopKit.TabletopInteractionValue) {
    switch value.phase {
    case .ended:
        guard let dst = value.proposedDestination else { return }
        // validate and apply move
    default: break
    }
}
```

Set up multiplayer with SharePlay:
```swift
import GroupActivities
// Start SharePlay
Button("SharePlay", systemImage: "shareplay") {
    Task { try! await Activity().activate() }
}
// Join sessions
Task { @MainActor in
    for await session in Activity.sessions { /* start multiplayer */ }
}
```

## Takeaways
- TabletopKit handles 3D placement, interaction routing, and multiplayer sync — implement `Equipment` and `TabletopInteraction` to describe your game's pieces and rules.
- All visual polish uses standard RealityKit — attach `AudioLibraryComponent` and particle emitters to entities just as you would in any RealityKit app.
- Multiplayer with spatial Personas requires minimal extra code: define a `GroupActivity`, activate it, and listen for sessions; TabletopKit keeps equipment state synchronized across players.
- Model each piece as a dedicated `Equipment`-conforming struct so Xcode's type system enforces correctness of your game state.

---
_Source: WWDC24 Session 10091 page (abstract, chapter list, code samples, and resource links)._
