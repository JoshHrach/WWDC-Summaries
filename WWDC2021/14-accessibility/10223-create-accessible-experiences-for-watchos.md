# Create Accessible Experiences for watchOS
**WWDC21 · Session 10223** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10223/)

_Platforms:_ watchOS 8

## Overview
watchOS 8 introduces two significant accessibility upgrades: large accessibility text sizes for Dynamic Type, and AssistiveTouch — a completely reimagined motor-accessibility feature that lets users navigate the Apple Watch without touching the screen, using only hand gestures or wrist motions. This session walks through building a SwiftUI watchOS app that fully supports both features, covering Dynamic Type layout strategies, VoiceOver best practices, and AssistiveTouch cursor and action menu customization.

The session is structured around a plant-care app that demonstrates real-world issues developers encounter — fixed font sizes breaking Dynamic Type, excessive VoiceOver navigation elements, custom controls without adjustable traits — and shows the exact API fixes for each.

## Key Topics

### Dynamic Type on watchOS 8
watchOS 8 adds large accessibility text sizes (previously only standard sizes were available). Users setting up Apple Watch now see a text size picker; the watch also auto-selects the closest size to the paired iPhone setting. Three rules for supporting Dynamic Type on watchOS:
1. **Always use a text style, not a fixed font size.** Use `.font(.title3)`, `.font(.caption2)`, etc. — the 11 system text styles scale automatically.
2. **Allow text to wrap.** Remove hard `lineLimit(1)` or set a higher limit (e.g., `.lineLimit(3)`) so text expands rather than truncates.
3. **Switch to a vertical layout for the largest sizes.** Read `@Environment(\.sizeCategory)` and render a stacked layout when `sizeCategory >= .extraExtraLarge`.

### VoiceOver in SwiftUI
SwiftUI provides accessibility for free on most standard controls; custom work is minimal. Key improvements:
- **Reduce navigation elements**: Let `NavigationLink` combine children automatically (removing `.accessibilityElement(children: .contain)` causes `NavigationLink` to merge all child accessibility info into one element).
- **Provide descriptive labels**: Use `.accessibilityLabel(_:)` on `Image` views and text with abbreviations. SF Symbol default labels (e.g., "Drop, fill") must be overridden for context-specific meaning ("Watering in five days").
- **Custom controls**: For custom stepper/counter views, use `.accessibilityElement()` to collapse children, `.accessibilityAdjustableAction` for swipe-up/down increment/decrement, `.accessibilityLabel` for the control name, and `.accessibilityValue` for the current reading (spoken on every change).
- **Complications**: Text abbreviations need `.accessibilityLabel` with full-form strings. Image complications need explicit labels.

### AssistiveTouch (New in watchOS 8)
AssistiveTouch enables full Apple Watch use without touching the screen. Navigation is via hand gestures: clench = tap; double-clench = action menu; pinch = next element; double-pinch = previous element. Wrist tilt (Dwell Control) moves an on-screen pointer for those who cannot use gestures.

**Focusable elements**: Only interactive elements (Button, Toggle, NavigationLink, Text with tap gesture or `.accessibilityAction`, elements with `.accessibilityAddTraits(.isButton)`) are focusable. Static labels and Text without interaction are not focusable.

**Making a static element focusable**: Apply `.accessibilityRespondsToUserInteraction(true)` to any view that should receive AssistiveTouch focus even though it is not a built-in interactive control.

**Cursor frame**: The AssistiveTouch cursor frame matches the element's tappable area. Enlarge it with `.contentShape(_:)` — e.g., `.contentShape(Circle().scale(1.5))` on a `NavigationLink` with a small icon.

**Action menu**: When the user double-clench brings up the menu, AssistiveTouch surfaces the element's VoiceOver custom actions as menu items. Custom actions defined with `.accessibilityAction(_:label:)` (using a `Label` with an image) display with an icon; otherwise the first letter of the name is used.

## APIs & Frameworks

**SwiftUI Accessibility** (`import SwiftUI`) — watchOS 8

