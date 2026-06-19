# Integrating SwiftUI
**WWDC19 · Session 231** · [Watch](https://developer.apple.com/videos/play/wwdc2019/231/)

_Platforms:_ iOS 13, iPadOS 13, macOS 10.15 Catalina, watchOS 6, tvOS 13

## Overview
SwiftUI is designed to integrate bidirectionally with existing UIKit, AppKit, and WatchKit codebases. This session covers all four integration paths: hosting SwiftUI content in existing view/controller hierarchies, embedding existing UIKit/AppKit/WatchKit views inside SwiftUI, connecting existing data models to SwiftUI's reactive update system, and using system services (drag and drop, paste, focus) from SwiftUI.

The session also covers Objective-C interoperability with SwiftUI and introduces two Xcode 11 features that enable easier wiring between storyboards and hosting controllers: `IBSegueAction` for type-safe segue instantiation, and Auto Layout support when hosting controllers are embedded in container views.

## Key Topics

**Hosting SwiftUI in Existing Apps**
- `UIHostingController<Content: View>` — `UIViewController` subclass; takes `rootView` at init; use in storyboards or push/present in code
- `NSHostingController<Content: View>` — same for AppKit/macOS; presents SwiftUI as `NSViewController`
- `NSHostingView<Content: View>` — `NSView` subclass for embedding SwiftUI in an AppKit view hierarchy without a view controller; respects `intrinsicContentSize` automatically
- `WKHostingController<Body: View>` — WatchKit subclass; override `body` property to return SwiftUI view; call `setNeedsBodyUpdate()` / `updateBodyIfNeeded()` when WatchKit-side data changes
- `WKUserNotificationHostingController` — SwiftUI for dynamic interactive notifications on watchOS; `body` updated on each `didReceive`
- `IBSegueAction` **[NEW in Xcode 11]** — Control+drag from a storyboard segue to code creates a method that initializes the destination view controller directly; use to pass `rootView` to a `UIHostingController` without `prepareForSegue`
- Auto Layout: container views embedding hosting controllers automatically respect `intrinsicContentSize` of the SwiftUI root view

**Embedding UIKit/AppKit/WatchKit Views in SwiftUI**
- `UIViewRepresentable` — protocol for embedding a `UIView` in SwiftUI; `makeUIView(context:)` + `updateUIView(_:context:)`
- `UIViewControllerRepresentable` — protocol for embedding a `UIViewController` in SwiftUI; `makeUIViewController(context:)` + `updateUIViewController(_:context:)`
- `NSViewRepresentable` — AppKit equivalent for embedding `NSView` in SwiftUI
- `NSViewControllerRepresentable` — AppKit equivalent for embedding `NSViewController`
- `WKInterfaceObjectRepresentable` — protocol for embedding supported WatchKit objects in SwiftUI
- `makeCoordinator()` — optional; return a custom coordinator class to manage delegation, data sources, and target-action between the UIKit/AppKit view and SwiftUI
- `RepresentableContext` — passed to make/update methods; provides `coordinator`, `environment` (reading `colorScheme`, size classes, layout direction, custom environment values), and `transaction` (animation detection)
- `dismantleUIView(_:coordinator:)` / `dismantleUIViewController(_:coordinator:)` — optional static cleanup before view removal

**Data Model Integration**
- `BindableObject` protocol — implement one property: `var didChange: PublishedType` where `PublishedType: Publisher`; SwiftUI subscribes to `didChange` automatically when a view uses `@ObjectBinding`
- `@ObjectBinding` property wrapper — wraps a `BindableObject`; the `$` prefix yields a `Binding` for two-way read/write access
- Data must be published on the main thread; use `.receive(on: RunLoop.main)` or `.receive(on: DispatchQueue.main)` operator on the publisher
- Combine publisher types usable as `didChange`:
  - `NotificationCenter.Publisher` — for `NotificationCenter`-posting data models
  - `KVO publisher` — `object.publisher(for: \.keyPath)` for any KVO-compliant object
  - Merged publishers — `publisher1.merge(with: publisher2)` for multiple change sources
  - `PassthroughSubject` — for manually triggered notifications (e.g., `NSFetchedResultsControllerDelegate.controllerDidChangeContent`)
- The data model remains the single source of truth; combining `BindableObject` + `@ObjectBinding` eliminates the need for view-controller-based data synchronization code

**Drag and Drop**
- `.onDrag { NSItemProvider }` modifier — enables view as drag source; snapshot of the view used as drag image
- `.onDrop(of:isTargeted:perform:)` modifier — accepts dropped items of specified UTI types; `action` closure receives `[NSItemProvider]` and drop `CGPoint`
- `.onDrop(of:delegate:)` — delegate variant gives visibility into drag-over location before drop
- `NSItemProvider` — Foundation class; asynchronous data loading; always process item provider results and update SwiftUI data on the main thread

**Paste and Focus**
- `.onPasteCommand(supportedContentTypes:perform:)` modifier — accepts paste action for listed UTI types; no location parameter (paste is indirect)
- Focus system determines which view receives indirect actions (keyboard input, menu commands, Digital Crown, tvOS Siri Remote): SwiftUI manages focus transitions automatically
- `.focusable(_ isFocusable: Bool, onFocusChange:)` modifier — opt view into the focus system; closure called when focus is gained/lost (for visual feedback)
- `.onCommand(_ selector: Selector, perform:)` modifier — handles Objective-C selector-based menu/toolbar actions dispatched to first responder; chainable for multiple commands
- `.onMoveCommand`, `.onPlayPauseCommand`, `.onExitCommand`, `.digitalCrownRotation` — platform-specific focus action modifiers
- `@Environment(\.undoManager) var undoManager` — access the shared `UndoManager`; no new code needed in most cases when SwiftUI views are hosted in an existing app

**Objective-C Integration**
- Mark a Swift hosting controller subclass `@objc` to instantiate it from Objective-C
- Wrap a `BindableObject` in Swift, hold a reference to the Objective-C data model, subscribe to its `NotificationCenter` notifications in `didChange`
- Pass the wrapped data model to the SwiftUI `rootView` from the hosting controller

## APIs & Frameworks

**SwiftUI**
- `UIHostingController<Content>` **[NEW]** — host SwiftUI in UIKit
- `NSHostingController<Content>` **[NEW]** — host SwiftUI in AppKit
- `NSHostingView<Content>` **[NEW]** — embed SwiftUI in an NSView hierarchy
- `WKHostingController<Body>` **[NEW]** — host SwiftUI in WatchKit interface controllers
- `WKUserNotificationHostingController<Body>` **[NEW]** — SwiftUI in notification controllers
- `UIViewRepresentable` **[NEW]** — embed UIView in SwiftUI
- `UIViewControllerRepresentable` **[NEW]** — embed UIViewController in SwiftUI
- `NSViewRepresentable` **[NEW]** — embed NSView in SwiftUI
- `NSViewControllerRepresentable` **[NEW]** — embed NSViewController in SwiftUI
- `WKInterfaceObjectRepresentable` **[NEW]** — embed WatchKit objects in SwiftUI
- `BindableObject` **[NEW]** — protocol; `var didChange: Publisher`
- `@ObjectBinding` **[NEW]** property wrapper — subscribe a view to a `BindableObject`
- `View.onDrag(_:)` **[NEW]**
- `View.onDrop(of:isTargeted:perform:)` **[NEW]**
- `View.onPasteCommand(supportedContentTypes:perform:)` **[NEW]**
- `View.focusable(_:onFocusChange:)` **[NEW]**
- `View.onCommand(_:perform:)` **[NEW]**
- `@Environment(\.undoManager)` **[NEW]**

**Combine (referenced)**
- `Publisher`, `PassthroughSubject`, `.receive(on:)`, `.merge(with:)` — used to implement `BindableObject.didChange`
- `NotificationCenter.Publisher` — `NotificationCenter.default.publisher(for:object:)`
- KVO publisher — `object.publisher(for: keyPath)`

## Code Highlights

Hosting SwiftUI in a UIKit app via storyboard IBSegueAction:
```swift
@IBSegueAction func showPlantDetails(_ coder: NSCoder) -> UIViewController? {
    return UIHostingController(coder: coder, rootView: PlantDetailsView(plant: selectedPlant))
}
```

Wrapping a UIView for use in SwiftUI:
```swift
struct RatingsControlRepresentation: UIViewRepresentable {
    @Binding var rating: Int

    func makeUIView(context: Context) -> UIKitRatingsControl {
        let control = UIKitRatingsControl()
        control.addTarget(context.coordinator, action: #selector(Coordinator.ratingChanged(_:)), for: .valueChanged)
        return control
    }

    func updateUIView(_ view: UIKitRatingsControl, context: Context) {
        view.rating = rating
    }

    func makeCoordinator() -> Coordinator { Coordinator(rating: $rating) }

    class Coordinator: NSObject {
        var rating: Binding<Int>
        init(rating: Binding<Int>) { self.rating = rating }
        @objc func ratingChanged(_ control: UIKitRatingsControl) {
            rating.wrappedValue = control.rating
        }
    }
}
```

Conforming an existing data model to BindableObject using NotificationCenter:
```swift
class PlantsDataModel: BindableObject {
    var didChange = NotificationCenter.default
        .publisher(for: .plantsDidChange, object: self)
        .receive(on: RunLoop.main)

    var plants: [Plant] = [] {
        didSet { NotificationCenter.default.post(name: .plantsDidChange, object: self) }
    }
}
```

## Takeaways
- `UIHostingController` with `IBSegueAction` is the simplest path to add SwiftUI to an existing storyboard-based app — no `prepareForSegue` boilerplate needed.
- `UIViewRepresentable.makeCoordinator()` is essential when a UIKit view needs to send events back to SwiftUI; return a coordinator and use target-action or delegation on it.
- Make `BindableObject.didChange` emit on the main thread — data mutations from background queues that skip `.receive(on: RunLoop.main)` will cause crashes or silent misbehavior.
- The `focusable()` modifier is required on tvOS and watchOS for any custom view that needs to receive Siri Remote or Digital Crown input.

---
_Source: WWDC19 Session 231 page (abstract, chapter summaries, code samples, and resource links)._
