# SwiftUI on watchOS
**WWDC19 · Session 219** · [Watch](https://developer.apple.com/videos/play/wwdc2019/219/)

_Platforms:_ watchOS 6

## Overview
SwiftUI is a completely new UI framework for watchOS — the first truly native, full-featured UI toolkit for the platform. This session demonstrates how SwiftUI transforms watchOS app development, enabling rich animations, interactive notifications, and immersive Digital Crown haptic experiences that were never previously possible with WatchKit alone.

Structured around a flashcard app called Pop Quiz, the session covers WatchKit/SwiftUI interoperability, new watchOS-specific list features (Carousel style, swipe-to-delete, drag-to-reorder), building interactive notification UIs, and the new `digitalCrownRotation` modifier family for creating custom crown-driven interactions with haptic feedback.

## Key Topics

### WatchKit and SwiftUI Interoperability
- `WKHostingController<T>` — a new `WKInterfaceController` subclass that hosts a SwiftUI view. Set via the `body` property (not IBOutlets). **[NEW]**
- `WKUserNotificationHostingController<T>` — hosting controller for notification interfaces. **[NEW]**
- `WKHostingController` can be used anywhere a `WKInterfaceController` is used: as initial controller in storyboard, in a paging interface, or pushed via WatchKit navigation APIs.
- SwiftUI `NavigationLink` can accept a WatchKit storyboard identifier as its destination (push from SwiftUI to WatchKit).
- SwiftUI views can embed WatchKit interface objects (e.g., Sign In with Apple button via `WKInterfaceObjectRepresentable`).

### List Enhancements for watchOS **[NEW]**
- `List` on watchOS now supports swipe to delete via `.onDelete(perform:)`. **[NEW]**
- Drag to reorder via `.onMove(perform:)`. **[NEW]**
- **Carousel list style** — `.listStyle(CarouselListStyle())`: items center in the screen as the user scrolls, giving focus and weight to each cell. **[NEW]** Best for small numbers of large/interactive cells.
- `.listRowPlatterColor(_:)` — colors the cell platter background. **[NEW]**
- Automatic right-to-left layout support with leading/trailing alignment.

### Interactive Notifications
- `WKUserNotificationHostingController` provides `didReceive(_:)` to extract notification payload and set notification actions.
- `body` property is automatically re-evaluated after `didReceive` — the notification view is always up-to-date.
- Full SwiftUI view hierarchy can be embedded in notification long look body.
- Gesture recognizers and animations work within notification bodies (e.g., tap to flip a flashcard, drag interaction).

### Digital Crown API **[NEW]**
Three usage patterns via `.digitalCrownRotation(_:from:through:by:sensitivity:isContinuous:isHapticFeedbackEnabled:)`:

1. **Free scrolling** — bind to a scroll offset; no discrete stops; rubber-banding haptics at limits:
   ```swift
   .digitalCrownRotation($offset, from: 0, through: maxOffset)
   ```

2. **Discrete steps (picker-style)** — use `by:` to define stride; settling haptic feedback at each step:
   ```swift
   .digitalCrownRotation($index, from: 0, through: 14, by: 1, sensitivity: .medium)
   ```

3. **Continuous rotation** — use `isContinuous: true` for wrap-around (e.g., alarm hour hand rotating around a circle):
   ```swift
   .digitalCrownRotation($angle, from: 0, through: 360, by: 1, sensitivity: .low, isContinuous: true)
   ```

- `.focusable(_:onFocusChange:)` — required to direct Digital Crown input to a specific view. **[NEW]**
- Haptic crown feedback (Series 4+): resistance and weight as content scrolls under crown input.
- `sensitivity` parameter: `.low`, `.medium`, `.high` — controls how much rotation is needed per step.

### Xcode Canvas Integration for watchOS
- SwiftUI previews run live in Xcode canvas without deploying to device — critical for fast watchOS iteration.
- The Xcode inspector (command-click) inserts real SwiftUI code (not storyboard XML) and teaches developers the API in context.
- Right-to-left layout support is automatic when localization strings include Arabic or Hebrew.

### `@EnvironmentObject` / App Object Binding
- `@EnvironmentObject` (called "app object binding" in this session) keeps model and views in sync — views update automatically when the model changes.

## APIs & Frameworks

### WatchKit — SwiftUI Integration **[NEW]**
- `WKHostingController<Body: View>` — hosts SwiftUI views in WatchKit interface controllers **[NEW]**
  - `var body: Body { get }` — override to return the root SwiftUI view
- `WKUserNotificationHostingController<Body: View>` — notification body hosting **[NEW]**
  - `func didReceive(_ notification: UNNotification)` — extract payload and actions
- `WKInterfaceObjectRepresentable` — embed WatchKit objects in SwiftUI

### SwiftUI — watchOS List **[NEW]**
- `List` — now supports swipe-to-delete and drag-to-reorder on watchOS **[UPDATED]**
- `.onDelete(perform:)` — swipe-to-delete handler **[NEW on watchOS]**
- `.onMove(perform:)` — drag-to-reorder handler **[NEW on watchOS]**
- `.listStyle(CarouselListStyle())` — carousel scrolling with centering **[NEW]**
- `.listRowPlatterColor(_:)` — cell background color **[NEW]**

### SwiftUI — Digital Crown **[NEW]**
- `.digitalCrownRotation(_:)` — simple binding to crown rotation value **[NEW]**
- `.digitalCrownRotation(_:from:through:)` — with range limits **[NEW]**
- `.digitalCrownRotation(_:from:through:by:sensitivity:isContinuous:isHapticFeedbackEnabled:)` — full control **[NEW]**
  - `sensitivity: DigitalCrownRotationalSensitivity` — `.low`, `.medium`, `.high`
  - `isContinuous: Bool` — wrap-around behavior
  - `isHapticFeedbackEnabled: Bool`
- `.focusable(_:onFocusChange:)` — directs crown input to a view **[NEW]**
- `animation(_:)` on bindings — `$value.animation(.default)` for animated crown updates **[NEW]**

## Code Highlights

Hosting SwiftUI in WatchKit:
```swift
class HostingController: WKHostingController<TopicList> {
    override var body: TopicList {
        return TopicList()
    }
}
```

List with delete, reorder, and carousel style:
```swift
List {
    ForEach(topics) { topic in
        NavigationLink(destination: FlashcardList(topic: topic)) {
            TopicCell(topic: topic)
        }
    }
    .onDelete { indices in topics.remove(atOffsets: indices) }
    .onMove { from, to in topics.move(fromOffsets: from, toOffset: to) }
}
.listStyle(CarouselListStyle())
```

Digital Crown with discrete steps:
```swift
@State private var selectedIndex: Double = 0

ZStack {
    ForEach(cards.indices, id: \.self) { index in
        CardView(card: cards[index])
            .modifier(CardTransformModifier(index: index, currentIndex: selectedIndex))
    }
}
.focusable(true)
.digitalCrownRotation(
    $selectedIndex.animation(.default),
    from: 0,
    through: Double(cards.count - 2),
    by: 1.0,
    sensitivity: .low
)
```

Interactive notification hosting:
```swift
class NotificationController: WKUserNotificationHostingController<FlipCardNotification> {
    var card: Flashcard?
    
    override func didReceive(_ notification: UNNotification) {
        let payload = notification.request.content.userInfo
        card = Flashcard(from: payload)
        addNotificationAction(...)
    }
    
    override var body: FlipCardNotification {
        FlipCardNotification(card: card)
    }
}
```

## Takeaways
- SwiftUI is a genuine replacement for WatchKit on watchOS 6, not just a supplementary layer — use `WKHostingController` as the entry point.
- The Carousel list style and the new delete/reorder support on watchOS `List` bring iOS-class list interactions to the watch.
- The `digitalCrownRotation` modifier family enables haptic, physics-like crown interactions (free scroll, discrete steps, continuous rotation) that WatchKit's `WKCrownSequencer` could never achieve.
- Live Xcode canvas previews eliminate the round-trip to device for iterating on watchOS UI — a major productivity improvement.

---
_Source: WWDC19 Session 219 page (abstract, chapter summaries, code samples, and resource links)._
