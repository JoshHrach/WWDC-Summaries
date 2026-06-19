# Meet Distributed Actors in Swift
**WWDC22 · Session 110356** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110356/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9, Linux (server-side Swift)

## Overview
Swift 5.7 introduces distributed actors — an extension of the existing actor model that allows actors to transparently operate across process and network boundaries. Where a regular Swift actor isolates state within a single process, a `distributed actor` can be running on a remote server, another device on the local network, or in the same process as the caller; the call site looks identical in all three cases. This property is called **location transparency** and is the central design goal of the feature.

The session walks through converting a local actor to a distributed actor, moving it to a server-side Swift process over WebSockets, and building a peer-to-peer multiplayer mode using a local-network actor system with a receptionist for actor discovery. It also introduces the open-source Swift Distributed Actors Cluster library for server-side clustering use cases.

## Key Topics
- **`distributed actor` declaration** — add `distributed` keyword before `actor`; the type automatically conforms to `DistributedActor` protocol; the compiler synthesizes `id` and `actorSystem` properties
- **Actor system requirement** — every distributed actor must declare `typealias ActorSystem` pointing to a concrete `DistributedActorSystem` type; the actor system handles all serialization, transport, and identity management
- **`LocalTestingDistributedActorSystem`** — ships with the `Distributed` module; enables distributed actor isolation checks and local use without any networking; ideal for unit testing
- **`distributed` methods** — only methods marked `distributed` may be called on potentially remote actor references; this defines the explicit remote API surface; non-`distributed` methods remain local-only
- **Serialization requirement** — all parameters and return types of `distributed` methods must conform to the actor system's serialization requirement (typically `Codable`); the compiler enforces this
- **Actor ID and `resolve`** — `DistributedActor.resolve(id:using:)` returns a local or remote reference to an actor with the given ID; the call is synchronous and non-blocking (no networking at resolve time); the actor may not yet exist on the remote side when resolved
- **On-demand actor creation** — a server-side actor system can implement a `registerOnDemandResolveHandler` so that actors are created the first time a message arrives for a previously unknown ID
- **Receptionist pattern** — actor systems may expose a `receptionist` that actors check in with to become discoverable; other nodes subscribe to listings by actor type and tag via an async sequence; enables type-safe peer-to-peer actor discovery without a central server
- **Cluster actor system** — the open-source `swift-distributed-actors` package provides a production-grade cluster system built on SwiftNIO with failure detection and a cluster-wide receptionist
- **Location transparency** — game logic, move generation, and network topology are completely decoupled; the same `distributed actor` code works locally, client-server, peer-to-peer, or in a cluster by swapping the `ActorSystem` typealias

## APIs & Frameworks
**`Distributed` module** **[NEW]** — `import Distributed`
- `distributed actor` keyword **[NEW]** — declares a distributed actor type
- `DistributedActor` protocol **[NEW]** — synthesized conformance; requires `actorSystem` property and `id` synthesized property
- `DistributedActorSystem` protocol **[NEW]** — implemented by actor system libraries; handles `resolve`, serialization, and transport
- `LocalTestingDistributedActorSystem` **[NEW]** — built-in no-networking actor system for testing and local use
- `distributed func` **[NEW]** — marks a method as callable on remote actor references; all parameters and return types must satisfy the actor system's serialization requirement
- `DistributedActor.resolve(id:using:) throws -> Self` **[NEW]** — static method; returns local instance or remote proxy for the given ID; synchronous
- `ActorIdentity` (or `ActorSystem.ActorID`) — actor identity type; assigned by the actor system on init
- Receptionist pattern — `actorSystem.receptionist.checkIn(actor:tag:)` and `actorSystem.receptionist.listing(of:tag:) -> AsyncSequence` (API shape varies by actor system implementation)

**Swift Distributed Actors Cluster (open source)**
- `swift-distributed-actors` package — SwiftNIO-based cluster actor system; failure detection, cluster-wide receptionist; available as beta alongside Swift 5.7

**Related Swift Concurrency**
- `actor` — existing local actor (prerequisite knowledge)
- `async`/`await`, `AsyncSequence` — used for remote calls and receptionist listings

## Code Highlights
Converting a local actor to a distributed actor:
```swift
import Distributed

public distributed actor BotPlayer {
    typealias ActorSystem = LocalTestingDistributedActorSystem

    var ai: RandomPlayerBotAI
    var gameState: GameState

    public init(team: CharacterTeam, actorSystem: ActorSystem) {
        self.actorSystem = actorSystem          // initialize synthesized property first
        self.gameState = .init()
        self.ai = RandomPlayerBotAI(playerID: self.id, team: team)  // use synthesized `id`
    }

    public distributed func makeMove() throws -> GameMove {
        return try ai.decideNextMove(given: &gameState)
    }

    public distributed func opponentMoved(_ move: GameMove) async throws {
        try gameState.mark(move)
    }
}
```

Resolving a remote actor reference (client side):
```swift
let opponentID: BotPlayer.ID = .randomID(opponentFor: self.id)
let bot = try BotPlayer.resolve(id: opponentID, using: sampleWebSocketSystem)
// bot may refer to a not-yet-created actor on the server
let move = try await bot.makeMove()
```

Server-side on-demand actor creation:
```swift
let system = try SampleWebSocketActorSystem(mode: .serverOnly(host: "localhost", port: 8888))
system.registerOnDemandResolveHandler { id in
    if system.isBotID(id) {
        return system.makeActorWithID(id) {
            OnlineBotPlayer(team: .rodents, actorSystem: system)
        }
    }
    return nil
}
try await system.terminated
```

Peer-to-peer actor discovery via receptionist:
```swift
let opponentTeam: CharacterTeam = model.team == .fish ? .rodents : .fish
let listing = await localNetworkSystem.receptionist.listing(of: OpponentPlayer.self,
                                                            tag: opponentTeam.tag)
for try await opponent in listing where opponent.id != self.player.id {
    model.foundOpponent(opponent, myself: self.player, informOpponent: true)
    return  // only need one opponent
}
```

## Takeaways
- `distributed actor` extends Swift's data-race-safety guarantees across process and network boundaries; only `distributed` methods are callable remotely, giving you explicit control over the remote API surface.
- The actor system is fully pluggable: swap the `ActorSystem` typealias to move from local testing → WebSocket client/server → local-network peer-to-peer → cluster, without changing the actor's business logic.
- `DistributedActor.resolve(id:using:)` is synchronous; actor IDs can be "made up" on the client and the actor materialized on-demand by the server when the first message arrives.
- The receptionist pattern provides type-safe, async actor discovery for peer-to-peer and cluster scenarios, replacing ad-hoc service discovery with strongly-typed Swift APIs.

---
_Source: WWDC22 Session 110356 page (abstract, chapter summaries, code samples, and resource links)._