- `.font(_ style: Font.TextStyle)` — use instead of fixed `.font(.system(size:))`; text styles: `.largeTitle`, `.title`, `.title2`, `.title3`, `.headline`, `.subheadline`, `.body`, `.callout`, `.footnote`, `.caption`, `.caption2`
- `.lineLimit(_ number: Int?)` — allow wrapping; pass `nil` for unlimited
- `@Environment(\.sizeCategory) var sizeCategory: ContentSizeCategory` — detect Dynamic Type category; compare with `<`, `>=` for layout switching
- `ContentSizeCategory` — `.extraSmall` through `.accessibilityExtraExtraExtraLarge`
- `.accessibilityLabel(_ label: Text)` / `.accessibilityLabel(_ string: String)` — provide VoiceOver label
- `.accessibilityValue(_ value: Text)` / `.accessibilityValue(_ string: String)` — spoken on value changes
- `.accessibilityElement(children: AccessibilityChildBehavior)` — `.combine`, `.contain`, `.ignore` (default is `.ignore`)
- `.accessibilityAdjustableAction(_ handler:)` — swipe-up/down callbacks for custom adjustable controls; receives `AccessibilityAdjustmentDirection` (`.increment`, `.decrement`)
- `.accessibilityAction(_ action:label:)` **[NEW watchOS 8]** — custom action with `Label` for AssistiveTouch action menu icon
- `.accessibilityRespondsToUserInteraction(_ value: Bool)` **[NEW watchOS 8]** — makes a non-interactive view focusable by AssistiveTouch
- `.contentShape(_ shape: Shape)` — changes tappable area and AssistiveTouch cursor frame

## Code Highlights

Dynamic Type: text style and wrapping:
```swift
struct PlantView: View {
    @Binding var plant: Plant
    var body: some View {
        VStack(alignment: .leading) {
            Text(plant.name)
                .font(.title3)          // text style, not fixed size
            PlantTaskList(plant: $plant)
        }
    }
}

struct PlantTaskLabel: View {
    var body: some View {
        HStack { /* ... */ }
            .lineLimit(3)               // allow wrapping, not hard-clipped to 1
            .font(.caption2)
    }
}
```

Dynamic Type: alternate layout at accessibility sizes:
```swift
struct PlantContainerView: View {
    @Environment(\.sizeCategory) var sizeCategory
    @Binding var plant: Plant
    var body: some View {
        if sizeCategory < .extraExtraLarge {
            PlantViewHorizontal(plant: $plant)
        } else {
            PlantViewVertical(plant: $plant)
        }
    }
}
```

Custom adjustable control for VoiceOver + AssistiveTouch:
```swift
CustomCounter(value: value, increment: increment, decrement: decrement)
    .accessibilityElement()
    .accessibilityAdjustableAction { direction in
        switch direction {
        case .increment: increment()
        case .decrement: decrement()
        default: break
        }
    }
    .accessibilityLabel("\(task.name) frequency")
    .accessibilityValue("\(value) days")
```

AssistiveTouch: make static view focusable and enlarge cursor:
```swift
FreeDrinkInfoView()
    .accessibilityRespondsToUserInteraction(true)   // focusable by AssistiveTouch

NavigationLink(destination: EditView()) {
    Image(systemName: "ellipsis").symbolVariant(.circle)
}
.contentShape(Circle().scale(1.5))                  // larger cursor frame
```

AssistiveTouch: custom action with icon in action menu:
```swift
PlantContainerView(plant: plant)
    .accessibilityElement(children: .combine)
    .accessibilityAction {
        // Edit action
    } label: {
        Label("Edit", systemImage: "ellipsis.circle")
    }
```

## Takeaways
- watchOS 8 large accessibility text sizes require switching from fixed font sizes to text styles and allowing multi-line wrapping; add an alternate vertical layout for the largest categories using `@Environment(\.sizeCategory)`.
- SwiftUI's automatic accessibility is excellent — most improvements come from removing unnecessary `.accessibilityElement(children: .contain)` groupings and adding a few `.accessibilityLabel` overrides for images and abbreviations.
- Custom controls (steppers, counters) need `.accessibilityElement()` + `.accessibilityAdjustableAction` + label + value to be usable by both VoiceOver and AssistiveTouch.
- AssistiveTouch focuses only interactive elements; use `.accessibilityRespondsToUserInteraction(true)` for tap-gesture views and `.contentShape(_:)` to enlarge cursor frames on small targets.
- Custom actions defined for VoiceOver automatically appear in the AssistiveTouch action menu; supply a `Label` with an image for a proper icon.

---
_Source: WWDC21 Session 10223 page (abstract, chapter summaries, code samples, and resource links)._
