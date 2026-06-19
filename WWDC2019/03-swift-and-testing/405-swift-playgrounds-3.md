# Swift Playgrounds 3
**WWDC19 · Session 405** · [Watch](https://developer.apple.com/videos/play/wwdc2019/405/)

_Platforms:_ iPadOS 13

## Overview
Swift Playgrounds 3 introduces a major new capability: user-accessible modules and multi-file editing. Users can now organize their Swift code across multiple files and modules directly in the iPad app, using a tabbed editing experience. This makes Swift Playgrounds a more capable development scratchpad for exploring APIs, prototyping ideas, and building small apps entirely on iPad.

The session also covers new authoring features for playground book creators: module modes that control how much of the module system is exposed to learners, per-page file activation controls, code completion directives for shared files, and the ability to write cutscene content in Swift instead of HTML.

## Key Topics

### Multi-File Editing and Modules **[NEW]**
- Users can add modules (directories of Swift files) and Swift source files to any module directly within Swift Playgrounds 3.
- Each page still has its own `main.swift` execution entry point.
- Modules are automatically imported into every page — no target/build configuration needed.
- Access levels work normally: `public` code in a module is accessible from all pages; `private` stays within the file; inter-module imports require explicit `import ModuleName`.
- Reset document resets all user edits, including added modules and files.

### Tabbed Editing Experience **[NEW]**
- Multiple source files can be open simultaneously as tabs.
- Tabs can be reordered by drag; closed individually.
- The file and module picker allows navigating to any file in the document.
- Code stepping (Step Through My Code) automatically switches to the file being executed.

### Inline Results and Execution Controls
- Expressions on their own line produce inline results (numbers, images, graphs for numeric sequences).
- Inline results can be disabled via Execution Controls Menu to speed up performance-sensitive code (e.g., SpriteKit game loops).
- `PlaygroundSupport.PlaygroundPage.current.needsIndefiniteExecution = true` keeps execution running for time-based observation.

### CoreMotion, SpriteKit, and SDK Access
- Swift Playgrounds 3 supports Swift 5 and the iOS 12.2 SDK.
- Full access to frameworks including CoreMotion, SpriteKit, ARKit 2, Core ML 2, UIKit.
- SpriteKit live views can be set as the playground live view for real-time rendered output.

### Issues Popover **[NEW]**
- New Issues popover shows build errors and runtime errors across all files in the document.
- Tapping an error navigates directly to the corresponding line in any file.

### Book Authoring: Module Modes **[NEW]**
Three modes set in the book-level `Manifest.plist` under `UserModuleMode`:
- **None:** No user modules; all code in private modules. Classic page/chapter experience. Code per page is completely independent. Examples: Learn to Code 1 & 2.
- **Limited:** One user-editable module with shared files that persist throughout the book. Good for teaching multi-file organization. Examples: Blu's Adventure, Assemble Your Camera.
- **Full:** Multiple user modules, creating/deleting modules, inter-module imports. Best for teaching code architecture. Examples: all Starting Points.

Private modules (not editable by learners) can be used in any mode — placed in the `Modules` folder.

### Book Authoring: Per-Page File Controls **[NEW]**
- `UserModuleSourceFilesToOpen` in the page-level `Manifest.plist` — specifies which shared files are pre-opened as tabs when a page loads.
- `UserModuleSourceFileToActivate` — specifies which file is frontmost (active) when arriving on a page (default is `main.swift`).

### Book Authoring: Code Completion Directives for Shared Files **[NEW]**
- Code completion directives now work in shared module files in addition to `main.swift`.
- Use `UserModuleCodeCompletionDirectives` in page-level Manifest with an array of directives.
- Key directives: `everything hide` (blank slate), `currentmodule show` (show user-created symbols), `module <name> show` (show public API from a module).
- Can also expose language keywords like `public` and `private` as students start working with shared files.

### Cutscenes in Swift **[NEW]**
- Previously cutscenes (intro/summary animations between pages) could only be authored in HTML.
- Now can be written in Swift using SpriteKit, UIKit, or CoreAnimation — no tool switching required.

## APIs & Frameworks

### PlaygroundSupport (for users)
- `PlaygroundSupport.PlaygroundPage.current.needsIndefiniteExecution = true` — keeps playground running
- `PlaygroundSupport.PlaygroundPage.current.setLiveView(_:)` — sets the live view (e.g., `SKView`)

### Swift Playgrounds Book Authoring
- `Manifest.plist` keys (book-level):
  - `UserModuleMode` — `"None"` | `"Limited"` | `"Full"` **[NEW]**
- `Manifest.plist` keys (page-level):
  - `UserModuleSourceFilesToOpen: [String]` — relative paths to shared files **[NEW]**
  - `UserModuleSourceFileToActivate: String` — path to active file **[NEW]**
  - `UserModuleCodeCompletionDirectives: [String]` — directives for shared file completion **[NEW]**
- Code Completion Directives (existing, now extended to shared files):
  - `everything, hide`
  - `currentmodule, show`
  - `module <name>, show`

### iOS Frameworks Available in Playgrounds (iOS 12.2 SDK)
- `CoreMotion` — `CMMotionManager`, accelerometer/gyroscope data
- `SpriteKit` — `SKView`, `SKScene`, `SKNode`, `SKPhysicsBody`, `SKAction`
- `UIKit` — full UIKit
- `CoreML` — Core ML 2 model inference
- `ARKit` — ARKit 2 scene understanding
- `Foundation` — `DispatchQueue`, `Timer`, etc.

## Code Highlights

Keeping a playground running indefinitely for sensor observation:
```swift
import PlaygroundSupport
PlaygroundPage.current.needsIndefiniteExecution = true
```

Reading accelerometer data continuously:
```swift
import CoreMotion
let manager = CMMotionManager()
manager.startAccelerometerUpdates()
// After delay:
manager.accelerometerData?.acceleration.x  // inline result shows graph
```

Setting a SpriteKit scene as the live view:
```swift
import PlaygroundSupport
import SpriteKit
let view = SKView(frame: CGRect(x: 0, y: 0, width: 400, height: 600))
let scene = GameScene(size: view.bounds.size)
view.presentScene(scene)
PlaygroundPage.current.liveView = view
```

Sharing a utility function across pages (in a module file):
```swift
// Timing.swift in Utilities module
import Foundation
public func repeatEvery(_ interval: TimeInterval, action: @escaping () -> Void) {
    action()
    DispatchQueue.main.asyncAfter(deadline: .now() + interval) {
        repeatEvery(interval, action: action)
    }
}
```

## Takeaways
- Swift Playgrounds 3's module system transforms it from a single-file sandbox into a multi-file Swift development environment usable entirely on iPad.
- Book authors should select the right `UserModuleMode` first — it determines the entire learner experience architecture.
- Inline result disable + SpriteKit/ARKit live views is the right pattern for performance-sensitive interactive prototypes.
- Code completion directives now work in shared module files, giving authors fine-grained control over API exposure across the book's multi-file structure.

---
_Source: WWDC19 Session 405 page (abstract, chapter summaries, code samples, and resource links)._
