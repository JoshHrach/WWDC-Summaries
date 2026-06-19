# The SwiftUI cookbook for focus
**WWDC23 · Session 10162** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10162/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
This session is a comprehensive guide to SwiftUI's focus system across all Apple platforms. Focus is the mechanism by which the system directs keyboard input, Apple TV Remote swipes, and Apple Watch Digital Crown rotations to the correct on-screen control — analogous to a cursor for user attention. The session covers four core ingredients (focusable views, focus state, focused values, focus sections) and then demonstrates three real-world recipes: programmatic focus control for a grocery list, a custom focusable emoji rating picker with keyboard and Digital Crown support, and a focusable photo grid with key-press handling and type selection.

Key new APIs in iOS 17 and macOS Sonoma include `focusable(interactions:)` (fine-grained focus interaction types), `defaultFocus(_:_:)` on iOS 17, `onKeyPress(_:action:)` for raw keyboard input, the `.highlight` hover effect on tvOS 17, and the `SelectionShapeStyle` for focus-aware selection indicators.

## Key Topics

### Focus Concepts
Focus directs input (keyboard, remote, Digital Crown) to a specific view. The focused view is visually emphasized: macOS draws a focus ring border, watchOS draws a green border, tvOS applies a hover effect (lift/highlight). Focus behaves differently per platform and control type:
- **Edit interactions** — control needs continuous focus input (e.g., `TextField`); always focusable; receives focus on tap
- **Activate interactions** — control is a focus-driven alternative to click/tap (e.g., `Button`); requires "Keyboard navigation" system setting on macOS/iPadOS to focus via keyboard; does not receive focus on click

### Focusable Views (Updated in iOS 17/macOS Sonoma)
`.focusable(interactions:)` now accepts a `FocusInteractions` option set:
- `.edit` — focus for continuous editing (text fields, sliders)
- `.activate` — focus as alternative to pointer activation (buttons, custom controls)
- No argument — all interactions (container views, grids)

**macOS Sonoma change:** Prior to Sonoma, `.focusable()` with no arguments applied only activation semantics. Existing macOS code using `.focusable()` may need an explicit `interactions:` argument to maintain prior behavior.

Use `.contentShape(.capsule)` (or any `Shape`) to customize the focus ring outline to match the visual clipping shape of a custom control.

### Focus State
`@FocusState` binds a property to the focus system. Use `Bool` for a single view; use any `Hashable` type (e.g., `UUID`) to track focus among multiple views:
- `.focused($binding)` — Boolean binding, fired when this view has focus
- `.focused($binding, equals: value)` — value binding; focus sets the binding to `value`
- `defaultFocus(_:_:)` modifier **[available on iOS 17, expanded from macOS]** — sets the initial focus target when the view hierarchy first appears; only fires if the view has not been focused before
- Set the `@FocusState` property directly (e.g., `focusedItem = newItem.id`) to programmatically move focus

### Focused Values
`FocusedValueKey` + `FocusedValues` extension — defines a custom key for data that flows from the focused view hierarchy to remote consumers (e.g., menu commands):
- `.focusedValue(\.key, value)` — associates a value with the focus context of the view
- `.focusedSceneValue(\.key, value)` — associates with the active scene's focus context
- `@FocusedValue(\.key)` property wrapper — reads the value in any view
- `@FocusedBinding(\.key)` property wrapper — reads a `Binding` wrapped in the focused value

Use `SelectionShapeStyle` (`.selection`) for borders or indicators on selected items — it automatically adapts to accent color when focused and turns gray when the parent loses focus.

### Focus Sections
`.focusSection()` — marks a container as a target for directional focus movement (Apple TV Remote swipes, Tab key navigation) without making the container itself focusable. The section guides focus to its nearest focusable descendant. Requires the section to occupy more space than its contents to create a large enough focus target (use `Spacer()` inside stacks).

### Key Press Handling (New in iOS 17 / macOS Sonoma)
`.onKeyPress(_:action:)` — responds to hardware keyboard key presses on a focusable view:
- `onKeyPress(.return) { ... }` — handle specific named keys
- `onKeyPress(characters: .alphanumerics, phases: .down) { keyPress in ... }` — handle character sets; useful for type-selection (jump to item by first letter)
- Return `.handled` if the press was consumed; return `.ignored` to allow propagation up the hierarchy

### Move Command
`.onMoveCommand(perform:)` — handles directional movement (arrow keys on macOS, directional swipes on tvOS Remote). Direction is a `MoveCommandDirection`: `.left`, `.right`, `.up`, `.down`. Respect `LayoutDirection` environment value and swap `.left`/`.right` for right-to-left languages.

### tvOS 17: Highlight Hover Effect
`.focusEffectDisabled()` — suppresses the automatic focus ring on container views where it is redundant.

New hover effect `.highlight` **[NEW tvOS 17]** — adds perspective shift and specular shine; ideal for artwork/photo thumbnails (use instead of `.lift` which suits text-based content).

### watchOS: Digital Crown
`.digitalCrownRotation(_:from:through:by:sensitivity:)` — binds a `Double` state to Digital Crown rotation; use `.onChange(of:)` to translate rotation to selection.

`@Environment(\.isFocused)` — reads whether the current view has focus; use to draw the green border on watchOS.

## APIs & Frameworks

