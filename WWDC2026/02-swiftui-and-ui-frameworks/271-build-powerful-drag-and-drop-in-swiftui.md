# Code-along: Build Powerful Drag and Drop in SwiftUI
**WWDC26 · Session 271** · [Watch](https://developer.apple.com/videos/play/wwdc2026/271/)

_Platforms:_ iOS, iPadOS, macOS, visionOS

## Overview
This code-along session builds a fully playable Solitaire game to demonstrate SwiftUI's expanded drag-and-drop capabilities available in the 2027 OS releases. Starting from a working card game skeleton, it progressively adds reordering, multi-item dragging, and precise drag/drop configuration to produce a polished, game-rule-aware interaction model.

The three new capabilities introduced are: a unified reorder API (`reorderable` + `reorderContainer`) that replaces ad-hoc `onDrop`/`onDrag` boilerplate; a `dragContainer` modifier that controls which items are lifted together when a drag starts; and `DragConfiguration`/`DropConfiguration` types that let source and destination negotiate the correct transfer operation (move vs. copy vs. forbidden). The companion sample project, "Making a card game with drag, drop, and reordering in SwiftUI," contains the finished code.

Prerequisite: familiarity with the `Transferable` protocol (WWDC22 "Meet Transferable").

## Key Topics

### Reordering (1:42)
The new `.reorderable()` modifier marks items inside a `ForEach` as eligible for drag-reorder. A `.reorderContainer(for:in:)` on an ancestor view receives a `ReorderDifference` callback when the user drops. Multiple `ForEach` groups can share a single container by providing a `collectionID`, enabling cross-pile moves in Solitaire. Face-down cards are excluded simply by not applying `.reorderable()` to that iteration.

### Drag Multiple Items (6:50)
The `.dragContainer(for:)` modifier intercepts the moment a drag begins on an item and returns an array of IDs to lift together — used to pick up an entire stack of face-up cards as one drag. Visual formation of the lifted items is set with `.dragPreviewsFormation(.stack)`. The appearance over a potential drop target is independently set with `.dropPreviewsFormation(.stack)`.

### Drag Configuration (9:59)
`DragConfiguration(allowMove: true)` on a drag source declares that the source permits the move operation. A `.dropConfiguration` closure on the destination returns a `DropConfiguration` specifying both the intended `DropOperation` (.move, .copy, .forbidden) and the target `ReorderDifference.Destination`. This two-sided negotiation prevents duplication: the card disappears from the deck only when the pile confirms acceptance.

## APIs & Frameworks

**SwiftUI**
- **[NEW]** `.reorderable(collectionID:)` view modifier
- **[NEW]** `.reorderContainer(for:in:)` view modifier
- **[NEW]** `ReorderDifference<ItemID, CollectionID>` — result of a reorder gesture
- **[NEW]** `ReorderDifference.Destination` (`.before(id:)` / `.end`)
- **[NEW]** `.dragContainer(for:)` modifier — customize which items are lifted
- **[NEW]** `DragConfiguration` — `allowMove: Bool`
- **[NEW]** `.dragConfiguration(_:)` modifier
- **[NEW]** `.dropConfiguration` closure modifier — returns `DropConfiguration`
- **[NEW]** `DropConfiguration` — `operation: DropOperation`, `destination:`
- **[NEW]** `DropOperation` enum (`.move`, `.copy`, `.forbidden`)
- **[NEW]** `.dragPreviewsFormation(_:)` modifier
- **[NEW]** `.dropPreviewsFormation(_:)` modifier
- **[NEW]** `DragPreviewsFormation` (`.stack`, `.default`, etc.)
- **[NEW]** `.draggable(containerItemID:)` modifier — marks an item as draggable within a container
- `DropSession.reorderDestination(for:in:)` — extract destination from session
- `DropSession.suggestedOperations` — the operation the source proposes
- `.dropDestination(for:)` modifier (existing, used in conjunction with new APIs)
- `Transferable` protocol (prerequisite)

**UIKit / AppKit**
- `Drag and drop` (UIKit) — referenced in Resources for platform integration

## Code Highlights

Reorderable pile in Solitaire:
```swift
ForEach(cards[index...], id: \.value) { card in
    CardView(card: card)
}
.reorderable(collectionID: Card.Group.pile(index))
```

Multi-item drag container:
```swift
.dragContainer(for: CardValue.self) { cardID in
    game.cardStack(startingAt: cardID)  // returns [CardValue]
}
.dragPreviewsFormation(.stack)
```

Drop configuration with rule enforcement:
```swift
.dropConfiguration { session in
    let allowed = session.suggestedOperations.contains(.move)
        && game.validateMove(session: session, destination: destination)
    return DropConfiguration(
        operation: allowed ? .move : .forbidden,
        destination: destination
    )
}
```

## Takeaways
- Adopt `.reorderable()` + `.reorderContainer` to replace custom `onDrag`/`onDrop` reorder implementations across List, LazyVGrid, and custom layouts.
- Use `.dragContainer` to bundle related items (e.g., card stacks) into a single multi-item drag without extra state management.
- Always pair `DragConfiguration(allowMove:)` on the source with a `.dropConfiguration` on the destination so moves are never duplicated or lost.
- Download the "Making a card game" sample project to see all three capabilities wired together end-to-end.

---
_Source: WWDC26 Session 271 page (abstract, chapter summaries, code samples, and resource links)._
