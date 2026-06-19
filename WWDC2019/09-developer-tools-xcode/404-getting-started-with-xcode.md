# Getting Started with Xcode
**WWDC19 · Session 404** · [Watch](https://developer.apple.com/videos/play/wwdc2019/404/)

_Platforms:_ iOS, iPadOS, macOS (Xcode 11)

## Overview
A comprehensive end-to-end walkthrough of Xcode 11 for developers new to the platform. Using a meditation app (Mind) as the running example, the session covers every major phase of development: project creation, UI building with SwiftUI and Canvas previews, debugging, adding Swift Package dependencies, creating framework targets, writing unit and UI tests, and distributing via App Store / TestFlight.

The session tours the Xcode 11 Source Editor's new and improved features — including the mini map, multi-cursor editing, live issues, Fix-Its, structured editing from the Action Menu, and the revamped Canvas for interactive SwiftUI previews. It also demonstrates how Swift Packages integrate natively into Xcode projects from the Project Editor, and how Test Plans in Xcode 11 organize and share test configurations across schemes.

## Key Topics

**Xcode UI Orientation**
- Source Editor, Project Editor, Navigator (Project, Find, Test, Debug, Report navigators), Inspector, Toolbar
- Scheme (build/run/test rules) + Run Destination (simulator, device, Mac)
- Show/Hide panels; Open Quickly (Cmd+Shift+O) for fast file/symbol navigation

**Source Editor Improvements (Xcode 11)**
- Mini Map: miniature file view in the right margin, navigable via section marks and hovering labels
- Multi-cursor editing: Control+Shift+click to add additional insertion points; commands (copy, paste, jump to placeholder) apply to all
- Live Issues: compiler errors highlighted as you type, before building
- Fix-Its: one-click fixes for compiler errors (e.g., add missing switch cases)
- Action Menu (Cmd+click on a symbol): Make Conditional, Add Documentation, Jump to Definition, Quick Help, Embed in VStack, etc.
- Code Completion with contextual suggestions and placeholder completion (Tab / Ctrl+/)
- Inspector panel with Attributes view: change view properties (font, color) with the UI updating live and code inserted automatically
- Library (+): Views, Modifiers, and Code Snippets; drag into Source Editor or directly onto Canvas
- Doc comments: Cmd+click → Add Documentation; renders in Quick Help

**SwiftUI Canvas (Xcode 11)**
- Editor > Canvas to open; live preview rebuilds as code changes
- `previewLayout(.sizeThatFits)` to show view at natural size
- Drag controls from Library onto Canvas to embed in stacks automatically
- Inspector + Attributes: select a view on canvas to edit modifiers graphically

**Debugging**
- Breakpoints: click line number; drag out to remove
- Debug Bar: step, continue, view debugger, memory graph debugger
- Variables View + Quick Look (eye icon): renders images, colors, custom `debugQuickLookObject()`
- Call Stack / Debug Navigator: switch frames to update editor and variables view
- Console for LLDB commands

**Swift Packages**
- Add via Project Editor → Swift Packages tab → +; can browse starred GitHub repos if GitHub account is linked
- Sources visible in Project Navigator under Swift Package Dependencies
- Import and use public APIs; `@testable import` for internal access in test targets

**Framework Targets**
- File > New Target → Framework to create a standalone testable framework
- Move source files by dragging in Project Navigator; Xcode auto-updates target membership
- Include unit test target checkbox when creating framework

**Testing**
- Unit tests: `XCTestCase` subclasses; test diamond in gutter to run individual tests
- UI tests: `XCUIApplication`, `XCUIElement`, `XCTAssertEqual`; automatic screenshots on failure
- Test Plans: shared JSON files describing which targets to run and how; editable in Test Plan Editor
- Report Navigator: detailed pass/fail breakdown, expandable activities, failure screenshots with jump-to-source
- `@testable import` to test internal interfaces

**Distribution**
- Product > Archive with "Generic iOS Device" destination
- Organizer: browse archives, click Distribute App to upload to App Store Connect / TestFlight
- Code signing managed via Signing & Capabilities tab

## APIs & Frameworks

**Xcode 11 Features** **[NEW]**
- Mini Map **[NEW]** — right-margin file overview with mark-based navigation
- Test Plans **[NEW]** — `.xctestplan` files; assign targets and configurations per scheme test action
- Multi-cursor editing **[NEW]** — Control+Shift+click
- SwiftUI Canvas interactive preview (separate from SwiftUI itself) **[NEW]**

**SwiftUI**
- `View`, `VStack`, `Text`, `Button`, `Image`, `Picker` — basic view types
- `previewLayout(_:)` modifier — `.sizeThatFits` for natural-size canvas preview
- `environmentObject(_:)` — injects `ObservableObject` into environment
- `@EnvironmentObject` property wrapper — accesses injected object
- `@ObservedObject` — wraps a model object for reactive updates
- `PreviewProvider` — supplies views to the Canvas

**HealthKit**
- `HKHealthStore` — main store class; used for reading/writing mindful session data
- `HKObjectType.categoryType(forIdentifier: .mindfulSession)` — mindful session category type
- `requestAuthorization(toShare:read:completion:)` — request permission

**XCTest**
- `XCTestCase` — base class for unit and UI tests
- `XCUIApplication` — launches and controls the app under test
- `XCUIElement` — represents a UI element in the app
- `XCTAssertEqual(_:_:)` — equality assertion
- `measure { }` — performance measurement closure

**Source Control**
- Git integration: Source Control > Commit; Preferences > Accounts to add GitHub/Bitbucket

## Code Highlights

Creating a conditional SwiftUI view via Action Menu:
```swift
// Cmd+click on Text → "Make Conditional" produces:
if meditationController.isActive {
    Text(meditationController.remainingTime)
        .font(.largeTitle)
}
```

Adding doc comment via Action Menu:
```swift
/// Requests permission from the user to access mindful sessions.
/// - Note: This method may execute asynchronously.
/// - Parameter completion: A closure to execute when the request is done.
func requestAccess(completion: @escaping (Bool) -> Void)
```

## Takeaways
- Xcode 11's mini map and multi-cursor editing substantially speed up navigation and repetitive edits — learn the keyboard shortcuts.
- Live Issues and Fix-Its surface compiler errors before a build; trust them to write correct Swift faster.
- Swift Packages integrate into Xcode projects natively; use the Project Editor Packages tab rather than external tooling.
- Use Test Plans to share test configurations across your team and tie performance tests to `XCTest.measure {}` for automatic regression detection.

---
_Source: WWDC19 Session 404 page (abstract, chapter summaries, code samples, and resource links)._