- **SwiftUI**
  - `focusable(interactions:)` **[UPDATED iOS 17/macOS Sonoma]**
    - `FocusInteractions` — `.edit`, `.activate`; no-arg = all
  - `.contentShape(_:)` — customize focus ring shape
  - `.focusEffectDisabled()` — suppress automatic focus ring
  - `@FocusState` — property wrapper for focus tracking
  - `.focused($binding)` / `.focused($binding, equals:)` — link view to FocusState binding
  - `.defaultFocus(_:_:)` — **[NEW on iOS 17]** set initial focus
  - `FocusedValueKey` protocol — define custom focused value keys
  - `FocusedValues` — extend to add custom key properties
  - `.focusedValue(\.key, value)` — associate value with view focus context
  - `.focusedSceneValue(\.key, value)` — associate value with scene focus context
  - `@FocusedValue(\.key)` — read focused value
  - `@FocusedBinding(\.key)` — read binding from focused value
  - `.focusSection()` — container as directional focus target
  - `.onKeyPress(_:action:)` **[NEW iOS 17/macOS Sonoma]** — handle keyboard key presses
    - `KeyPress.Result` — `.handled`, `.ignored`
    - `KeyEquivalent` — `.return`, `.escape`, etc.
    - `KeyPress.Phases` — `.down`, `.up`, `.repeat`, `.all`
  - `.onMoveCommand(perform:)` — handle directional movement
    - `MoveCommandDirection` — `.left`, `.right`, `.up`, `.down`
  - `SelectionShapeStyle` (`.selection`) **[NEW]** — focus-aware selection indicator color; gray when unfocused, accent color when focused
  - `@Environment(\.isFocused)` — read focus state in any view
  - `@Environment(\.layoutDirection)` — `LayoutDirection.leftToRight` / `.rightToLeft`
  - `.digitalCrownRotation(_:from:through:by:sensitivity:)` — watchOS Digital Crown binding
  - `.hoverEffect(.highlight)` **[NEW tvOS 17]** — photo/artwork hover effect
  - `.hoverEffect(.lift)` — existing tvOS hover effect for text-based content

## Code Highlights

Programmatic focus with `@FocusState` and `defaultFocus`:
```swift
struct GroceryListView: View {
    @State private var list = GroceryList()
    @FocusState private var focusedItem: GroceryList.Item.ID?

    var body: some View {
        List($list.items) { $item in
            TextField("Item Name", text: $item.name)
                .focused($focusedItem, equals: item.id)
                .onSubmit { addEmptyItem() }
        }
        .defaultFocus($focusedItem, list.items.last?.id)
        .toolbar {
            Button(action: addEmptyItem) {
                Label("New Item", systemImage: "plus")
            }
        }
    }

    private func addEmptyItem() {
        let newItem = list.addItem()
        focusedItem = newItem.id  // programmatically move focus
    }
}
```

Custom focusable control with move command and content shape:
```swift
struct RatingPicker: View {
    @Environment(\.layoutDirection) private var layoutDirection
    @Binding var rating: Rating?

    var body: some View {
        HStack { /* emoji views */ }
            .contentShape(.capsule)          // focus ring follows capsule
            .focusable(interactions: .activate)
            .onMoveCommand { direction in
                var dir = direction
                if layoutDirection == .rightToLeft {
                    if dir == .left { dir = .right } else if dir == .right { dir = .left }
                }
                // update rating based on dir
            }
    }
}
```

Grid with `onKeyPress` for Return and type selection:
```swift
LazyVGrid(columns: columns) { /* cells */ }
    .focusable()
    .focusEffectDisabled()
    .focusedValue(\.selectedRecipe, $selection)
    .onMoveCommand { direction in selectRecipe(direction, layoutDirection: layoutDirection) }
    .onKeyPress(.return) {
        navigateToRecipe(id: selection)
        return .handled
    }
    .onKeyPress(characters: .alphanumerics, phases: .down) { keyPress in
        selectRecipe(matching: keyPress.characters)
    }
```

tvOS focus sections and highlight hover effect:
```swift
HStack {
    List(categories, id: \.self) { NavigationLink($0, destination: Color.gray) }
        .focusSection()

    ScrollView {
        LazyVGrid(columns: [GridItem(), GridItem()]) {
            ForEach(recipes) { recipe in
                RecipeTile(recipe: recipe)
                    .focusable()
                    .hoverEffect(.highlight)
            }
        }
    }
    .focusSection()
}
```

## Takeaways

- Use `focusable(interactions: .edit)` for views that capture continuous input and `focusable(interactions: .activate)` for controls that are click/tap alternatives — on macOS Sonoma the distinction affects whether the view receives focus on click and whether it needs "Keyboard navigation" enabled.
- `@FocusState` with a `Hashable` ID type (not just `Bool`) enables precise programmatic focus management across a list; set the property value directly (e.g., `focusedItem = newItem.id`) to move focus without additional modifiers.
- `onKeyPress(_:action:)` is the iOS 17 / macOS Sonoma replacement for character-level keyboard handling in custom focusable views — use it for Return/Escape key actions and type selection (`.alphanumerics` character set).
- `focusSection()` solves directional navigation gaps on tvOS and macOS by making a container a valid focus movement target even when the actual focusable descendants are not spatially adjacent to the current focus position.

---
_Source: WWDC23 Session 10162 page (abstract, chapters, transcript, and code samples)._
