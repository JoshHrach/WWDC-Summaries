# What's New in watchOS Design
**WWDC20 · Session 10171** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10171/)

_Platforms:_ watchOS 7

## Overview
watchOS 7 eliminated all gesture-based Force Touch contextual menus from native apps and replaced them with explicit visual controls — buttons that are always visible, discoverable, and predictable. The session walks through five patterns Apple used across Activity, Stocks, Maps, Camera, Photos, Calendar, Messages, Mail, and Home to surface secondary actions without cluttering primary UI: sort/filter pickers in lists, swipe actions (`onDelete`), a More button for glanceable views, bottom-of-scroll action buttons, and a new toolbar button that hides beneath the navigation bar until needed.

## Key Topics

### Why Force Touch Menus Were Removed
Force Touch contextual menus made actions invisible — users had no visual cue that secondary actions existed. In watchOS 7, all actions must be surfaced through visible, predictable controls. The design goals: actions are discoverable and predictable, relevant actions are always visible, and no functionality is lost in the transition.

### Pattern 1: Sort and Filter Picker at Top of List
For long, scrollable lists, a sort or filter button at the very top of the list opens a modal with a linked table (Picker nested inside a List in SwiftUI). Tapping a row dismisses the modal and updates the list immediately. Examples: Activity Sharing (sort by exercise, steps, etc.) and Stocks (switch between percentage and points views).

Implementation: nest a `Picker` inside a `List` in SwiftUI. The picker presents as a modal table view automatically on watchOS.

### Pattern 2: Swipe Actions (onDelete)
Standard iOS swipe-to-delete (right-to-left swipe on a list row reveals a red Delete button) is now used in watchOS list views. Example: World Clock lets users swipe to remove a city. Implemented with the `.onDelete` modifier on a `ForEach` inside a `List`.

### Pattern 3: More Button (Ellipsis) for Glanceable Views
When a view is non-scrolling (full-screen map, camera viewfinder), a circular button with the SF Symbols `ellipsis` glyph in the app's key color sits in the bottom-right corner. Tapping it presents a modal sheet with contextual options.

Design spec for the More button: white circular container at 85% opacity, 1-point black outer glow at 50% opacity — legible on any background. Use an action-specific glyph instead of ellipsis when there is exactly one secondary action (e.g., Photos app uses a custom Watch Face glyph for "Create Watch Face"). Never put primary actions inside a More menu; be extremely selective about which secondary actions appear there.

### Pattern 4: Action Button at Bottom of Scrolling Detail View
A tappable button at the bottom of a detail view provides the most discoverable location for secondary actions on that content. Non-actionable information sits directly on the black background; only the action button is in a container that looks tappable. Use red text for destructive actions and add a confirmation alert if the deleted content is not easily recoverable. Examples: Calendar event detail shows a "Delete Event" (red) or "Email Sender" button.

### Pattern 5: Toolbar Button (New API)
A new SwiftUI `.toolbar` modifier places a button that hides beneath the navigation bar by default and becomes visible when the user scrolls up. Use it sparingly — only for actions that are essential to the app but not the primary reason the user opened the view. Example: the Compose button in Messages and Mail.

Design rules:
- Only use in a scrolling view (scrolling is what makes it discoverable)
- Only for actions essential to app function, not the most common action
- The button surfaces on upward scroll, hides again when scrolling down

### Hierarchical Navigation for Destination Switching
Some apps need a persistent destination-switching model rather than action buttons. When the app lands one level in (e.g., All Inboxes in Mail, primary home in Home app), tapping the navigation bar Back button reveals a top-level destination list. The app remembers the chosen destination across launches. Use this model only when the destination choice warrants permanence across sessions.

## APIs & Frameworks

### SwiftUI (watchOS)
- `Picker` inside `List` — presents as a modal linked-table picker for sort/filter **[pattern]**
- `ForEach.onDelete(_:)` modifier — swipe-to-delete in list rows **[existing, now recommended for watchOS]**
- `.toolbar { Button(...) }` modifier **[NEW — watchOS 7]** — button hidden under nav bar, revealed on scroll
- `Label(_:systemImage:)` — used inside toolbar button for title + SF Symbol icon
- `sheet(isPresented:content:)` — backs the More button modal presentation

### Design Tokens (No API)
- More button: `circle` container, SF Symbols `ellipsis`, app key color, 85% white opacity background, 1pt black glow at 50%
- Destructive button: red label text, confirmation alert for non-recoverable actions
- Hit region: follow HIG-recommended minimum tap targets per watch size; add transparent padding if needed

## Code Highlights

Sort/filter picker nested in a list:
```swift
List {
    Picker(selection: $viewing,
           label: Text("Viewing")) {
        // Viewing options (e.g., Text("Percentage").tag(0), Text("Points").tag(1))
    }
    // Remaining list rows
}
```

Swipe to delete with onDelete:
```swift
List {
    ForEach(model.locations) {
        ClockCell(location: $0)
    }
    .onDelete { deleteClock(index: $0) }
}
```

Toolbar button (hidden under nav bar, appears on scroll):
```swift
.toolbar {
    Button(action: newMessage) {
        Label("New Message",
              systemImage: "square.and.pencil")
    }
}
```

## Takeaways

- Force Touch contextual menus are gone in watchOS 7 — every action must be surfaced through a visible, tappable control; design for discoverability from the start.
- The five replacement patterns map cleanly to UI context: picker-in-list for sort/filter, `onDelete` for list row deletion, More button for non-scrolling glanceable views, bottom action button for detail views, and `.toolbar` button for important-but-secondary actions in scrolling views.
- The `.toolbar` modifier is new for watchOS in WWDC20 and matches SwiftUI's cross-platform toolbar API — use it only in scrolling views and only for actions that are essential but not primary.
- Hierarchical navigation (back to a destination list) is appropriate when destination choice is persistent across sessions; it is not a substitute for action buttons.

---
_Source: WWDC20 Session 10171 page (abstract, transcript, code samples, and resource links)._
